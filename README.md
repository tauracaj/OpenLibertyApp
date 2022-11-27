# Open Liberty App
## NOTE: Sample Open Liberty App
## Source https://github.com/OpenLiberty/guide-docker
### https://github.com/OpenLiberty/guide-docker
### https://openliberty.io/guides/docker.html


## Prerequisites
- Azure Container Registry
- Azure Kubernetes Cluster
- Terraform code for provisioning Infra can be found - https://github.com/tauracaj/AzureK8s-OpenLiberty

## Step-Docker: Docker File
- Docker File is located under main/docker

## Step-AKS: AKS
### openlibertyapplication-agic
- Aks Yaml file located main/aks

## Step-BuildAndDeploy: BuildAndDeploy
### BuildAndDeploy
- BuildAndDeploy is the github workflow and can be found under .github/workflow
- workflow_dispatch
- Uses Input Variables for
   - acrName
   - appNamespace
   - clusterName
   - clusterRGName
- Build
  - Checkout
  - Setup BuildX
  - Cache Docker layers
  - Setup environment variables
  - Build Open Liberty Project
  - Persist workflow data as artificats 
  - Login To azure
  - Read Image Name and Version from the Solution
  - Build and push
  - Scan image
  - Upload Anchore scan SARIF report
  - Inspect action SARIF report
  - Archive openlibertyapplication-agic.yaml
  - Set AKS context
  - Setup kubectl
  - Deploy to AKS
