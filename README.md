# Express T2S App - ECR Deployment Guide (Beginner Friendly)

This project demonstrates how to push a Docker image of the Express Web App to AWS Elastic Container Registry (ECR) using three different methods: Bash Shell Script, Python Script, and Terraform.

---

## Method 1: Bash Shell Script to Push Docker Image to AWS ECR

```bash
#!/bin/bash
```
- Declares this file is a Bash script.

```bash
REPO_NAME=t2s-express-app
REGION=us-east-1
```
- Sets the repository name and AWS region as variables. You’ll use these multiple times later.

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```
- Retrieves your AWS account ID dynamically so you don’t need to hardcode it.

```bash
IMAGE_TAG=latest
```
- Tags the Docker image version as `latest`.

```bash
aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com
```
- Authenticates your Docker client with AWS ECR using your credentials and region.

```bash
aws ecr describe-repositories --repository-names $REPO_NAME --region $REGION > /dev/null 2>&1
if [ $? -ne 0 ]; then
  aws ecr create-repository --repository-name $REPO_NAME --region $REGION
fi
```
- Checks if the ECR repository exists. If not, it creates it.

```bash
docker build -t $REPO_NAME .
docker tag $REPO_NAME:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
docker push $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
```
- Builds the Docker image, tags it, and pushes it to the ECR repository.

---

## Method 2: Python Script to Push Docker Image to AWS ECR

```python
import boto3
import subprocess
import os
```
- Imports libraries:
  - `boto3`: AWS SDK for Python
  - `subprocess`: Used to run shell commands
  - `os`: For file system navigation

```python
os.chdir("..")
```
- Moves up one directory — assumes your Dockerfile is in the parent folder.

```python
repo_name = "t2s-express-app"
region = "us-east-1"
image_tag = "latest"
```
- Sets your repository name, AWS region, and Docker image tag.

```python
ecr = boto3.client('ecr', region_name=region)
sts = boto3.client('sts')
```
- Initializes clients to interact with AWS ECR and STS (to get account identity).

```python
account_id = sts.get_caller_identity()["Account"]
repo_uri = f"{account_id}.dkr.ecr.{region}.amazonaws.com/{repo_name}"
```
- Builds the full URI of the ECR repository.

```python
try:
    ecr.describe_repositories(repositoryNames=[repo_name])
except ecr.exceptions.RepositoryNotFoundException:
    ecr.create_repository(repositoryName=repo_name)
```
- Checks if the repository exists. If it doesn’t, the script creates it.

```python
subprocess.run(f"aws ecr get-login-password --region {region} | docker login --username AWS --password-stdin {repo_uri}", shell=True, check=True)
subprocess.run(f"docker build -t {repo_name} .", shell=True, check=True)
subprocess.run(f"docker tag {repo_name}:{image_tag} {repo_uri}:{image_tag}", shell=True, check=True)
subprocess.run(f"docker push {repo_uri}:{image_tag}", shell=True, check=True)
```
- Logs in to ECR, builds the Docker image, tags it, and pushes it.

---

## Method 3: Terraform Scripts to Create ECR Repository

### main.tf

```hcl
provider "aws" {
  region = var.region
}

resource "aws_ecr_repository" "app" {
  name         = var.repo_name
  force_delete = true
}
```
- Configures AWS provider and creates an ECR repository with `force_delete = true` to allow deletion even if images exist.

### variables.tf

```hcl
variable "region" {
  description = "AWS region"
}

variable "repo_name" {
  description = "Repository name"
}
```
- Declares input variables.

### terraform.tfvars

```hcl
region    = "us-east-1"
repo_name = "t2s-express-app"
```
- Sets the actual values for the declared variables.

### outputs.tf (optional)

```hcl
output "ecr_repo_url" {
  value       = aws_ecr_repository.app.repository_url
  description = "URL of the created ECR repository"
}
```
- Outputs the ECR repo URL when provisioning is complete.

---

This guide is part of a beginner-friendly Express Web App DevOps project. See full project: https://github.com/Here2ServeU/express-t2s-app-v2
