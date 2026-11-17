# AWS VPC Endpoints using Terraform

This repository provisions a sample AWS networking and VPC endpoints setup using Terraform. It creates a VPC, public/private subnets, route tables, internet gateway, S3 gateway endpoint, and several AWS interface endpoints such as SSM, EC2 Messages, SSM Messages, STS, and EC2. The design is split into separate networking and application-layer environment modules to keep infrastructure reusable and easier to manage.

## Overview

The repository demonstrates how to:

- Build a VPC with public and private subnets
- Create a private network path for AWS service access without internet exposure
- Use VPC Gateway and Interface Endpoints for AWS services
- Deploy a private EC2 instance with IAM permissions to access an S3 bucket via AWS internal networking
- Keep Terraform state in a remote S3 backend

## Repository structure

```text
.
├── .gitignore
├── enviroments/
│   └── aws-vpc-endpoints/
│       ├── endpoints/
│       │   ├── backend.tf
│       │   ├── data.tf
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   ├── terraform.tfvars
│       │   └── variables.tf
│       └── network/
│           ├── backend.tf
│           ├── main.tf
│           ├── outputs.tf
│           ├── terraform.tfvars
│           └── variables.tf
└── modules/
    └── aws-vpc-s3-endpoints/
        ├── ec2/
        ├── endpoint-policy/
        ├── iam/
        ├── s3/
        ├── security-group/
        └── vpc/
```

## What is deployed

### Network layer

The `enviroments/aws-vpc-endpoints/network` configuration creates:

- VPC
- Internet Gateway
- Public and private subnets
- Public and private route tables
- Route table associations
- Private network ACL
- S3 Gateway endpoint
- Interface endpoints for:
  - SSM
  - EC2 Messages
  - SSM Messages
  - STS
  - EC2

### Application layer

The `enviroments/aws-vpc-endpoints/endpoints` configuration provisions:

- Security group for EC2 and endpoint traffic
- S3 bucket
- IAM role and instance profile
- EC2 instance configured for private access

## Modules

The reusable modules under `modules/aws-vpc-s3-endpoints` include:

- `vpc` — VPC, subnets, route tables, gateways, and endpoint-related networking
- `security-group` — EC2 and endpoint security rules
- `s3` — S3 bucket creation
- `iam` — IAM policy and role for S3 access
- `ec2` — EC2 instance deployment
- `endpoint-policy` — policy support for endpoint access rules

## Prerequisites

Before deploying this infrastructure, ensure you have:

- An AWS account with permissions to create VPCs, subnets, EC2 instances, IAM roles, S3 buckets, and VPC endpoints
- Terraform installed locally
- AWS CLI configured with valid credentials
- A suitable S3 bucket for Terraform remote state backend

## Configuration

The environment values are defined in the `.tfvars` files:

- `enviroments/aws-vpc-endpoints/network/terraform.tfvars`
- `enviroments/aws-vpc-endpoints/endpoints/terraform.tfvars`

Example configuration values in this repository include:

- `environment = "dev"`
- `aws_region = "ap-south-1"`
- `vpc_cidr = "172.16.0.0/16"`
- public/private subnet CIDR ranges and AZs defined for the VPC
- S3 bucket name and EC2 instance type configured per environment

## Remote backend

The network stack uses an S3 backend configured in:

- `enviroments/aws-vpc-endpoints/network/backend.tf`

This is configured with a bucket named `tf-backend-08` and a Terraform state key under `network/bld/dev/vpc-endpoints-demo/terraform.tfstate`.

## Deployment steps

From the repository root:

1. Initialize and apply the network layer:

```bash
cd enviroments/aws-vpc-endpoints/network
terraform init
terraform plan
terraform apply
```

2. Initialize and apply the endpoints layer:

```bash
cd ../endpoints
terraform init
terraform plan
terraform apply
```

3. Destroy when no longer needed:

```bash
cd ../endpoints
terraform destroy

cd ../network
terraform destroy
```

## Notes

- The directory name is spelled `enviroments` in this repository, matching the current project structure.
- This project focuses on an AWS private-access pattern using VPC endpoints instead of routing all AWS service traffic over the public internet.
- The example is useful for learning VPC endpoint architecture, private service access, and Terraform modular design.

## License

This project does not appear to include an explicit license file. If you plan to reuse it publicly, confirm the intended licensing terms before publishing or distributing the code.
