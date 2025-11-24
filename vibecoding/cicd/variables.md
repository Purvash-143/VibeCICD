# GitHub Actions Variables and Secrets Configuration (Updated)

## 🎯 **Recommended: OIDC Authentication (Secure)**

### Required Secrets (Only 1)
```
AWS_ROLE_ARN
Description: IAM Role ARN for OIDC authentication
Example: arn:aws:iam::777203855866:role/GitHubActionsRole
Required: Yes
```

### Required Variables (5)
```
AWS_REGION
Value: ap-south-1
Description: AWS region where resources are located

AWS_ACCOUNT_ID  
Value: 777203855866
Description: Your AWS Account ID (12-digit number)

ECR_REPOSITORY
Value: insurance-app
Description: ECR repository name for Docker images

EKS_CLUSTER_NAME
Value: eks-nvai-devops
Description: Name of your EKS cluster

KUBERNETES_NAMESPACE
Value: insurance-application
Description: Kubernetes namespace for deployment
```

## 🚫 **What You DON'T Need Anymore:**
- ❌ `AWS_ACCESS_KEY_ID` (replaced by OIDC)
- ❌ `AWS_SECRET_ACCESS_KEY` (replaced by OIDC)  
- ❌ `AWS_SESSION_TOKEN` (never needed for CI/CD)

## 🔧 **OIDC Setup (One-time)**

### 1. Create IAM OIDC Provider
```bash
aws iam create-open-id-connect-provider \
    --url https://token.actions.githubusercontent.com \
    --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
    --client-id-list sts.amazonaws.com
```

### 2. Create IAM Role for GitHub Actions
```bash
# Create trust policy (replace YOUR_USERNAME/YOUR_REPO)
cat > trust-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::777203855866:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:YOUR_USERNAME/YOUR_REPO:*"
                }
            }
        }
    ]
}
EOF

# Create role
aws iam create-role \
    --role-name GitHubActionsRole \
    --assume-role-policy-document file://trust-policy.json
```

### 3. Create and Attach Policy
```bash
cat > permissions-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability", 
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow", 
            "Action": [
                "eks:DescribeCluster",
                "eks:ListClusters"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "sts:GetCallerIdentity",
            "Resource": "*"
        }
    ]
}
EOF

# Create policy
aws iam create-policy \
    --policy-name GitHubActionsPolicy \
    --policy-document file://permissions-policy.json

# Attach to role
aws iam attach-role-policy \
    --role-name GitHubActionsRole \
    --policy-arn arn:aws:iam::777203855866:policy/GitHubActionsPolicy

# Get role ARN (use as AWS_ROLE_ARN secret)
aws iam get-role --role-name GitHubActionsRole --query 'Role.Arn' --output text
```

## AWS IAM Permissions

Your AWS user/role needs these permissions:

### ECR Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage"
            ],
            "Resource": "*"
        }
    ]
}
```

### EKS Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:DescribeCluster",
                "eks:ListClusters"
            ],
            "Resource": "*"
        }
    ]
}
```

### Additional AWS Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        }
    ]
}
```

## Environment-Specific Configurations

### Development Environment
```yaml
Variables:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: insurance-app-dev
  EKS_CLUSTER_NAME: eks-nvai-devops-dev
  KUBERNETES_NAMESPACE: insurance-application-dev
```

### Production Environment
```yaml
Variables:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: insurance-app
  EKS_CLUSTER_NAME: eks-nvai-devops
  KUBERNETES_NAMESPACE: insurance-application
```

## Setup Commands

### 1. Create ECR Repository (if not exists)
```bash
aws ecr create-repository \
    --repository-name insurance-app \
    --region ap-south-1
```

### 2. Tag EKS Subnets (if needed)
```bash
# Get VPC ID
VPC_ID=$(aws eks describe-cluster --name eks-nvai-devops --region ap-south-1 --query cluster.resourcesVpcConfig.vpcId --output text)

# Tag public subnets
aws ec2 create-tags \
    --resources subnet-xxx subnet-yyy \
    --tags Key=kubernetes.io/role/elb,Value=1

# Tag private subnets
aws ec2 create-tags \
    --resources subnet-aaa subnet-bbb \
    --tags Key=kubernetes.io/role/internal-elb,Value=1
```

### 3. Verify Setup
```bash
# Test ECR access
aws ecr get-login-password --region ap-south-1

# Test EKS access
aws eks describe-cluster --name eks-nvai-devops --region ap-south-1

# Test kubectl access
aws eks update-kubeconfig --name eks-nvai-devops --region ap-south-1
kubectl get nodes
```

## Security Best Practices

1. **Rotate AWS Keys Regularly**: Update AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY every 90 days
2. **Use Least Privilege**: Grant only required permissions to AWS user
3. **Monitor Access**: Enable CloudTrail for AWS API monitoring
4. **Separate Environments**: Use different AWS accounts/credentials for dev/prod
5. **Secret Scanning**: GitLeaks will scan for exposed secrets in code

## Troubleshooting Variables

### Common Issues

**Error**: `The security token included in the request is invalid`
**Solution**: Check AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY are correct

**Error**: `No cluster found for name: xxx`
**Solution**: Verify EKS_CLUSTER_NAME variable matches actual cluster name

**Error**: `repository does not exist`
**Solution**: Create ECR repository or verify ECR_REPOSITORY variable

**Error**: `error: You must be logged in to the server (Unauthorized)`
**Solution**: Check AWS credentials have EKS permissions

### Variable Validation Script
```bash
#!/bin/bash
echo "Validating GitHub Actions variables..."

# Check AWS credentials
aws sts get-caller-identity

# Check ECR repository
aws ecr describe-repositories --repository-names $ECR_REPOSITORY --region $AWS_REGION

# Check EKS cluster
aws eks describe-cluster --name $EKS_CLUSTER_NAME --region $AWS_REGION

# Test kubectl
aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION
kubectl get nodes

echo "✅ All variables validated successfully!"
```

## Current Configuration Summary

Based on your setup:

```yaml
Secrets:
  AWS_ACCESS_KEY_ID: [Your AWS Access Key]
  AWS_SECRET_ACCESS_KEY: [Your AWS Secret Key]
  AWS_ACCOUNT_ID: 777203855866

Variables:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: insurance-app
  EKS_CLUSTER_NAME: eks-nvai-devops
  KUBERNETES_NAMESPACE: insurance-application
```
