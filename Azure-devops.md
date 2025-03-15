# Importing Repository & Deploying Microservices using Azure ACR, AKS, and Azure DevOps

## Prerequisites
Before setting up the CI/CD pipeline, ensure the following Azure resources are created:
- **Azure Container Registry (ACR)** for storing Docker images.
- **Azure Kubernetes Service (AKS)** for deploying microservices.
- **Service Connection** in Azure DevOps to securely connect with Azure resources.

---
## Step 1: Create an Azure Account
1. Sign up or log in to the [Azure Portal](https://portal.azure.com).
2. Set up a **subscription** and a **resource group** for managing resources.

---
## Step 2: Setup Azure DevOps Organization & Project
1. Navigate to [Azure DevOps](https://dev.azure.com/).
2. Create an **organization** and a **new project**.
3. Enable Git repositories and pipeline options.

---
## Step 3: Clone Git Repository & Import to Azure Repos
1. In your Azure DevOps project, navigate to **Repos**.
2. Select "Import Repository" and provide the source Git URL.
3. Clone the newly imported repository to your local machine using:
   ```sh
   git clone <repo-url>
   ```

---
## Step 4: Launch a VM Server for the Agent
1. In the Azure Portal, create a **Virtual Machine (VM)**.
2. Choose an Ubuntu or Windows server depending on the agent requirements.
3. Connect to the VM via SSH or RDP.

---
## Step 5: Create an Agent Pool
1. Navigate to **Azure DevOps > Organization Settings > Agent Pools**.
2. Click on "New Agent Pool" and provide a name.
3. Assign appropriate permissions.

---
## Step 6: Install Agent on the VM
1. Navigate to **Azure DevOps > Project Settings > Agent Pools**.
2. Select the agent pool created earlier.
3. Download and install the agent using:
   ```sh
   mkdir myagent && cd myagent
   curl -O https://vstsagentpackage.azureedge.net/agent/3.220.2/vsts-agent-linux-x64-3.220.2.tar.gz
   tar zxvf vsts-agent-linux-x64-3.220.2.tar.gz
   ./config.sh
   ./svc.sh install
   ./svc.sh start
   ```
4. Verify the agent status in Azure DevOps.

---
## Step 7: Create a Service Connection
1. Navigate to **Azure DevOps > Project Settings > Service Connections**.
2. Click "New Service Connection" and select **Azure Resource Manager**.
3. Authenticate and select the **ACR and AKS resources**.
4. Save the connection for CI/CD pipelines.

---
## Step 8: CI Pipeline - Build & Push Docker Image to ACR
1. Navigate to **Azure DevOps > Pipelines**.
2. Click "New Pipeline" and select **YAML**.
3. Use the following sample pipeline to **checkout code, build a Docker image, and push to ACR**:
   ```yaml
   trigger:
     - main

   pool:
     name: Default

   steps:
     - task: Docker@2
       inputs:
         command: buildAndPush
         repository: $(containerRegistry)/cart-service
         dockerfile: Dockerfile
         tags: latest
     - script: echo "Build and push to ACR completed"
   ```

---
## Step 9: CD Pipeline - Deploy to AKS
1. Navigate to **Azure DevOps > Pipelines > Releases**.
2. Click "New Release Pipeline" and select **Kubernetes Deployment**.
3. Use the following **Kubernetes manifest for deploying the service**:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cart-service
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: cart
     template:
       metadata:
         labels:
           app: cart
       spec:
         containers:
         - name: cart
           image: <acr-name>.azurecr.io/cart-service:latest
           ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: cart-service
   spec:
     selector:
       app: cart
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```
4. Deploy the pipeline and monitor logs for errors.

---
## Step 10: Verify Deployment
1. Run the following commands to validate the deployment:
   ```sh
   kubectl get pods -n <namespace>
   kubectl get services -n <namespace>
   ```
2. If successful, your microservice is now deployed to **Azure Kubernetes Service (AKS)**.

---
## Summary
- Imported GitHub repository to Azure Repos.
- Configured **Azure DevOps Agent & Service Connection**.
- Built Docker images and pushed them to ACR using **CI Pipeline**.
- Deployed microservices to **AKS using CD Pipeline**.
