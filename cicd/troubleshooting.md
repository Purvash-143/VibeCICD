# CI/CD Pipeline Troubleshooting Guide

## Common Issues and Solutions

### 1. GitLeaks Stage Issues

#### Issue: GitLeaks fails to scan
```
Error: fatal: not a git repository
```

**Solution:**
- Ensure `fetch-depth: 0` is set in checkout action
- Check repository permissions

#### Issue: False positives in secret detection
**Solution:**
- Create `.gitleaksignore` file in repository root
- Add patterns to ignore false positives:
```
# .gitleaksignore
test-secret-123
dummy-api-key
example.com/api/key
```

### 2. Linting Stage Issues

#### Issue: ESLint configuration not found
```
Error: No ESLint configuration found
```

**Solution:**
- Add `.eslintrc.json` in vibecoding directory:
```json
{
  "extends": ["next/core-web-vitals"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "warn",
    "@next/next/no-img-element": "warn"
  }
}
```

#### Issue: Prettier configuration conflicts
**Solution:**
- Add `.prettierrc` in vibecoding directory:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### 3. CodeQL Stage Issues

#### Issue: CodeQL build fails
```
Error: We were unable to automatically build your code
```

**Solution:**
- Ensure Node.js dependencies are installed before CodeQL analysis
- Add manual build steps if auto-build fails
- Check supported languages in CodeQL config

### 4. Build Stage Issues

#### Issue: Docker build fails
```
Error: failed to solve: failed to read dockerfile
```

**Solution:**
- Verify Dockerfile exists in `vibecoding/` directory
- Check Dockerfile syntax
- Ensure working directory is correct in GitHub Actions

#### Issue: ECR push fails
```
Error: denied: Your authorization token has expired
```

**Solution:**
- Verify AWS credentials are not expired
- Check ECR repository exists
- Ensure AWS_ACCOUNT_ID is correct

### 5. Trivy Scan Issues

#### Issue: Trivy fails to scan image
```
Error: failed to scan image
```

**Solution:**
- Ensure image was successfully built and pushed
- Check ECR repository permissions
- Verify image tag format

#### Issue: High/Critical vulnerabilities found
**Solution:**
- Review Trivy report in Security tab
- Update base images to latest versions
- Add vulnerability exceptions if needed

### 6. Deployment Stage Issues

#### Issue: kubectl authentication fails
```
Error: You must be logged in to the server (Unauthorized)
```

**Solution:**
- Verify AWS credentials have EKS permissions
- Check EKS cluster name and region
- Ensure IAM user has necessary EKS policies

#### Issue: Image pull errors in Kubernetes
```
Error: Failed to pull image "xxx": rpc error: code = Unknown desc = Error response from daemon: pull access denied
```

**Solution:**
- Verify ECR secret is created correctly
- Check ECR repository permissions
- Ensure image tag exists in ECR

#### Issue: LoadBalancer external IP pending
```
Status: LoadBalancer Ingress: <pending>
```

**Solution:**
- Check subnet tagging for ELB
- Verify AWS Load Balancer Controller is running
- Ensure security groups allow traffic

### 7. General AWS Issues

#### Issue: AWS credentials rotation
**Solution:**
- Update GitHub Secrets with new AWS keys
- Test credentials with AWS CLI
- Check IAM user policies

#### Issue: Rate limiting errors
```
Error: Throttling: Rate exceeded
```

**Solution:**
- Add delays between AWS API calls
- Use AWS CLI retry configuration
- Implement exponential backoff

### 8. GitHub Actions Specific Issues

#### Issue: Workflow doesn't trigger
**Solution:**
- Check branch names in trigger configuration
- Verify file is in `.github/workflows/` directory
- Check YAML syntax

#### Issue: Secrets not accessible
```
Error: The secret `AWS_ACCESS_KEY_ID` was not found
```

**Solution:**
- Verify secrets are set in repository settings
- Check secret names match exactly
- Ensure secrets are available in current environment

### 9. Performance Issues

#### Issue: Pipeline takes too long
**Solution:**
- Use caching for Node.js dependencies
- Implement Docker layer caching
- Run stages in parallel where possible

#### Issue: Out of memory errors
**Solution:**
- Increase Node.js memory limit
- Optimize Docker build process
- Use smaller base images

### 10. Environment-Specific Issues

#### Issue: Different behavior in dev vs prod
**Solution:**
- Use environment-specific variables
- Implement environment gates
- Test in staging environment first

## Debugging Commands

### AWS Debugging
```bash
# Check AWS credentials
aws sts get-caller-identity

# List ECR repositories
aws ecr describe-repositories --region ap-south-1

# Check EKS cluster
aws eks describe-cluster --name eks-nvai-devops --region ap-south-1

# Test kubectl connectivity
kubectl get nodes
kubectl get pods -n insurance-application
```

### Docker Debugging
```bash
# List ECR images
aws ecr list-images --repository-name insurance-app --region ap-south-1

# Test image pull
docker pull 777203855866.dkr.ecr.ap-south-1.amazonaws.com/insurance-app:latest

# Scan image locally
trivy image 777203855866.dkr.ecr.ap-south-1.amazonaws.com/insurance-app:latest
```

### Kubernetes Debugging
```bash
# Check deployment status
kubectl rollout status deployment/insurance-app-deployment -n insurance-application

# Check pod logs
kubectl logs -l app=insurance-app -n insurance-application

# Check service
kubectl get svc insurance-app-service -n insurance-application

# Describe problematic resources
kubectl describe pod <pod-name> -n insurance-application
```

## Log Analysis Tips

### GitLeaks Logs
- Look for "leak found" messages
- Check file paths and line numbers
- Review secret types detected

### Build Logs
- Check for dependency installation errors
- Look for compilation failures
- Review Docker build context issues

### Deployment Logs
- Monitor pod startup events
- Check image pull status
- Review service endpoint creation

## Escalation Process

1. **Check Pipeline Status**: Review all stage results
2. **Examine Logs**: Download and analyze action logs
3. **Test Locally**: Reproduce issues in local environment
4. **AWS Console**: Check AWS resources directly
5. **GitHub Issues**: Create issue with logs if needed

## Prevention Strategies

1. **Pre-commit Hooks**: Add GitLeaks and linting locally
2. **Branch Protection**: Require successful checks before merge
3. **Regular Updates**: Keep dependencies and tools updated
4. **Monitoring**: Set up alerts for pipeline failures
5. **Documentation**: Keep troubleshooting guide updated

## Quick Fixes Checklist

- [ ] AWS credentials are valid and not expired
- [ ] ECR repository exists and has correct permissions
- [ ] EKS cluster is accessible and running
- [ ] GitHub secrets are properly configured
- [ ] Branch names match trigger conditions
- [ ] Dockerfile and K8s manifests are valid
- [ ] Subnet tagging is correct for LoadBalancer
- [ ] Node.js dependencies are compatible
