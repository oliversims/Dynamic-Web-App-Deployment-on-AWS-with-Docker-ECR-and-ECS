# Dynamic Web App Deployment on AWS with Docker, ECR, and ECS

This project demonstrates the deployment of a dynamic web application on AWS using containerization with Docker, Amazon ECR for image storage, and Amazon ECS for orchestration. The infrastructure is designed for high availability, security, and scalability.

## Architecture Overview

The application is deployed using a multi-tier architecture with the following components:

-   **Frontend**: Dynamic web application containerized with Docker
-   **Backend**: MySQL RDS database in private subnets
-   **Infrastructure**: VPC with public/private subnets, Load Balancer, and ECS cluster
-   **Security**: Security Groups, IAM roles, and SSL/TLS encryption

## Infrastructure Components

### Network Infrastructure

-   **VPC**: Custom Virtual Private Cloud with public and private subnets across multiple availability zones
-   **Internet Gateway**: Provides internet access to public subnets
-   **NAT Gateways**: Enable outbound internet connectivity for private subnet resources
-   **Security Groups**: Fine-grained access controls between EC2 instances, RDS databases, and load balancers

### Database

-   **MySQL RDS Instance**: Deployed in private subnets for secure, high-availability database hosting
-   **Bastion Host**: EC2 instance for secure access to the private RDS instance
-   **Flyway**: Database migration tool for managing SQL scripts and schema changes

### Application Hosting

-   **Amazon ECS**: Container orchestration service managing the application deployment
-   **Application Load Balancer**: Distributes traffic with HTTPS listener using AWS Certificate Manager
-   **Amazon ECR**: Container registry for storing Docker images

### Domain and DNS

-   **Route 53**: Custom domain registration and DNS management
-   **SSL Certificate**: AWS Certificate Manager for encrypted communication

## Deployment Process

### 1. Local Development Setup

```powershell
# Make shell script executable (PowerShell)
Set-ExecutionPolicy -ExecutionPolicy Unrestricted

```

```bash
# Make shell script executable (Mac/Linux)
chmod +x build_image.sh

```

### 2. Docker Image Build and Push

#### Build Script (PowerShell)

```powershell
# Set up buildx builder for multi-architecture builds
$builderName = "multiarch-builder"
$existing = docker buildx ls | Select-String $builderName

if (-not $existing) {
    docker buildx create --name $builderName
}

docker buildx use $builderName

# Build the image for linux/amd64
docker buildx build `
  --platform linux/amd64 `
  --build-arg PERSONAL_ACCESS_TOKEN='your_token' `
  --build-arg GITHUB_USERNAME='your_username' `
  --build-arg REPOSITORY_NAME='applications-codes' `
  --build-arg WEB_FILE_ZIP='rentzone.zip' `
  --build-arg WEB_FILE_UNZIP='rentzone' `
  --build-arg DOMAIN_NAME='your_domain.com' `
  --build-arg RDS_ENDPOINT='your_rds_endpoint' `
  --build-arg RDS_DB_NAME='applicationdb' `
  --build-arg RDS_MASTER_USERNAME='admin' `
  --build-arg RDS_DB_PASSWORD='your_password' `
  -t rentzone .

```

#### ECR Commands

```bash
# Create ECR repository
aws ecr create-repository --repository-name <repository-name> --region <region>

# Login to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

# Retag Docker image
docker tag <local-image> <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:latest

# Push Docker image to ECR
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:latest

```

### 3. Database Migration with SSH Tunnel

#### PowerShell

```powershell
ssh -i <key_pair.pem> ec2-user@<bastion-host-ip> -L 3306:<rds-endpoint>:3306 -N

```

#### Linux/macOS

```bash
ssh -i "YOUR_EC2_KEY" -L LOCAL_PORT:RDS_ENDPOINT:REMOTE_PORT EC2_USER@EC2_HOST -N -f

```

### 4. ECS Deployment

-   **ECS Cluster**: Container orchestration cluster
-   **Task Definition**: Container specifications and resource requirements
-   **ECS Service**: Manages desired number of running tasks
-   **IAM Role**: Grants container access to AWS services (S3, CloudWatch)

## Key Issues Encountered and Solutions

### Multi-Architecture Build Issue

**Problem**: ECS was unable to pull the container image from ECR, receiving the error:

```
CannotPullContainerError: image Manifest does not contain descriptor matching platform 'linux/amd64'

```

**Root Cause**: The Docker image in ECR was missing the `linux/amd64` architecture required by ECS Fargate.

**Solution**:

1.  **Implemented Docker Buildx**: Used Docker Buildx to build multi-architecture images
2.  **Platform Specification**: Added `--platform linux/amd64` flag to target the correct architecture
3.  **Builder Setup**: Created a dedicated buildx builder for consistent multi-architecture builds
4.  **Updated Build Process**: Modified the build script to use `docker buildx build` instead of `docker build`

The solution ensures that all images built for ECS deployment are compatible with the `linux/amd64` platform, preventing deployment failures.

## Security Considerations

-   **Environment Variables**: Sensitive data stored in S3 and injected at runtime
-   **IAM Roles**: Least privilege access for ECS tasks
-   **Private Subnets**: Database and sensitive resources isolated from direct internet access
-   **Security Groups**: Network-level access controls
-   **SSL/TLS**: Encrypted communication via Application Load Balancer

## Prerequisites

-   AWS CLI installed and configured
-   Docker Desktop with virtualization enabled
-   IAM user with programmatic access
-   Custom domain registered in Route 53
-   SSL certificate from AWS Certificate Manager

## Repository Structure

```
├── Dockerfile
├── build_image.ps1
├── AppServiceProvider.php
├── sql-scripts/
│   └── migration files
└── README.md

```

## References

-   [Dockerfile Reference](https://apps.abacus.ai/chatllm/link-to-dockerfile)
-   [Push Commands](https://apps.abacus.ai/chatllm/link-to-push-commands)
-   [Environment File Reference](https://apps.abacus.ai/chatllm/link-to-env-file)
-   [IAM Policy Reference](https://apps.abacus.ai/chatllm/link-to-iam-policy)

## Contributing

1.  Fork the repository
2.  Create a feature branch
3.  Commit your changes
4.  Push to the branch
5.  Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
