# CI/CD Configuration for Insurance Application

This directory contains the complete CI/CD pipeline configuration for the Insurance Application.

## Pipeline Overview

The CI/CD pipeline consists of 6 main stages:

### 1. 🔍 GitLeaks - Secret Detection
- Scans code for exposed secrets, API keys, and credentials
- Continues on error (warns but doesn't fail)
- Uploads results as artifacts

### 2. 🧹 Code Linting & Syntax Check
- ESLint for code quality and syntax
- Prettier for code formatting
- Continues on error (warns but doesn't fail)
- Uploads linting results

### 3. 🔒 CodeQL Security Analysis
- GitHub's semantic code analysis
- Identifies security vulnerabilities
- Continues on error (warns but doesn't fail)
- Results appear in Security tab

### 4. 🏗️ Build Application
- Node.js application build
- Docker image creation
- Push to AWS ECR
- Only fails if build actually fails

### 5. 🛡️ Trivy Docker Image Scan
- Vulnerability scanning of Docker images
- SARIF format results
- Continues on error (warns but doesn't fail)
- Uploads to GitHub Security tab

### 6. 🚀 Deploy to EKS
- Deploys to AWS EKS cluster
- Updates Kubernetes manifests
- Health checks and validation
- Only runs if build succeeds

## Trigger Conditions

- **Push**: `main` and `develop` branches
- **Pull Request**: targeting `main` branch

## File Structure

```
.github/
├── workflows/
│   └── cicd-pipeline.yml    # Main CI/CD pipeline
└── ...

cicd/
├── README.md               # This file
├── variables.md            # Required GitHub Actions variables
└── troubleshooting.md      # Common issues and solutions
```

## Security Features

- All test stages continue on error (warn only)
- Secrets detection with GitLeaks
- Security analysis with CodeQL
- Container vulnerability scanning with Trivy
- AWS credentials securely managed

## Monitoring & Artifacts

- All scan results uploaded as artifacts
- Deployment summary in GitHub Actions
- Health check validation
- External URL provided after deployment

## Quick Start

1. Set up required GitHub Actions secrets (see variables.md)
2. Push code to `main` or `develop` branch
3. Monitor pipeline in GitHub Actions tab
4. Access deployed application via provided URL

## Support

For issues or questions:
- Check troubleshooting.md
- Review GitHub Actions logs
- Verify AWS credentials and permissions
