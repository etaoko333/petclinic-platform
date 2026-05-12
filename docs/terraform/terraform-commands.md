# GPM-244: Terraform Commands — fmt, validate, plan, apply

## Prerequisites
- Terraform >= 1.10.0 installed
- AWS CLI configured with correct credentials
- S3 backend bucket exists: petclinit-tf-state-achievers11

## Step 1 — Initialise Terraform
cd petclinic-k8s-platform/terraform
terraform init -backend-config=backend.hcl

## Step 2 — Format all Terraform files
terraform fmt -recursive
# Automatically formats all .tf files to standard style
# Run this before every commit

## Step 3 — Check formatting
terraform fmt -check -recursive
# Returns exit code 0 if all files are correctly formatted
# Returns exit code 1 if any file needs formatting

## Step 4 — Validate configuration
terraform validate
# Checks syntax and internal consistency
# Does NOT check AWS resources — just the Terraform code

## Step 5 — Plan infrastructure
terraform plan -out=tfplan
# Shows exactly what will be created/changed/destroyed
# Always review plan output before applying
# -out=tfplan saves plan for deterministic apply

## Step 6 — Apply infrastructure
terraform apply tfplan
# Applies the saved plan — no surprises
# Takes 15-20 minutes for full stack

## Step 7 — Destroy infrastructure (after every session)
terraform plan -destroy -out=tfplan
terraform apply tfplan
# ALWAYS destroy after practice to avoid costs

## GitHub Actions Automation (infra.yml)
The infra.yml workflow automates all steps above:
- Triggers on: push/PR to main (terraform/** changes)
- Also supports manual trigger via workflow_dispatch
- Actions available: plan, apply, destroy
- apply and destroy only allowed from main branch
- Uses AWS_ROLE_ARN for OIDC authentication

## Cost Warning
Running terraform apply creates real AWS resources that cost money.
- EKS cluster: ~$0.10/hour for control plane
- EC2 nodes (2x t3.small): ~$0.046/hour each
- RDS (db.t3.micro): ~$0.017/hour
- Total: approximately $0.22/hour

ALWAYS run terraform destroy when done practicing.
