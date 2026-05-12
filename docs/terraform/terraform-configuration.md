# GPM-242: Terraform Configuration for AWS Infrastructure

## Provider Configuration (providers.tf)
- Terraform version: >= 1.10.0
- AWS provider version: ~> 5.0
- Random provider version: ~> 3.0
- Backend: S3 with state locking

## VPC Module (terraform/modules/vpc/)

### What it creates
- 1 VPC with CIDR 10.0.0.0/16
- 2 public subnets (10.0.1.0/24, 10.0.2.0/24)
- 1 Internet Gateway for outbound traffic
- 1 Route Table with 0.0.0.0/0 route to IGW
- Route table associations for both subnets

### Subnet tags for EKS
- kubernetes.io/cluster/petclinic-eks = shared
- kubernetes.io/role/elb = 1

### Outputs
- vpc_id: ID of the created VPC
- public_subnet_ids: List of subnet IDs
- vpc_cidr: VPC CIDR block (used by RDS security group)

## EKS Module (terraform/modules/eks/)

### What it creates
- EKS Cluster: petclinic-eks (Kubernetes 1.32)
- Cluster IAM Role with AmazonEKSClusterPolicy
- Node IAM Role with 3 policies:
  AmazonEKSWorkerNodePolicy
  AmazonEKS_CNI_Policy
  AmazonEC2ContainerRegistryReadOnly
- Security Group: allows HTTPS 443 inbound
- Managed Node Group: 2x t3.small (min 1, max 3)
- OIDC Provider for IRSA (pod-level AWS permissions)
- EBS CSI Driver addon (for persistent volumes)
- CoreDNS addon
- kube-proxy addon

### Inputs
- cluster_name: petclinic-eks
- vpc_id: from VPC module
- subnet_ids: from VPC module

### Outputs
- cluster_name, cluster_endpoint, cluster_ca
- node_role_arn, oidc_provider_arn

## ECR Module (terraform/modules/ecr/)

### What it creates
- 8 ECR repositories (one per service):
  config-server, discovery-server, api-gateway
  customers-service, vets-service, visits-service
  admin-server, genai-service
- image_tag_mutability: MUTABLE
- force_delete: true (allows clean terraform destroy)
- scan_on_push: true
- Lifecycle policy: keep last 10 tagged, expire untagged after 1 day

### Outputs
- repository_urls: map of service name to ECR URL
- registry_id: AWS account ID

## RDS Module (terraform/modules/rds/)

### What it creates
- Random password: 16 chars, no special chars
- Security Group: allows MySQL 3306 from VPC CIDR 10.0.0.0/16
- DB Subnet Group across both public subnets
- RDS MySQL 8.0 instance (db.t3.micro, 20GB gp2)
- Database: petclinic, Username: petclinic
- multi_az: false, publicly_accessible: false
- skip_final_snapshot: true
- Secrets Manager secret: petclinic/database-credentials
  Contains: username, password, endpoint, port, dbname

### Inputs
- vpc_id: from VPC module
- subnet_ids: from VPC module
- eks_cidr: VPC CIDR from VPC module

### Outputs
- db_endpoint, db_port, db_name, secret_arn
