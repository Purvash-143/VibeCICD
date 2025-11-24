# Simple GitHub Actions Variables Setup

## 📋 **Complete Variables List (All in GitHub Actions Variables)**

Go to **Settings → Secrets and variables → Actions → Variables** and add these **7 variables**:

### **Required Variables (7 total):**

```
AWS_ACCESS_KEY_ID
Value: [Your AWS Access Key ID]
Example: AKIAIOSFODNN7EXAMPLE
Description: AWS Access Key for ECR and EKS access

AWS_SECRET_ACCESS_KEY  
Value: [Your AWS Secret Access Key]
Example: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Description: AWS Secret Key for ECR and EKS access

AWS_REGION
Value: ap-south-1
Description: AWS region where your EKS cluster is located

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

## 🚫 **What You DON'T Need:**
- ❌ No secrets required
- ❌ No IAM roles to create
- ❌ No OIDC setup
- ❌ No complex AWS configurations

## ✅ **Simple Setup Steps:**

### 1. Get Your AWS Credentials
```bash
# Check your AWS credentials
aws configure list

# Or get them from AWS Console:
# IAM → Users → Your User → Security credentials → Create access key
```

### 2. Get Your AWS Account ID
```bash
aws sts get-caller-identity --query Account --output text
```

### 3. Add All 7 Variables to GitHub
1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **Variables** tab
4. Click **New repository variable**
5. Add each of the 7 variables above

### 4. Verify Your Values
Make sure these match your actual AWS setup:
- **AWS_REGION**: `ap-south-1` (your EKS region)
- **AWS_ACCOUNT_ID**: `777203855866` (your account ID)
- **ECR_REPOSITORY**: `insurance-app` (your ECR repo name)
- **EKS_CLUSTER_NAME**: `eks-nvai-devops` (your cluster name)
- **KUBERNETES_NAMESPACE**: `insurance-application` (your K8s namespace)

## 🔧 **Required AWS Permissions**

Your AWS user needs these permissions:

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

## 🚀 **Test Your Setup**

After adding variables, test locally:

```bash
# Test AWS credentials
aws sts get-caller-identity

# Test ECR access
aws ecr get-login-password --region ap-south-1

# Test EKS access  
aws eks describe-cluster --name eks-nvai-devops --region ap-south-1

# Test kubectl (if configured)
kubectl get nodes
```

## 📋 **Quick Checklist**

- [ ] Added `AWS_ACCESS_KEY_ID` variable
- [ ] Added `AWS_SECRET_ACCESS_KEY` variable  
- [ ] Added `AWS_REGION` variable (ap-south-1)
- [ ] Added `AWS_ACCOUNT_ID` variable (777203855866)
- [ ] Added `ECR_REPOSITORY` variable (insurance-app)
- [ ] Added `EKS_CLUSTER_NAME` variable (eks-nvai-devops)
- [ ] Added `KUBERNETES_NAMESPACE` variable (insurance-application)
- [ ] AWS user has ECR and EKS permissions
- [ ] ECR repository exists
- [ ] EKS cluster is accessible

## 🎉 **That's It!**

No roles, no OIDC, no complex setup. Just **7 variables** and you're ready to deploy! 

Push your code to `main` or `develop` branch and watch the pipeline run! 🚀
