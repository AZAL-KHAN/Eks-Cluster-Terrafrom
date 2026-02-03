# EKS Cluster using Terraform

This repository provisions an Amazon EKS cluster using **Terraform**.

## Features
- Kubernetes v1.34
- Managed Node Groups
- Auto Scaling (3–5 nodes)
- Public Subnets Only
- No NAT Gateway (Cost Optimized)

## Tech Stack
- Terraform
- AWS EKS
- IAM
- VPC

## 📁 Repository Structure
```
Eks-Cluster-Terrafrom/
├── versions.tf        # Terraform & provider versions
├── provider.tf        # AWS provider configuration
├── variables.tf       # Input variables
├── vpc.tf             # VPC, subnets, routing (No NAT Gateway)
├── iam.tf             # IAM roles & policies for EKS
├── eks.tf             # EKS cluster & managed node group
├── outputs.tf         # Useful outputs
└── README.md
```

## Deployment

terraform init  
terraform apply
