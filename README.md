# NextGen Telco Infrastructure as Code

This repository contains Terraform configurations for deploying a complete cloud infrastructure for NextGen Telco applications.

## Project Structure

```
terraform/
├── modules/           # Reusable Terraform modules
│   ├── vpc/          # Virtual Private Cloud module
│   ├── eks/          # Amazon EKS Kubernetes module
│   ├── ecr/          # Amazon ECR Docker registry module
│   ├── alb/          # Application Load Balancer module
│   ├── iam/          # IAM roles and policies module
│   └── security-groups/  # Security groups module
│
├── environments/       # Environment-specific configurations
│   ├── dev/          # Development environment
│   ├── staging/      # Staging environment
│   └── prod/         # Production environment
│
├── global/           # Global resources shared across environments
│   ├── s3-backend/   # S3 backend for Terraform state
│   └── iam-roles/    # Global IAM roles
│
├── scripts/          # Utility scripts
│   ├── eks-connect.sh      # Connect to EKS cluster
│   └── terraform-init.sh   # Initialize Terraform
│
└── README.md         # This file
```

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI configured with credentials
- Terraform >= 1.0
- kubectl (for Kubernetes interaction)
- bash shell

## Getting Started

### 1. Initialize the Backend

First, set up the S3 backend for storing Terraform state:

```bash
cd terraform/scripts
chmod +x terraform-init.sh eks-connect.sh
./terraform-init.sh --backend
```

### 2. Initialize an Environment

Initialize the development environment (or staging/prod):

```bash
./terraform-init.sh --environment dev
```

### 3. Review and Apply Configuration

```bash
cd ../environments/dev
terraform plan
terraform apply
```

## Environments

### Development
- **VPC CIDR**: 10.0.0.0/16
- **EKS Nodes**: 2-5 (t3.medium)
- **Deletion Protection**: Disabled

### Staging
- **VPC CIDR**: 10.1.0.0/16
- **EKS Nodes**: 3-10 (t3.large)
- **Deletion Protection**: Enabled

### Production
- **VPC CIDR**: 10.2.0.0/16
- **EKS Nodes**: 5-20 (m5.xlarge)
- **Deletion Protection**: Enabled

## Modules

### VPC Module
Creates a Virtual Private Cloud with public and private subnets across multiple availability zones.

**Key Resources**:
- VPC with custom CIDR
- Public and private subnets
- Internet Gateway
- Route tables and associations

### EKS Module
Provisions Amazon EKS cluster with managed node groups.

**Key Resources**:
- EKS cluster
- Managed node groups
- IAM roles and policies
- CloudWatch logging

### ECR Module
Creates Amazon ECR repositories for Docker images.

**Key Resources**:
- ECR repositories
- Image scanning
- Lifecycle policies
- Repository policies

### ALB Module
Sets up Application Load Balancer for routing traffic.

**Key Resources**:
- Application Load Balancer
- Target groups
- Listeners
- Health checks

### IAM Module
Manages IAM roles and policies for applications.

**Key Resources**:
- IAM roles
- Custom policies
- Role attachments
- Instance profiles

### Security Groups Module
Configures security groups for network traffic control.

**Key Resources**:
- Security groups
- Ingress rules
- Egress rules

## Useful Commands

### Plan Changes
```bash
cd environments/<env>
terraform plan
terraform plan -out=tfplan  # Save plan to file
```

### Apply Changes
```bash
terraform apply
terraform apply tfplan  # Apply saved plan
```

### Destroy Resources
```bash
terraform destroy
terraform destroy -auto-approve  # Skip confirmation
```

### View Outputs
```bash
terraform output
terraform output -json  # Output as JSON
```

### Format Configuration
```bash
terraform fmt -recursive
```

### Validate Configuration
```bash
terraform validate
```

## Scripts

### eks-connect.sh
Configures kubectl to connect to your EKS cluster.

```bash
./scripts/eks-connect.sh dev    # Connect to dev cluster
./scripts/eks-connect.sh staging
./scripts/eks-connect.sh prod
```

### terraform-init.sh
Initializes Terraform for different environments.

```bash
./scripts/terraform-init.sh --backend           # Initialize backend
./scripts/terraform-init.sh --environment dev  # Initialize dev
./scripts/terraform-init.sh --all              # Initialize everything
./scripts/terraform-init.sh --validate         # Validate all
```

## State Management

Terraform state is stored in S3 with the following configuration:

- **Bucket**: nextgen-telco-tfstate
- **Encryption**: AES256
- **Versioning**: Enabled
- **State Lock**: DynamoDB table (terraform-locks)

## Variables and Locals

### Common Variables
- `aws_region`: AWS region (default: us-east-1)
- `environment`: Environment name (dev/staging/prod)
- `common_tags`: Tags to apply to all resources

### Environment-Specific Variables
See `terraform.tfvars` in each environment directory for specific values.

## Outputs

Each environment provides the following outputs:

- `vpc_id`: VPC identifier
- `vpc_cidr`: VPC CIDR block
- `eks_cluster_endpoint`: EKS API endpoint
- `eks_cluster_security_group_id`: EKS security group
- `alb_dns_name`: Load balancer DNS name
- `ecr_repository_urls`: ECR repository URLs

## Security Considerations

1. **State Management**: Terraform state contains sensitive information
   - Ensure S3 bucket has proper access controls
   - Enable versioning and encryption
   - Use state locking with DynamoDB

2. **IAM Roles**: Use least privilege principle
   - Review IAM policies in modules
   - Restrict access based on use case

3. **Network Security**: Security groups follow best practices
   - Minimal ingress rules
   - Explicit egress rules

4. **Encryption**: 
   - S3 encryption enabled
   - EBS encryption recommended
   - Consider KMS keys for sensitive data

## Troubleshooting

### Backend is locked
```bash
cd environments/<env>
terraform force-unlock <LOCK_ID>
```

### Resources not destroying
```bash
terraform destroy -auto-approve
```

### Permission denied errors
Check AWS credentials and IAM permissions.

### Terraform not finding modules
Ensure you're in the correct directory and run `terraform init`.

## Contributing

1. Test changes in dev environment first
2. Review Terraform plan output carefully
3. Use `terraform fmt` to maintain code style
4. Document changes in PR description

## Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Terraform Best Practices](https://www.terraform.io/language)

## Support

For issues or questions, please open an issue in the repository.
