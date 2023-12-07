## Comprehensive CI/CD Pipeline for Redis Deployment in AKS with Spectral and ShiftLeft Scanning

This repository contains a Jenkins pipeline configuration and Kubernetes deployment manifests for a complete CI/CD workflow. The pipeline is designed to build, scan, and deploy a Redis Docker image to Azure Kubernetes Service (AKS). It integrates Spectral and ShiftLeft scans for security and vulnerability analysis.

### Pipeline Overview
1. **Clone GitHub Repository**: Retrieves the latest code from the GitHub repository.
2. **Spectral Installation**: Installs Spectral, a tool for scanning codebases for secrets, misconfigurations, and Infrastructure as Code (IaC) vulnerabilities.
3. **Pre-Build Spectral Scan**: Executes a Spectral scan before the Docker image build process.
4. **Docker Image Build**: Constructs the Docker image for Redis and saves it locally as `redis.tar`.
5. **ShiftLeft Container Image Scans**: Performs online and offline ShiftLeft scans on the Docker image for security vulnerabilities.
6. **Post-Build Spectral Scan**: Runs another Spectral scan after the Docker image build process.
7. **Azure ACR Login**: Authenticates with Azure Container Registry (ACR) using service principal credentials.
8. **Push Image to ACR**: Tags and pushes the built Redis image to ACR.
9. **Deployment to AKS**: Deploys the Redis image to AKS using Kubernetes manifests.

### Kubernetes Configuration Files
- **Deployment Configuration**: Defines the deployment settings for the Redis application in AKS, specifying the image to be pulled from ACR and the container port.
- **Service Configuration**: Creates a LoadBalancer service for the Redis deployment, exposing it on the specified port.

### Key Aspects of the Pipeline
- **Security Integration**: Incorporates Spectral and ShiftLeft scans to ensure the Redis image is secure and free of vulnerabilities before deployment.
- **CI/CD Automation**: Fully automates the build, scan, and deployment process, ensuring a streamlined and efficient workflow.
- **Azure Integration**: Seamlessly integrates with Azure services, including Azure Container Registry and Azure Kubernetes Service.

### Usage Scenario
This setup is ideal for teams looking to deploy Redis in a Kubernetes environment with an emphasis on security and automated workflows. The pipeline ensures that each deployment of Redis is secure, up-to-date, and consistent with the source code in the repository.

---

This readme summary provides a comprehensive overview of the repository's contents and its purpose, highlighting the integration of security scanning tools and automated deployment processes in a Kubernetes environment. It serves as a guide for users looking to understand and leverage this pipeline for deploying secure and reliable applications in Azure Kubernetes Service.
