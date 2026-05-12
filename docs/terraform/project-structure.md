# GPM-240: Terraform Project Structure

## Repository
Achievers11-DevOps/petclinic-k8s-platform

## Terraform Directory Structure
terraform/
  backend.hcl        <- S3 backend configuration
  providers.tf       <- AWS and Random providers
  main.tf            <- Root module calling all 4 modules
  variables.tf       <- Input variables (region, cluster name)
  outputs.tf         <- Output values (cluster name, ECR URLs)
  modules/
    vpc/             <- VPC, subnets, internet gateway
      main.tf
      variables.tf
      outputs.tf
    eks/             <- EKS cluster, node group, OIDC, addons
      main.tf
      variables.tf
      outputs.tf
    ecr/             <- ECR repositories for all 8 services
      main.tf
      variables.tf
      outputs.tf
    rds/             <- RDS MySQL, security group, Secrets Manager
      main.tf
      variables.tf
      outputs.tf

## S3 Backend (backend.hcl)
- Bucket: petclinit-tf-state-achievers11
- Key: dev/petclinic/terraform.tfstate
- Region: us-east-1
- Encryption: enabled
- State locking: enabled

## Root Variables
| Variable | Default | Purpose |
|----------|---------|---------|
| aws_region | us-east-1 | AWS deployment region |
| cluster_name | petclinic-eks | EKS cluster name |
| environment | production | Resource tagging |
| domain | eta-oko.com | Application domain |

## Module Dependencies
vpc module -> eks module (provides vpc_id, subnet_ids)
vpc module -> rds module (provides vpc_id, subnet_ids, vpc_cidr)
ecr module -> independent (no dependencies)
