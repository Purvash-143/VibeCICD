# Complete Variables List for CI/CD Pipeline

## 🔐 **GitHub Actions Secrets (Required)**

Add these in **Settings → Secrets and variables → Actions → Secrets**:

### AWS Authentication (OIDC - Recommended)
```
AWS_ROLE_ARN
Description: IAM Role ARN for OIDC authentication
Example: arn:aws:iam::777203855866:role/GitHubActionsRole
Required: Yes
```

### Alternative: AWS Long-term Credentials (Not Recommended)
If you can't use OIDC, you can use these instead:
```
AWS_ACCESS_KEY_ID
Description: AWS Access Key (only if OIDC not available)
Example: AKIAIOSFODNN7EXAMPLE
Required: Only if not using OIDC

AWS_SECRET_ACCESS_KEY
Description: AWS Secret Key (only if OIDC not available)
Example: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Required: Only if not using OIDC
```

### GitHub Token (Auto-provided)
```
GITHUB_TOKEN
Description: Automatically provided by GitHub Actions
Required: No (auto-provided)
```

## 📊 **GitHub Actions Variables (Required)**

Add these in **Settings → Secrets and variables → Actions → Variables**:

```
AWS_REGION
Value: ap-south-1
Description: AWS region where resources are located
Required: Yes

AWS_ACCOUNT_ID
Value: 777203855866
Description: Your AWS Account ID (12-digit number)
Required: Yes

ECR_REPOSITORY
Value: insurance-app
Description: ECR repository name for Docker images
Required: Yes

EKS_CLUSTER_NAME
Value: eks-nvai-devops
Description: Name of your EKS cluster
Required: Yes

KUBERNETES_NAMESPACE
Value: insurance-application
Description: Kubernetes namespace for deployment
Required: Yes
```

## 🎯 **Recommended Setup: OIDC Authentication**

### Step 1: Create IAM Role for GitHub Actions
```bash
# Create trust policy for GitHub Actions OIDC
cat > github-actions-trust-policy.json << 'EOF'
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
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/YOUR_REPO_NAME:*"
                }
            }
        }
    ]
}
EOF

# Create the IAM role
aws iam create-role \
    --role-name GitHubActionsRole \
    --assume-role-policy-document file://github-actions-trust-policy.json

# Get the role ARN (use this as AWS_ROLE_ARN secret)
aws iam get-role --role-name GitHubActionsRole --query 'Role.Arn' --output text
```

### Step 2: Create IAM Policy for Required Permissions
```bash
cat > github-actions-policy.json << 'EOF'
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
            "Action": [
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        }
    ]
}
EOF

# Create the policy
aws iam create-policy \
    --policy-name GitHubActionsPolicy \
    --policy-document file://github-actions-policy.json

# Attach policy to role
aws iam attach-role-policy \
    --role-name GitHubActionsRole \
    --policy-arn arn:aws:iam::777203855866:policy/GitHubActionsPolicy
```

### Step 3: Create OIDC Identity Provider (if not exists)
```bash
# Check if OIDC provider exists
aws iam list-open-id-connect-providers

# If not exists, create it
aws iam create-open-id-connect-provider \
    --url https://token.actions.githubusercontent.com \
    --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
    --client-id-list sts.amazonaws.com
```

## 🚫 **What You DON'T Need:**

- ❌ `AWS_ACCESS_KEY_ID` (if using OIDC)
- ❌ `AWS_SECRET_ACCESS_KEY` (if using OIDC)
- ❌ `AWS_SESSION_TOKEN` (never needed for CI/CD)
- ❌ Hard-coded AWS credentials in code
- ❌ Long-term access keys

## ✅ **Complete Variables Summary:**

### **Secrets (Only 1 required):**
```
AWS_ROLE_ARN: arn:aws:iam::777203855866:role/GitHubActionsRole
```

### **Variables (5 required):**
```
AWS_REGION: ap-south-1
AWS_ACCOUNT_ID: 777203855866
ECR_REPOSITORY: insurance-app
EKS_CLUSTER_NAME: eks-nvai-devops
KUBERNETES_NAMESPACE: insurance-application
```

## 🔄 **Alternative: If You Can't Use OIDC**

If OIDC setup is not possible, you can use long-term credentials:

### **Secrets (3 required):**
```
AWS_ACCESS_KEY_ID: [Your AWS Access Key]
AWS_SECRET_ACCESS_KEY: [Your AWS Secret Key]
AWS_ACCOUNT_ID: 777203855866
```

### **Variables (4 required):**
```
AWS_REGION: ap-south-1
ECR_REPOSITORY: insurance-app
EKS_CLUSTER_NAME: eks-nvai-devops
KUBERNETES_NAMESPACE: insurance-application
```

Then update the pipeline to use the old authentication method.

## 🎯 **Benefits of OIDC Approach:**

- ✅ **No long-term credentials** stored in GitHub
- ✅ **Automatic credential rotation**
- ✅ **Fine-grained permissions** per repository
- ✅ **AWS security best practices**
- ✅ **No credential expiration issues**

## 🚀 **Quick Setup Commands:**

```bash
# 1. Get your GitHub repo info
GITHUB_REPO="YOUR_USERNAME/YOUR_REPO_NAME"

# 2. Create OIDC provider (if needed)
aws iam create-open-id-connect-provider \
    --url https://token.actions.githubusercontent.com \
    --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
    --client-id-list sts.amazonaws.com

# 3. Create IAM role with proper trust policy
# (Replace YOUR_USERNAME/YOUR_REPO_NAME with actual values)

# 4. Add AWS_ROLE_ARN to GitHub Secrets

# 5. Add all 5 variables to GitHub Variables

# 6. Push code and watch pipeline run!
```

That's it! With OIDC, you only need **1 secret** and **5 variables** - much cleaner and more secure! 🎉
