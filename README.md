# 1. Cloudformation Eks-Cluster

The scripts will create a VPC and an EKS cluster in: us-east-1

### How to run it
- export AWS_PROFILE=user1 # The name of the profile you want to use
- ./create-vpc-stack.sh
- ./create-eks-stack.sh

### How to clean up
- ./delete-eks-stack.sh
- ./delete-vpc-stack.sh


### Tips

### Update Kubeconfig
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster

# 2. Deploying NGINX with Helm on an EKS 
This guide explains how to create and deploy the provided Helm chart to launch an NGINX container on an Amazon EKS cluster. The deployment is exposed using a Kubernetes LoadBalancer service

## Prerequisites

1. **Amazon EKS cluster** running and properly configured.
2. **kubectl** installed and configured to interact with the EKS cluster.
3. **Helm** installed on your local machine.
4. Proper IAM permissions to manage resources in your AWS account.


### 1. Create the Helm Chart 
use the command helm chart and edit the files to fit the specification

### 2. Install the Helm Chart
To install the chart, use the following command:

```bash
helm install <release-name> ./nginx-deployment
```

Replace `<release-name>` with a unique name for this deployment.

### 3. Verify the Deployment
Check if the deployment is running:

```bash
kubectl get deployments
```

Check if the service is exposed:

```bash
kubectl get services
```

# 3. CI/CD Pipeline for Building and Pushing Docker Images to Amazon ECR
This document explains how the CI/CD pipeline works for building a Docker image and pushing it to an Amazon Elastic Container Registry (ECR) using GitHub Actions.

## Workflow Overview

The CI/CD pipeline is defined in the `.github/workflows` directory within the repository. It automates the following steps:

1. **Triggers on Push**: The workflow is triggered whenever changes are pushed to the `main` branch.
2. **Builds a Docker Image**: The pipeline uses the `Dockerfile` in the repository to build a Docker image.
3. **Tags the Image**: The built image is tagged with the latest version and configured for Amazon ECR.
4. **Pushes the Image to ECR**: The image is pushed to the specified Amazon ECR repository.

## Key Steps in the Workflow

### 1. Checkout Code
The `actions/checkout` action retrieves the repository's code to the runner.

```yaml
- name: Checkout Code
  uses: actions/checkout@v3
```

### 2. Log in to Amazon ECR
The `aws-actions/amazon-ecr-login` action logs in to the Amazon ECR service using credentials provided as secrets.

```yaml
- name: Log in to Amazon ECR
  uses: aws-actions/amazon-ecr-login@v1
```

### 3. Build Docker Image
The pipeline builds a Docker image using the repository's `Dockerfile`.

```yaml
- name: Build Docker Image
  run: |
    docker build -t ${{ secrets.ECR_REPOSITORY }}:latest .
```

### 4. Tag Docker Image
The built image is tagged with the ECR repository URI, which includes the AWS account ID and region.

```yaml
- name: Tag Docker Image
  run: |
    docker tag ${{ secrets.ECR_REPOSITORY }}:latest ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:latest
```

### 5. Push Docker Image to ECR
The tagged image is pushed to the Amazon ECR repository.

```yaml
- name: Push Docker Image to ECR
  run: |
    docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:latest
```

## Configuration

### Required Secrets
The following secrets must be configured in the GitHub repository:

1. **`AWS_ACCOUNT_ID`**: The AWS account ID where the ECR repository resides.
2. **`AWS_REGION`**: The AWS region where the ECR repository is hosted.
3. **`ECR_REPOSITORY`**: The name of the ECR repository to push the image.
