# Movie Picture Pipeline

## Project Overview

This project implements a CI/CD pipeline for a full-stack movie application using GitHub Actions, Docker, Amazon ECR, Amazon EKS, and Kubernetes.

The application consists of:

- A React frontend
- A Flask backend API
- Docker containers for both applications
- GitHub Actions for Continuous Integration and Continuous Deployment
- Amazon Elastic Container Registry (ECR) for Docker images
- Amazon Elastic Kubernetes Service (EKS) for deployment
- Kubernetes Deployments and LoadBalancer Services

The goal of the project is to automatically test, build, containerize, publish, and deploy the frontend and backend applications through separate CI/CD workflows.

---

# Architecture

The overall pipeline is:

```text
                    GitHub Repository
                           |
                           |
                  +--------+--------+
                  |                 |
            Frontend Code      Backend Code
                  |                 |
                  v                 v
          Frontend CI          Backend CI
          GitHub Actions       GitHub Actions
                  |                 |
          Lint + Tests          Lint + Tests
                  |                 |
              Docker Build      Docker Build
                  |                 |
                  +--------+--------+
                           |
                           v
                    GitHub Actions CD
                           |
                           v
                  Amazon ECR Repositories
                    /              \
                   /                \
          Frontend Image        Backend Image
                   \                /
                    \              /
                           v
                       Amazon EKS
                           |
                    Kubernetes Cluster
                           |
                  +--------+--------+
                  |                 |
             Frontend Pod      Backend Pod
                  |                 |
             LoadBalancer       LoadBalancer
                  |                 |
                  v                 v
             React App          Flask API



Technologies Used
React
Flask
Python
Node.js
Docker
GitHub
GitHub Actions
Amazon ECR
Amazon EKS
Kubernetes
Terraform
AWS IAM
AWS VPC
Repository Structure
.
├── .github/
│   └── workflows/
│       ├── frontend-ci.yaml
│       ├── frontend-cd.yaml
│       ├── backend-ci.yaml
│       └── backend-cd.yaml
│
├── starter/
│   ├── frontend/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   ├── src/
│   │   └── k8s/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   │
│   └── backend/
│       ├── Dockerfile
│       ├── Pipfile
│       ├── app.py
│       └── k8s/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── kustomization.yaml
│
└── setup/
    └── terraform/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        └── ...
Continuous Integration

Two separate GitHub Actions workflows were created for Continuous Integration.

Frontend CI

Workflow:

.github/workflows/frontend-ci.yaml

The frontend CI workflow:

Runs when frontend-related changes are submitted through a pull request to main.
Can also be triggered manually using workflow_dispatch.
Runs frontend linting.
Runs frontend tests.
Builds the frontend Docker image after the lint and test jobs succeed.

The lint and test jobs run independently, and the Docker build waits for both jobs to complete successfully.

The frontend Docker build uses:

REACT_APP_MOVIE_API_URL=http://localhost:5000

as the build argument.

Backend CI

Workflow:

.github/workflows/backend-ci.yaml

The backend CI workflow:

Runs when backend-related changes are submitted through a pull request to main.
Can also be triggered manually.
Runs backend linting.
Runs backend tests.
Builds the backend Docker image after the lint and test jobs succeed.

The backend CI workflow was successfully tested after resolving the uWSGI build issue in the Docker environment.

Continuous Deployment

Two separate GitHub Actions workflows were created for Continuous Deployment.

Frontend CD

Workflow:

.github/workflows/frontend-cd.yaml

The frontend CD workflow:

Runs on changes to the frontend on the main branch.
Supports manual execution.
Runs linting and tests.
Builds the frontend Docker image.
Logs in to Amazon ECR.
Tags the Docker image using the Git commit SHA.
Pushes the image to the frontend ECR repository.
Updates the Kubernetes configuration with the image repository and tag.
Deploys the frontend to Amazon EKS.

The frontend ECR repository is:

975879576846.dkr.ecr.us-east-1.amazonaws.com/frontend

A successful frontend deployment image was pushed using commit SHA:

cbfe1407a02a63596edb2855471f518ae74cb923
Backend CD

Workflow:

.github/workflows/backend-cd.yaml

The backend CD workflow:

Runs on changes to the backend on the main branch.
Supports manual execution.
Runs linting and tests.
Builds the backend Docker image.
Logs in to Amazon ECR.
Tags the image using the Git commit SHA.
Pushes the image to the backend ECR repository.
Updates the Kubernetes configuration.
Deploys the backend to Amazon EKS.

The backend ECR repository is:

975879576846.dkr.ecr.us-east-1.amazonaws.com/backend

A successful backend deployment image was pushed using commit SHA:

59823e6f5c540af5124cef683b1e0e8bf6dd627d
GitHub Secrets

AWS credentials were not hard-coded into the GitHub Actions workflow files.

The workflows use GitHub repository secrets for AWS authentication.

This keeps AWS credentials outside the source code and allows GitHub Actions to authenticate with AWS during deployment.

AWS Infrastructure

The AWS infrastructure was provisioned using Terraform.

The Terraform configuration created the infrastructure required for the project, including:

Amazon ECR repositories
Amazon EKS cluster
EKS node group
IAM roles
VPC/networking resources
Supporting AWS resources
ECR

Two ECR repositories were created:

frontend
backend

with the following registry:

975879576846.dkr.ecr.us-east-1.amazonaws.com
Amazon EKS

The Kubernetes cluster is:

cluster

AWS Region:

us-east-1

Kubernetes version:

1.32

The EKS node group is:

udacity

The worker node configuration used:

Instance type: t3.small
Capacity type: ON_DEMAND
AMI type: AL2023_x86_64_STANDARD
Kubernetes Deployment

Both applications are deployed using Kubernetes manifests.

Frontend

The frontend Kubernetes resources include:

Deployment
Service
Kustomization

The frontend service is exposed through a Kubernetes LoadBalancer.

The frontend container uses the ECR image:

975879576846.dkr.ecr.us-east-1.amazonaws.com/frontend
Backend

The backend Kubernetes resources include:

Deployment
Service
Kustomization

The backend service is exposed through a Kubernetes LoadBalancer.

The backend container uses the ECR image:

975879576846.dkr.ecr.us-east-1.amazonaws.com/backend
Successful Deployment Verification

During the successful deployment, Kubernetes showed both frontend and backend pods running on the EKS worker node.

The services were exposed using AWS LoadBalancers.

Frontend LoadBalancer:

ac2bef5c2cc634a55a3042601908326d-950500971.us-east-1.elb.amazonaws.com

Backend LoadBalancer:

a61a60bd7beaa49099c759a674fa34ca-1877305021.us-east-1.elb.amazonaws.com

The backend API was successfully tested using curl and returned movie data.

The frontend and backend were both successfully running in the EKS cluster during deployment verification.

CI/CD Workflow Summary

The complete workflow is:

Developer pushes code
        |
        v
GitHub Repository
        |
        +----------------------+
        |                      |
        v                      v
Frontend CI              Backend CI
        |                      |
   Lint + Test             Lint + Test
        |                      |
        v                      v
 Docker Build             Docker Build
        |                      |
        +----------+-----------+
                   |
                   v
             Merge to main
                   |
                   v
              CD Workflows
                   |
            +------+------+
            |             |
            v             v
        Frontend       Backend
          ECR            ECR
            |             |
            +------+------+
                   |
                   v
                EKS
                   |
          +--------+--------+
          |                 |
       Frontend          Backend
       Deployment        Deployment
          |                 |
          v                 v
     Movie UI             Movie API
Docker

Both applications are containerized.

Frontend Docker Image

The frontend Dockerfile:

Uses Node.js 18
Installs dependencies using npm ci
Builds the React application
Exposes port 3000
Serves the built application

The frontend API URL is supplied during the Docker build.

Backend Docker Image

The backend Dockerfile:

Uses the required Python environment
Installs backend dependencies
Runs the Flask application using uWSGI
Exposes the backend service

The Docker build was tested successfully.

Testing

The project was tested at multiple levels.

Frontend
Frontend linting passed.
Frontend tests passed.
Frontend Docker image built successfully.
Backend
Backend linting passed.
Backend tests passed.
Backend Docker image built successfully.
Kubernetes

The deployments were verified using Kubernetes commands including:

kubectl get nodes
kubectl get pods
kubectl get services
kubectl get deployments

During successful deployment verification, both application pods reached the running state.

The backend API was also tested through its LoadBalancer endpoint and returned movie information.

Evidence

The project was successfully deployed and verified in AWS before the AWS training account was subsequently stopped and restarted.

Evidence for the implementation includes:

Successful GitHub Actions CI runs
Successful GitHub Actions CD runs
Docker image builds
Images successfully pushed to Amazon ECR
EKS cluster creation
Kubernetes deployments
Running frontend and backend pods
Kubernetes LoadBalancer services
Backend API response containing movie data

The successful deployment was verified from the terminal using AWS CLI and Kubernetes CLI commands.

Important Deployment Notes

The ECR image tags use Git commit SHA values rather than mutable tags such as latest.

This provides traceability between:

Git commit
     ↓
Docker image
     ↓
ECR image
     ↓
Kubernetes deployment

For example, the successfully deployed frontend image was associated with:

cbfe1407a02a63596edb2855471f518ae74cb923

and the backend deployment image was associated with:

59823e6f5c540af5124cef683b1e0e8bf6dd627d
Project Completion

The CI/CD pipeline was successfully implemented using GitHub Actions and AWS.

The completed solution integrates:

GitHub
   ↓
GitHub Actions
   ↓
Docker
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes
   ↓
Frontend + Backend

The application was successfully deployed and tested in the AWS environment during the project implementation.


### One thing I would change before submitting

I **would not claim browser verification** in the README, because you didn't actually check the application in a browser. We can honestly say it was verified through the terminal/API/Kubernetes output, which is what we actually did.

Also, **don't put your AWS access key, secret key, or any credentials in the README.**

If the Udacity submission form asks for screenshots, we can use your existing terminal evidence. The README itself can be the central documentation.
