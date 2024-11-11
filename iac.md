# Infrastructure as Code (IaC)
## Description
IaC for AWS EKS environment.
## Design
For the purpose of this demonstration project there will be one EKS environment that is shared by development and production, separated by namespaces.
In a real environment this would not happen, but for this demo project the 
environment is created and deleted to run the demo, and does not persist.

The IaC design consists of the following components:
- Base VPC configuration
- EKS config
- Install of Load Balancer Controller to handle Ingress for EKS
- Install of kube-prometheus monitoring stack

### Base VPC configuration
Uses the AWS VPC module (terraform-aws-modules/vpc/aws)
- Target region set in variables.tf
- Finds a set of three AZs in the region that are available
- Creates private and public subnets in each AZ
- Hosts a single NAT gateway (used to fetch container images) in one of the public subnets
- Module takes care of setting up route tables for public subnets and NAT gateways

### EKS configuration
Uses the AWS EKS module (terraform-aws-modules/eks/aws)
- Includes the VPC-CNI addon so that the CNI can access the VPC information
- Sets up both public (for kubectl on local PC) and private (for access within cluster without using NAT/IGW) endpoints for cluster API access
- Sets cluster permissions to allow kubectl admin access to the AWS identity that creates the cluter
- Creates a managed node group that spans the private subnets across AZs that starts with 3 nodes and can grow up to 5 (scaling not implemented at this point)
- Modifies the node security group to allow port 80 traffic between pods

### Load Balancer Controller (LBC)
The LBC is an open-source project that AWS recommends be used to configure ingress into EKS clusters.  
The LBC will automatically configure AWS NLBs (for services) and ALBs (for ingresses) and connect them into the EKS cluster.  
This superseeds the k8s in-tree AWS load balancer that provisions a classic load balancer.

Uses the AWS IAM module and resources from the AWS provider
- Creates an AWS IAM role that has permissions to provision AWS load balancers (ELB for services, ALB for ingress) that can be assumed by a specific k8s service account (kube-system:aws-load-balancer-controller)
- Creates the k8s service account
- Uses Helm to install the LBC and sets it up to use the service account

### Monitoring stack
Installs the community kube-prometheus stack, which includes the Prometheus Operator and Grafana.
Grafana will monitor k8s for a ConfigMap with the appropriate label, and insert a custom dashboard based on ConfigMap.
The ConfigMap is installed as part of the application Helm chart.

## Status
Complete.

See [terraform code](https://github.com/lago-morph/chiller-iac/tree/main/exp/aws).
