##########################
#### Work In Progress ####
##########################


# EKS Cluster Terraform Setup

This is a complete Terraform configuration for deploying a production ready EKS cluster on AWS. It handles all the networking, IAM roles, and cluster setup you need to get started with Kubernetes on AWS.

## What This Creates

The setup provisions a full VPC with public and private subnets across two availability zones for high availability. You get an EKS cluster with managed node groups that can scale based on your workload. All the necessary security groups and IAM roles are configured automatically.

The infrastructure includes NAT gateways for private subnet internet access, an internet gateway for public subnets, and proper routing tables. The EKS cluster is deployed with best practices including private worker nodes and configurable public API access.

## Prerequisites

You need Terraform 1.0 or higher installed on your machine. Make sure you have the AWS CLI configured with credentials that have permissions to create VPCs, EKS clusters, EC2 instances, and IAM roles. You should also have kubectl installed if you want to interact with the cluster after creation.

## Quick Start

First clone this repository and navigate to the directory. Then initialize Terraform to download the required providers.

```
terraform init
```

Review what resources will be created by running a plan.

```
terraform plan
```

When you're ready to create the infrastructure, apply the configuration.

```
terraform apply
```

This will take about 10 to 15 minutes to complete since EKS cluster creation takes some time.

## Connecting to Your Cluster

After the cluster is created, configure kubectl to use it by running this AWS CLI command.

```
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster
```

Replace the region and cluster name with your values if you changed them. Now you can verify connectivity.

```
kubectl get nodes
```

You should see your worker nodes listed and ready.

## Customization

You can customize the deployment by modifying the variables. The main ones you might want to change are in variables.tf.

The cluster_name variable sets the name of your EKS cluster and is used as a prefix for all resources. The aws_region determines where everything gets deployed.

For the node group, you can adjust node_instance_types to change the EC2 instance size. The desired_capacity, min_capacity, and max_capacity control how many nodes you want running.

If you need a larger network, modify the vpc_cidr variable. The default gives you plenty of IP addresses for most use cases.

The kubernetes_version lets you specify which K8s version to use. Make sure to check AWS documentation for supported versions.

## Cost Considerations

Running this infrastructure will incur AWS charges. The main costs come from the EKS cluster itself which has an hourly fee, the EC2 instances in your node group, the NAT gateways for each availability zone, and the EBS volumes attached to your nodes.

With the default settings using t3.medium instances, you're looking at roughly $150 to $200 per month depending on your region and actual usage. You can reduce costs by using smaller instance types or fewer nodes if this is just for testing.

## Cleaning Up

When you're done with the cluster, you can destroy all resources to avoid ongoing charges.

```
terraform destroy
```

Make sure to remove any LoadBalancers or persistent volumes created by Kubernetes workloads first, otherwise the VPC deletion might fail due to remaining dependencies.

## Network Architecture

The VPC is split into public and private subnets. Public subnets host the NAT gateways and have direct internet access through an internet gateway. Private subnets contain your EKS worker nodes and route internet traffic through the NAT gateways.

This setup keeps your worker nodes secure while still allowing them to pull container images and communicate with the EKS control plane. The EKS control plane itself is managed by AWS and sits outside your VPC.

## Security

The configuration creates minimal security groups that allow necessary traffic for EKS to function. The worker nodes are in private subnets and not directly accessible from the internet.

The EKS API endpoint is publicly accessible by default but you can restrict this by modifying the endpoint_public_access setting in main.tf. You might want to limit API access to specific CIDR ranges in a production environment.

All IAM roles follow the principle of least privilege and only have the permissions needed for EKS and worker nodes to operate.

## Next Steps

After your cluster is running, you'll probably want to install additional components. Common additions include the AWS Load Balancer Controller for ingress, the EBS CSI driver for persistent volumes, and a metrics server for autoscaling.

You can also add more node groups with different instance types or configure spot instances to save costs. The cluster is ready for you to deploy applications using kubectl or Helm.

## Troubleshooting

If nodes aren't joining the cluster, check that your NAT gateways are working and the worker nodes can reach the internet. The nodes need to download the EKS optimized AMI and register with the control plane.

For authentication issues, verify your AWS credentials have the necessary permissions. The IAM user or role running Terraform needs extensive permissions to create all these resources.

If kubectl can't connect, make sure you ran the aws eks update-kubeconfig command with the correct cluster name and region.
