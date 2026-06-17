# Secure CI/CD Pipeline Using DevOps Tools
## Complete Implementation Guide for MERN Stack Application
### Final Year Major Project — Azure for Students ($100 Budget)

---

> **Role Context:** This guide is written by a Senior DevOps Engineer / DevSecOps Architect / Kubernetes Administrator / Azure Cloud Engineer with 15+ years of experience, specifically tailored for a Final Year CSE student with no prior DevOps experience.

---

# TABLE OF CONTENTS

1. [Project Architecture Overview](#1-project-architecture-overview)
2. [Azure for Students Setup](#2-azure-for-students-setup)
3. [Cost Estimation and Budget Planning](#3-cost-estimation-and-budget-planning)
4. [Resource Group Creation](#4-resource-group-creation)
5. [Azure CLI Installation](#5-azure-cli-installation)
6. [Azure Login](#6-azure-login)
7. [Azure Container Registry (ACR) Creation](#7-azure-container-registry-acr-creation)
8. [AKS Cluster Creation](#8-aks-cluster-creation)
9. [kubectl Installation](#9-kubectl-installation)
10. [Docker Installation](#10-docker-installation)
11. [Jenkins Installation](#11-jenkins-installation)
12. [Jenkins Configuration](#12-jenkins-configuration)
13. [SonarQube Installation](#13-sonarqube-installation)
14. [SonarQube Configuration](#14-sonarqube-configuration)
15. [Trivy Installation](#15-trivy-installation)
16. [OWASP ZAP Installation](#16-owasp-zap-installation)
17. [Docker Files — Frontend and Backend](#17-docker-files--frontend-and-backend)
18. [Build and Push Docker Images](#18-build-and-push-docker-images)
19. [Kubernetes Manifests](#19-kubernetes-manifests)
20. [Deploy to AKS](#20-deploy-to-aks)
21. [Prometheus and Grafana Monitoring](#21-prometheus-and-grafana-monitoring)
22. [Complete Jenkins Pipeline](#22-complete-jenkins-pipeline)
23. [Security Reports and Verification](#23-security-reports-and-verification)
24. [Troubleshooting Guide](#24-troubleshooting-guide)
25. [Azure Cost Optimization](#25-azure-cost-optimization)
26. [Viva Questions and Answers](#26-viva-questions-and-answers)
27. [Interview Questions and Answers](#27-interview-questions-and-answers)

---

# 1. PROJECT ARCHITECTURE OVERVIEW

## ASCII Architecture Diagram

```
Developer Workstation
        |
        | git push
        v
+--------------------+
|  GitHub Repository |
+--------------------+
        |
        | Webhook (HTTP POST)
        v
+--------------------+      +------------------+
|      Jenkins       |----->|   SonarQube      |
|  (CI/CD Engine)    |      | (SAST Analysis)  |
+--------------------+      +------------------+
        |
        | Docker Build
        v
+--------------------+      +------------------+
|   Docker Images    |----->|   Trivy Scanner  |
|  (Frontend+Backend)|      | (Image Scanning) |
+--------------------+      +------------------+
        |
        | docker push
        v
+-------------------------------+
| Azure Container Registry (ACR)|
+-------------------------------+
        |
        | kubectl apply
        v
+-------------------------------+
| Azure Kubernetes Service (AKS)|
|                               |
|  +------------+  +---------+  |
|  | React FE   |  | Node BE |  |
|  | Pod x2     |  | Pod x2  |  |
|  +------------+  +---------+  |
|         |              |      |
|         v              v      |
|  +-----------------------------+
|  |    Nginx Ingress Controller |
|  +-----------------------------+
+-------------------------------+
        |
        | DAST Scan
        v
+--------------------+
|    OWASP ZAP       |
| (Dynamic Analysis) |
+--------------------+
        |
        v
+--------------------+      +------------------+
|    Prometheus      |----->|     Grafana      |
|  (Metrics Scraping)|      |   (Dashboards)   |
+--------------------+      +------------------+
```

## CI/CD Workflow

```
Developer git push
        |
        v
GitHub Webhook fires → Jenkins triggered
        |
        +---> Stage 1: Git Checkout
        |
        +---> Stage 2: NPM Install (Frontend)
        |
        +---> Stage 3: NPM Install (Backend)
        |
        +---> Stage 4: React Build (npm run build)
        |
        +---> Stage 5: Backend Lint/Audit
        |
        +---> Stage 6: SonarQube Static Analysis
        |
        +---> Stage 7: SonarQube Quality Gate (PASS/FAIL)
        |
        +---> Stage 8: Docker Build Frontend Image
        |
        +---> Stage 9: Docker Build Backend Image
        |
        +---> Stage 10: Trivy FS Scan
        |
        +---> Stage 11: Trivy Image Scan (Frontend)
        |
        +---> Stage 12: Trivy Image Scan (Backend)
        |
        +---> Stage 13: Push Images to ACR
        |
        +---> Stage 14: Deploy to AKS (kubectl apply)
        |
        +---> Stage 15: OWASP ZAP DAST Scan
        |
        +---> Stage 16: Deployment Verification
        |
        +---> Stage 17: Archive Security Reports
        |
        v
   Pipeline Result: SUCCESS / FAILURE
        |
        v
   Metrics flowing to Prometheus → Grafana
```

---

# 2. AZURE FOR STUDENTS SETUP

## What is Azure for Students?

Azure for Students is a free offer from Microsoft that gives computer science students **$100 USD in Azure credits** for 12 months with **no credit card required**. It also includes free access to many popular developer tools.

## Step 1: Activate Azure for Students

### Go to the activation page

Open your browser and navigate to:
```
https://azure.microsoft.com/en-us/free/students/
```

Click **"Start free"**.

### Sign in with your university email

- Use your **college/university email address** (e.g., `yourname@college.edu`)
- Microsoft verifies student status automatically through your institution
- If automatic verification fails, you can verify via SheerID using your student ID card

### Verification process

```
Step 1: Enter university email
Step 2: Microsoft checks Azure Active Directory for student status
Step 3: If not found → SheerID verification page appears
Step 4: Upload student ID or enrollment certificate
Step 5: Wait 24-72 hours for manual verification
Step 6: Receive confirmation email
Step 7: $100 credit activated in your subscription
```

### Confirm activation

After activation, go to:
```
https://portal.azure.com
```

Navigate to: **Cost Management + Billing → Azure credits**

You should see:
```
Available credits: $100.00 USD
Expiry date: [12 months from activation]
```

---

# 3. COST ESTIMATION AND BUDGET PLANNING

## Complete Resource Cost Table

| Resource | Type | SKU | Est. Hourly | Est. Daily | Est. Monthly | Notes |
|---|---|---|---|---|---|---|
| Resource Group | Free | — | $0.00 | $0.00 | $0.00 | No cost |
| ACR | Basic Tier | Basic | $0.007 | $0.17 | $5.00 | 10GB storage included |
| AKS Control Plane | Free tier | — | $0.00 | $0.00 | $0.00 | Free on AKS |
| AKS Node 1 | Standard_B2s | 2 vCPU / 4GB | $0.048 | $1.15 | $34.85 | Main cost |
| OS Disk | Standard SSD | 30GB | $0.002 | $0.05 | $1.50 | Per node |
| Public IP (Ingress) | Static | — | $0.004 | $0.10 | $3.00 | For Ingress |
| Load Balancer | Basic | — | $0.005 | $0.12 | $3.65 | AKS managed |
| Jenkins VM | B2s (self-hosted) | 2 vCPU / 4GB | $0.048 | $1.15 | $34.85 | STOP when not in use |
| **TOTAL (running 24x7)** | | | **~$0.11** | **~$2.74** | **~$82.85** | |

## Budget Allocation (USD $100)

```
AKS Cluster (30 days)         : ~$36.35
Jenkins VM (demo period only) : ~$10.00  (assumed 7 days)
ACR (30 days)                 : ~$5.00
Networking / IPs              : ~$6.65
Buffer / Testing              : ~$12.00
                                --------
TOTAL ESTIMATED               : ~$70.00
REMAINING BUFFER              : ~$30.00
```

## CRITICAL: Cost Control Rules

```
RULE 1: STOP AKS node pool when not testing (saves ~$1.15/day)
RULE 2: STOP Jenkins VM overnight (saves ~$0.048/hour)
RULE 3: DELETE everything after project demonstration
RULE 4: Use Standard_B2s (burstable) NOT Standard_D2s_v3
RULE 5: Use Basic ACR NOT Standard
RULE 6: Single node AKS NOT multi-node for demo
```

---

# 4. RESOURCE GROUP CREATION

## What is a Resource Group?

A Resource Group in Azure is a **logical container** that holds all related Azure resources for a project. Think of it like a folder on your computer — all your project files (VM, database, network) go inside one folder.

**Why create one?**
- Easy to manage all resources together
- Can delete all resources at once by deleting the group
- Better cost tracking
- Logical organization

## Create the Resource Group

```bash
az group create \
  --name mern-cicd-rg \
  --location eastus
```

### Command Breakdown

| Part | Meaning |
|---|---|
| `az group create` | Azure CLI command to create a resource group |
| `--name mern-cicd-rg` | Name for your resource group (you choose this) |
| `--location eastus` | Azure datacenter region — East US is cheapest for students |

### Expected Output

```json
{
  "id": "/subscriptions/YOUR-SUBSCRIPTION-ID/resourceGroups/mern-cicd-rg",
  "location": "eastus",
  "managedBy": null,
  "name": "mern-cicd-rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

### Common Errors

| Error | Cause | Fix |
|---|---|---|
| `az: command not found` | Azure CLI not installed | Install Azure CLI first (Section 5) |
| `AuthorizationFailed` | Not logged in | Run `az login` first |
| `InvalidResourceGroupName` | Special characters in name | Use only letters, numbers, hyphens |
| `LocationNotAvailableForSubscription` | Region not available for students | Use `eastus` or `westus2` |

### Safe Deletion Command

```bash
# WARNING: This deletes ALL resources inside this group
az group delete --name mern-cicd-rg --yes --no-wait
```

---

# 5. AZURE CLI INSTALLATION

## What is Azure CLI?

Azure CLI (Command Line Interface) is a tool that lets you manage Azure resources from your terminal/command prompt. Instead of clicking through the Azure portal website, you type commands. This is faster, scriptable, and repeatable.

## Installation on Ubuntu/Debian Linux (Recommended for Jenkins VM)

```bash
# Step 1: Update your package list
# This downloads the latest list of available software packages
sudo apt-get update

# Step 2: Install dependencies that Azure CLI needs
sudo apt-get install -y ca-certificates curl apt-transport-https lsb-release gnupg

# Step 3: Add Microsoft's signing key to your system
# This verifies that the software you download is genuinely from Microsoft
curl -sL https://packages.microsoft.com/keys/microsoft.asc |
    gpg --dearmor |
    sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

# Step 4: Add the Azure CLI software repository
# This tells apt-get WHERE to download Azure CLI from
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" |
    sudo tee /etc/apt/sources.list.d/azure-cli.list

# Step 5: Update package list again (now includes Azure CLI repo)
sudo apt-get update

# Step 6: Install Azure CLI
sudo apt-get install -y azure-cli
```

### Verify Installation

```bash
az version
```

Expected output:
```json
{
  "azure-cli": "2.61.0",
  "azure-cli-core": "2.61.0",
  "azure-cli-telemetry": "1.1.0",
  "extensions": {}
}
```

## Installation on Windows

```powershell
# Open PowerShell as Administrator
# Run this single command:
winget install -e --id Microsoft.AzureCLI
```

Or download the MSI installer from:
```
https://aka.ms/installazurecliwindows
```

## Installation on macOS

```bash
brew update && brew install azure-cli
```

---

# 6. AZURE LOGIN

## Login to Azure

```bash
az login
```

### What Happens When You Run This

1. A browser window opens automatically
2. You log in with your university Microsoft account
3. The terminal receives an authentication token
4. You are now authenticated for ~60 minutes

### Expected Output in Terminal

```
A web browser has been opened at https://login.microsoftonline.com/...
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "YOUR-TENANT-ID",
    "id": "YOUR-SUBSCRIPTION-ID",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Azure for Students",
    "state": "Enabled",
    "tenantId": "YOUR-TENANT-ID",
    "user": {
      "name": "yourname@college.edu",
      "type": "user"
    }
  }
]
```

### If Running on a Headless Server (No Browser)

```bash
az login --use-device-code
```

This shows a code like:
```
To sign in, use a web browser to open the page https://microsoft.com/devicelogin 
and enter the code ABCD1234 to authenticate.
```

Open the URL on your laptop browser, enter the code, and log in.

### Set the Active Subscription

```bash
# List all subscriptions
az account list --output table

# Set the Azure for Students subscription as active
az account set --subscription "Azure for Students"
```

---

# 7. AZURE CONTAINER REGISTRY (ACR) CREATION

## What is ACR?

Azure Container Registry (ACR) is a private Docker image registry hosted on Azure. It is like Docker Hub but private — only your AKS cluster and authorized users can pull your images.

**Why use ACR instead of Docker Hub?**
- Private (your images are not public)
- Integrated with AKS — no separate login needed
- Built-in vulnerability scanning
- Geo-replication available
- Lower latency since both ACR and AKS are in Azure

## Create ACR

```bash
az acr create \
  --resource-group mern-cicd-rg \
  --name merncicdacr \
  --sku Basic \
  --location eastus
```

### Command Breakdown

| Part | Meaning |
|---|---|
| `--resource-group mern-cicd-rg` | Which resource group to put ACR in |
| `--name merncicdacr` | ACR name — must be globally unique, only lowercase letters and numbers |
| `--sku Basic` | Basic tier: cheapest, 10GB storage, single region |
| `--location eastus` | Same region as your AKS (important for speed) |

### Expected Output

```json
{
  "adminUserEnabled": false,
  "creationDate": "2024-01-01T00:00:00+00:00",
  "id": "/subscriptions/.../resourceGroups/mern-cicd-rg/providers/Microsoft.ContainerRegistry/registries/merncicdacr",
  "location": "eastus",
  "loginServer": "merncicdacr.azurecr.io",
  "name": "merncicdacr",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

**Your login server URL is: `merncicdacr.azurecr.io`**
Save this — you will use it throughout the project.

## Enable Admin Access (for Jenkins to push images)

```bash
az acr update \
  --name merncicdacr \
  --resource-group mern-cicd-rg \
  --admin-enabled true
```

## Get ACR Credentials (for Jenkins)

```bash
az acr credential show \
  --name merncicdacr \
  --resource-group mern-cicd-rg
```

Expected output:
```json
{
  "passwords": [
    {
      "name": "password",
      "value": "YOUR-ACR-PASSWORD-1"
    },
    {
      "name": "password2",
      "value": "YOUR-ACR-PASSWORD-2"
    }
  ],
  "username": "merncicdacr"
}
```

**Save the username and password1 — you will add these to Jenkins credentials.**

## Cost Details

| Resource | Cost |
|---|---|
| ACR Basic tier | $5.00/month |
| Storage (first 10GB) | Included |
| Pull operations | $0.004 per 10,000 |
| **Running total after ACR** | **~$5.00/month** |

### Safe Deletion

```bash
az acr delete \
  --name merncicdacr \
  --resource-group mern-cicd-rg \
  --yes
```

---

# 8. AKS CLUSTER CREATION

## What is AKS?

Azure Kubernetes Service (AKS) is a managed Kubernetes cluster. Kubernetes is an orchestration platform that automatically manages your Docker containers — starting them, restarting them if they crash, scaling them up/down, and routing traffic to them.

**Why AKS?**
- Microsoft manages the control plane (free of charge)
- Integrated with Azure networking, ACR, monitoring
- Supports RBAC, network policies, secrets management

## Recommended Configuration for $100 Budget

| Parameter | Value | Reason |
|---|---|---|
| Node SKU | Standard_B2s | Cheapest burstable VM with 2 vCPU / 4GB RAM |
| Node count | 1 | Minimum for demo; add 2nd only if needed |
| Kubernetes version | 1.30.x (latest stable) | Supported, stable |
| OS disk | 30GB Standard SSD | Minimum size, cheapest |
| Network plugin | kubenet | Simpler, cheaper than Azure CNI |
| Load balancer | Basic | Cheapest, sufficient for demo |

## Create the AKS Cluster

```bash
az aks create \
  --resource-group mern-cicd-rg \
  --name mern-cicd-aks \
  --node-count 1 \
  --node-vm-size Standard_B2s \
  --node-osdisk-size 30 \
  --node-osdisk-type Managed \
  --network-plugin kubenet \
  --generate-ssh-keys \
  --attach-acr merncicdacr \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 2 \
  --kubernetes-version 1.30.0 \
  --tier free \
  --location eastus
```

### Command Breakdown (Every Flag Explained)

| Flag | Meaning |
|---|---|
| `--node-count 1` | Start with 1 node (VM) to save money |
| `--node-vm-size Standard_B2s` | VM size: 2 vCPU, 4GB RAM — cheapest that works |
| `--node-osdisk-size 30` | 30GB disk — minimum, saves money |
| `--node-osdisk-type Managed` | Azure manages the disk (required) |
| `--network-plugin kubenet` | Simple networking — cheaper than Azure CNI |
| `--generate-ssh-keys` | Creates SSH keys so you can access nodes |
| `--attach-acr merncicdacr` | Automatically allows AKS to pull from your ACR |
| `--enable-cluster-autoscaler` | Automatically adds/removes nodes based on load |
| `--min-count 1` | Never go below 1 node |
| `--max-count 2` | Maximum 2 nodes (cost safety limit) |
| `--tier free` | Free control plane (no charge for Kubernetes master) |

> **Note:** This command takes 5-10 minutes to complete. Do not interrupt it.

### Expected Output (partial)

```json
{
  "agentPoolProfiles": [
    {
      "count": 1,
      "maxCount": 2,
      "minCount": 1,
      "mode": "System",
      "name": "nodepool1",
      "nodeImageVersion": "AKSUbuntu-2204gen2containerd-202401.01.0",
      "osDiskSizeGb": 30,
      "vmSize": "Standard_B2s"
    }
  ],
  "dnsPrefix": "mern-cicd-aks-dns",
  "fqdn": "mern-cicd-aks-dns-xxxx.hcp.eastus.azmk8s.io",
  "kubernetesVersion": "1.30.0",
  "name": "mern-cicd-aks",
  "provisioningState": "Succeeded"
}
```

## Stop AKS (to save credits when not in use)

```bash
# STOP (pauses billing for node VMs — you still pay for disks and IPs)
az aks stop \
  --name mern-cicd-aks \
  --resource-group mern-cicd-rg

# START again when needed
az aks start \
  --name mern-cicd-aks \
  --resource-group mern-cicd-rg
```

## AKS Cost Summary

| Metric | Value |
|---|---|
| Control plane | FREE (tier: free) |
| Standard_B2s per hour | $0.048 |
| Standard_B2s per day | $1.15 |
| Standard_B2s per month | $34.85 |
| OS disk 30GB per month | $1.20 |
| **AKS total per month** | **~$36.05** |
| **Running total** | **~$41.05/month** |

## Safe AKS Deletion Commands

```bash
# Delete just the AKS cluster
az aks delete \
  --name mern-cicd-aks \
  --resource-group mern-cicd-rg \
  --yes \
  --no-wait

# OR delete everything (Resource Group)
az group delete \
  --name mern-cicd-rg \
  --yes \
  --no-wait
```

---

# 9. KUBECTL INSTALLATION

## What is kubectl?

`kubectl` is the command-line tool for interacting with Kubernetes clusters. You use it to deploy applications, inspect pods, view logs, and manage cluster resources.

## Install kubectl on Ubuntu/Debian

```bash
# Step 1: Download kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Step 2: Make it executable
chmod +x kubectl

# Step 3: Move to system path so you can run it from anywhere
sudo mv kubectl /usr/local/bin/

# Step 4: Verify installation
kubectl version --client
```

Expected output:
```
Client Version: v1.30.0
Kustomize Version: v5.0.4-0
```

## Configure kubectl to Connect to Your AKS Cluster

```bash
az aks get-credentials \
  --resource-group mern-cicd-rg \
  --name mern-cicd-aks
```

### What This Command Does

This command downloads the Kubernetes configuration (called a "kubeconfig") from Azure and saves it to `~/.kube/config`. This config file contains:
- The API server URL of your AKS cluster
- Authentication certificates
- Context (cluster + user + namespace combination)

After running this, `kubectl` automatically connects to your AKS cluster.

### Verify Connection

```bash
kubectl get nodes
```

Expected output:
```
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-12345678-vmss000000   Ready    agent   5m      v1.30.0
```

If you see `Ready`, your cluster is working correctly.

---

# 10. DOCKER INSTALLATION

## What is Docker?

Docker is a platform that packages your application and all its dependencies into a **container** — a lightweight, portable, self-contained unit. A container runs the same way on your laptop, on a server, or in the cloud.

**Analogy:** Think of Docker as a shipping container. The contents (your app) are the same no matter which ship (server) carries it.

## Install Docker on Ubuntu/Debian

```bash
# Step 1: Remove any old Docker versions
sudo apt-get remove -y docker docker-engine docker.io containerd runc

# Step 2: Update package index
sudo apt-get update

# Step 3: Install required packages
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Step 4: Add Docker's official GPG key (verifies authenticity)
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step 5: Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 6: Update package index (now includes Docker repo)
sudo apt-get update

# Step 7: Install Docker Engine
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Step 8: Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Step 9: Add your user to docker group (so you don't need sudo)
sudo usermod -aG docker $USER

# Step 10: Apply group changes (or logout and login)
newgrp docker
```

### Verify Docker Installation

```bash
docker --version
docker run hello-world
```

Expected:
```
Docker version 26.1.0, build a07abb9
Hello from Docker!
```

---

# 11. JENKINS INSTALLATION

## What is Jenkins?

Jenkins is an open-source automation server. It automatically runs your CI/CD pipeline whenever code is pushed to GitHub. Think of Jenkins as a robot that does all your build, test, scan, and deploy steps automatically.

## Option A: Install Jenkins on Azure VM (Recommended for this project)

### Create the Jenkins VM on Azure

```bash
az vm create \
  --resource-group mern-cicd-rg \
  --name jenkins-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard
```

### Open Port 8080 for Jenkins

```bash
az vm open-port \
  --resource-group mern-cicd-rg \
  --name jenkins-vm \
  --port 8080
```

### Get the Public IP

```bash
az vm show \
  --resource-group mern-cicd-rg \
  --name jenkins-vm \
  --show-details \
  --query [publicIps] \
  --output tsv
```

### SSH into the VM

```bash
ssh azureuser@<PUBLIC-IP>
```

### Install Java (Jenkins requires Java)

```bash
# Jenkins requires Java 17 or Java 21
sudo apt-get update
sudo apt-get install -y fontconfig openjdk-17-jre

# Verify Java
java -version
# Expected: openjdk version "17.0.x"
```

### Install Jenkins

```bash
# Step 1: Add Jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Step 2: Add Jenkins repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Step 3: Update package list
sudo apt-get update

# Step 4: Install Jenkins
sudo apt-get install -y jenkins

# Step 5: Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Step 6: Check Jenkins status
sudo systemctl status jenkins
```

Expected status output:
```
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled)
     Active: active (running) since Mon 2024-01-01 00:00:00 UTC; 5s ago
```

### Get Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

This shows a random password like: `a3f2b1c4d5e6f7890abc1234def56789`

### Access Jenkins Web UI

Open browser and go to:
```
http://<JENKINS-VM-PUBLIC-IP>:8080
```

You will see the Jenkins setup wizard.

---

# 12. JENKINS CONFIGURATION

## Initial Setup Wizard

1. Enter the initial admin password from the previous step
2. Click **"Install suggested plugins"** — wait 3-5 minutes
3. Create your admin user:
   - Username: `admin`
   - Password: `YourSecurePassword123`
   - Full name: Your Name
   - Email: your@email.com
4. Configure Jenkins URL: `http://<JENKINS-VM-PUBLIC-IP>:8080/`
5. Click **"Save and Finish"**
6. Click **"Start using Jenkins"**

## Install Required Jenkins Plugins

Navigate to: **Manage Jenkins → Plugins → Available Plugins**

Search and install each of these:

```
1. Docker Pipeline          - Allows Docker commands in Jenkinsfile
2. Docker Commons           - Docker integration support
3. Azure Credentials        - Store Azure credentials securely
4. Kubernetes               - Deploy to Kubernetes from Jenkins
5. SonarQube Scanner        - Run SonarQube analysis
6. OWASP Markup Formatter   - Format ZAP reports
7. HTML Publisher           - Publish HTML security reports
8. Pipeline Utility Steps   - Utility functions for pipelines
9. GitHub                   - GitHub webhook integration
10. GitHub Branch Source    - Branch-based pipelines
11. Credentials Binding     - Bind credentials to env variables
12. Workspace Cleanup       - Clean workspace before builds
```

After selecting all, click **"Install"** and check **"Restart Jenkins when installation is complete"**.

## Configure Jenkins Credentials

Navigate to: **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

### Credential 1: ACR Credentials

```
Kind: Username with password
Username: merncicdacr
Password: <your-acr-password-from-step-7>
ID: acr-credentials
Description: Azure Container Registry Login
```

### Credential 2: Azure Service Principal (for AKS deployment)

First, create a Service Principal on Azure:

```bash
az ad sp create-for-rbac \
  --name "jenkins-sp" \
  --role Contributor \
  --scopes /subscriptions/<YOUR-SUBSCRIPTION-ID>/resourceGroups/mern-cicd-rg
```

Output:
```json
{
  "appId": "YOUR-APP-ID",
  "displayName": "jenkins-sp",
  "password": "YOUR-SP-SECRET",
  "tenant": "YOUR-TENANT-ID"
}
```

In Jenkins, add:
```
Kind: Secret text
Secret: YOUR-SP-SECRET
ID: azure-sp-secret
Description: Azure Service Principal Secret
```

Also add:
```
Kind: Secret text
Secret: YOUR-APP-ID
ID: azure-sp-appid
```

```
Kind: Secret text
Secret: YOUR-TENANT-ID
ID: azure-tenant-id
```

### Credential 3: SonarQube Token

(Add after configuring SonarQube in Section 14)
```
Kind: Secret text
Secret: <sonarqube-token>
ID: sonarqube-token
Description: SonarQube Authentication Token
```

## Configure SonarQube in Jenkins

Navigate to: **Manage Jenkins → Configure System**

Scroll down to **SonarQube servers**:
```
Name: SonarQube
Server URL: http://localhost:9000
Server authentication token: (select sonarqube-token credential)
```

## Configure GitHub Webhook

In your GitHub repository:
1. Go to **Settings → Webhooks → Add webhook**
2. Payload URL: `http://<JENKINS-VM-PUBLIC-IP>:8080/github-webhook/`
3. Content type: `application/json`
4. Events: Select **"Just the push event"**
5. Click **"Add webhook"**

Now every `git push` automatically triggers your Jenkins pipeline.

---

# 13. SONARQUBE INSTALLATION

## What is SonarQube?

SonarQube is a **Static Application Security Testing (SAST)** tool. It scans your source code without running it, looking for:
- Security vulnerabilities (SQL injection, XSS, etc.)
- Code quality issues (bugs, code smells)
- Test coverage
- Duplicate code

**Why SAST?** It catches security issues early in development, before the code is deployed.

## Install SonarQube on the Jenkins VM

### Prerequisites

```bash
# SonarQube requires Java 17 (already installed)
# SonarQube requires at least 2GB RAM (Standard_B2s has 4GB — sufficient)

# Increase virtual memory limit (required by SonarQube's Elasticsearch)
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072

# Make these settings permanent
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf
echo "fs.file-max=131072" | sudo tee -a /etc/sysctl.conf
```

### Install SonarQube as Docker Container (Simplest Method)

```bash
# Create a dedicated network
docker network create sonarnet

# Run SonarQube container
docker run -d \
  --name sonarqube \
  --network sonarnet \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  --restart always \
  sonarqube:lts-community
```

### What Each Docker Flag Means

| Flag | Meaning |
|---|---|
| `-d` | Run in background (detached mode) |
| `--name sonarqube` | Name the container |
| `--network sonarnet` | Connect to our custom network |
| `-p 9000:9000` | Map port 9000 on host to port 9000 in container |
| `-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true` | Skip Elasticsearch checks (needed on small VMs) |
| `--restart always` | Restart automatically if the container crashes |
| `sonarqube:lts-community` | Use the free community edition, LTS version |

### Wait for SonarQube to Start

```bash
# Watch the logs (press Ctrl+C to stop watching)
docker logs -f sonarqube

# Wait until you see: "SonarQube is operational"
```

### Open SonarQube Port on Azure VM

```bash
az vm open-port \
  --resource-group mern-cicd-rg \
  --name jenkins-vm \
  --port 9000
```

### Access SonarQube

```
URL: http://<JENKINS-VM-PUBLIC-IP>:9000
Default username: admin
Default password: admin
```

> You will be forced to change the password on first login. Use something memorable like `Admin@12345`.

---

# 14. SONARQUBE CONFIGURATION

## Create a SonarQube Project

1. Log in to SonarQube at `http://<IP>:9000`
2. Click **"Create project manually"**
3. Project key: `mern-cicd-project`
4. Project name: `MERN CICD Pipeline`
5. Main branch: `main`
6. Click **"Set up"**
7. Select **"With Jenkins"** for CI/CD integration
8. Follow the setup — select **"GitHub"** as the repository hosting
9. Click **"Configure analysis"**

## Generate SonarQube Token

1. Go to **My Account → Security → Generate Token**
2. Token name: `jenkins-token`
3. Token type: `Project Analysis Token`
4. Project: `mern-cicd-project`
5. Expiry: `No expiration` (for project duration)
6. Click **"Generate"**
7. **Copy the token immediately** — it will not be shown again

The token looks like: `sqp_abc123def456...`

Add this token to Jenkins credentials as described in Section 12.

## Configure Quality Gate

A Quality Gate is a set of rules. If the code does not meet these conditions, the pipeline fails.

1. Go to **Quality Gates → Create**
2. Name: `MERN Security Gate`
3. Add conditions:
   - Security Rating: ≥ A
   - Reliability Rating: ≥ B
   - Coverage: ≥ 70%
   - Duplicated Lines: ≤ 10%
4. Click **"Set as Default"**

---

# 15. TRIVY INSTALLATION

## What is Trivy?

Trivy is an open-source **vulnerability scanner** by Aqua Security. It scans:
- Docker images for known CVEs (Common Vulnerabilities and Exposures)
- Filesystem for vulnerable packages
- Kubernetes manifests for misconfigurations
- Git repositories for secrets

**Why Trivy?** It is fast, comprehensive, and works without an account or license.

## Install Trivy on Jenkins VM

```bash
# Step 1: Add Trivy repository
sudo apt-get install -y wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
    gpg --dearmor | \
    sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
    https://aquasecurity.github.io/trivy-repo/deb \
    $(lsb_release -sc) main" | \
    sudo tee -a /etc/apt/sources.list.d/trivy.list

# Step 2: Install
sudo apt-get update
sudo apt-get install -y trivy

# Step 3: Verify
trivy --version
```

Expected: `Version: 0.50.x`

## Test Trivy

```bash
# Scan a test image
trivy image nginx:latest

# Scan current directory
trivy fs .
```

---

# 16. OWASP ZAP INSTALLATION

## What is OWASP ZAP?

OWASP ZAP (Zed Attack Proxy) is a **Dynamic Application Security Testing (DAST)** tool. Unlike SonarQube which reads source code, ZAP **actually runs your application** and attacks it like a hacker would, looking for runtime vulnerabilities like:
- SQL Injection
- Cross-Site Scripting (XSS)
- Authentication bypasses
- Security header misconfigurations

## Install OWASP ZAP

```bash
# Install ZAP as a Docker container (easiest method)
docker pull ghcr.io/zaproxy/zaproxy:stable

# Verify
docker images | grep zap
```

## Test ZAP Works

```bash
docker run --rm ghcr.io/zaproxy/zaproxy:stable zap.sh -version
```

ZAP is run during the Jenkins pipeline against the deployed application — you do not need to configure it separately.

---

# 17. DOCKER FILES — FRONTEND AND BACKEND

## Understanding Docker Images and Layers

A Docker image is built from a `Dockerfile` — a set of instructions. Each instruction creates a **layer**. Layers are cached, so unchanged layers do not need to be rebuilt.

**Multi-stage builds** are used to keep the final image small. The build stage compiles code; the production stage contains only the compiled output.

---

## frontend/Dockerfile

```dockerfile
# ============================================================
# STAGE 1: BUILD STAGE
# Purpose: Install dependencies and compile the React app
# This stage is temporary — not included in final image
# ============================================================

# Use Node.js 20 Alpine as the build environment
# Alpine is a minimal Linux distro — makes image smaller
# node:20-alpine = Node.js 20 on Alpine Linux (~180MB vs ~900MB for full Node)
FROM node:20-alpine AS builder

# Set the working directory inside the container
# All subsequent commands run from this directory
WORKDIR /app

# Copy package.json and package-lock.json FIRST
# Why? Docker caches layers. If package.json hasn't changed,
# Docker reuses the cached node_modules from a previous build
# This makes builds MUCH faster
COPY package.json package-lock.json ./

# Install all dependencies defined in package.json
# --ci flag: uses exact versions from package-lock.json (reproducible builds)
# --legacy-peer-deps: handles peer dependency conflicts in React projects
RUN npm ci --legacy-peer-deps

# Copy the rest of the source code
# Done AFTER npm install so dependency installation is cached separately
COPY . .

# Build the React application for production
# Creates optimized static files in /app/build
# These are HTML, CSS, and JS files — no Node.js needed to serve them
RUN npm run build

# ============================================================
# STAGE 2: PRODUCTION STAGE
# Purpose: Serve the built React app using Nginx
# Only this stage ends up in the final Docker image
# ============================================================

# Use Nginx Alpine — a lightweight web server
# This image is only ~23MB — perfect for serving static files
FROM nginx:alpine

# Copy built React files from the builder stage
# The /app/build directory contains: index.html, static/css/, static/js/
# We copy them to Nginx's default web root directory
COPY --from=builder /app/build /usr/share/nginx/html

# Copy custom Nginx configuration
# This is needed to handle React Router (client-side routing)
# Without this, refreshing any page except "/" gives 404
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80 — this is the port Nginx listens on
# This is documentation only — it doesn't actually open a port on the host
EXPOSE 80

# Start Nginx in the foreground
# "daemon off" keeps Nginx running in foreground (required for Docker)
CMD ["nginx", "-g", "daemon off;"]
```

## frontend/nginx.conf

```nginx
# Nginx configuration for React Single Page Application
# This file must be placed at /etc/nginx/conf.d/default.conf

server {
    # Listen on port 80 for HTTP traffic
    listen 80;
    
    # Accept requests for any hostname
    server_name _;
    
    # Root directory where the React build files are located
    root /usr/share/nginx/html;
    
    # Default file to serve
    index index.html;
    
    # Handle React Router (client-side routing)
    # When a user navigates to /dashboard or /about, 
    # those routes exist only in React, not on disk
    # This rule: "if the file doesn't exist, serve index.html"
    # React Router then handles the routing in the browser
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # API proxy: Forward /api/* requests to the backend service
    # This avoids CORS issues — the browser thinks everything comes from one server
    location /api/ {
        # "backend-service" is the Kubernetes Service name for our Node.js backend
        proxy_pass http://backend-service:5000/;
        
        # Pass the original request headers to the backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Serve static assets with caching headers
    # CSS, JS, images can be cached for 1 year since their filenames include content hash
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## frontend/.dockerignore

```
# .dockerignore file for the React frontend
# Tells Docker which files to EXCLUDE when building the image
# This reduces build time and image size

# Node modules — these are installed fresh inside the container
# They can be hundreds of MB and contain platform-specific binaries
node_modules/

# Build output — we'll build this inside the container
build/
dist/

# Environment files — NEVER include .env in Docker images
# Use Kubernetes ConfigMaps and Secrets instead
.env
.env.local
.env.development
.env.production
.env.test

# Git history — not needed in the image
.git/
.gitignore

# Documentation
README.md
CHANGELOG.md
docs/

# Test files — not needed in production
**/*.test.js
**/*.spec.js
**/__tests__/
coverage/

# Editor configuration
.vscode/
.idea/
*.swp
*.swo

# OS generated files
.DS_Store
Thumbs.db

# Log files
*.log
npm-debug.log*

# TypeScript cache
*.tsbuildinfo

# ESLint cache
.eslintcache
```

---

## backend/Dockerfile

```dockerfile
# ============================================================
# STAGE 1: DEPENDENCY INSTALLATION STAGE
# Purpose: Install only production dependencies
# ============================================================

FROM node:20-alpine AS deps

# Set working directory
WORKDIR /app

# Copy only package files first (for layer caching)
COPY package.json package-lock.json ./

# Install ONLY production dependencies (skip devDependencies)
# --omit=dev: excludes development-only packages (testing tools, etc.)
# This reduces the final image size significantly
RUN npm ci --omit=dev --legacy-peer-deps

# ============================================================
# STAGE 2: PRODUCTION STAGE
# Purpose: Run the Node.js backend server
# ============================================================

FROM node:20-alpine AS production

# Create a non-root user for security
# Running as root inside a container is a security risk
# If an attacker escapes the container, they would have root on the host
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001 -G nodejs

# Set working directory
WORKDIR /app

# Copy production dependencies from deps stage
# We copy only node_modules, not dev dependencies
COPY --from=deps /app/node_modules ./node_modules

# Copy application source code
COPY . .

# Change ownership of all files to the non-root user
RUN chown -R nodeuser:nodejs /app

# Switch to non-root user
# All subsequent operations run as 'nodeuser', not root
USER nodeuser

# Expose port 5000 (or whatever port your Node.js app uses)
# Change this to match your backend's PORT configuration
EXPOSE 5000

# Health check: Docker will periodically run this command
# If it fails 3 times, the container is marked unhealthy
# Kubernetes uses this to decide if traffic should be sent to this pod
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:5000/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1) })"

# Start the Node.js application
# "node server.js" is more efficient than "npm start" (skips npm overhead)
CMD ["node", "server.js"]
```

## backend/.dockerignore

```
# Node modules — reinstalled inside container
node_modules/

# Environment files — use Kubernetes Secrets instead
.env
.env.local
.env.development
.env.production
.env.test

# Git
.git/
.gitignore

# Documentation
README.md
docs/

# Test files
**/*.test.js
**/*.spec.js
**/__tests__/
coverage/

# Logs
*.log
logs/

# Editor files
.vscode/
.idea/

# OS files
.DS_Store

# Build artifacts
dist/
build/

# Docker files themselves (no need to copy these into image)
Dockerfile
.dockerignore
docker-compose.yml
```

---

## docker-compose.yml (For Local Development and Testing)

```yaml
# Docker Compose file for local development
# Version 3.8 supports all modern Docker features
version: '3.8'

# ============================================================
# SERVICES: Each service is a container
# ============================================================
services:

  # --------------------------------------------------------
  # Frontend Service: React Application
  # --------------------------------------------------------
  frontend:
    # Build from the Dockerfile in ./frontend directory
    build:
      context: ./frontend
      dockerfile: Dockerfile
    
    # Name of the container (optional but helpful)
    container_name: mern-frontend
    
    # Map host port 3000 to container port 80
    # Access app at http://localhost:3000
    ports:
      - "3000:80"
    
    # Environment variables passed into the container
    # REACT_APP_API_URL tells React where the backend API is
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    
    # This service starts only after the backend is ready
    depends_on:
      backend:
        condition: service_healthy
    
    # Connect to the custom network
    networks:
      - mern-network
    
    # Automatically restart if the container crashes
    restart: unless-stopped

  # --------------------------------------------------------
  # Backend Service: Node.js API Server
  # --------------------------------------------------------
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    
    container_name: mern-backend
    
    # Map host port 5000 to container port 5000
    ports:
      - "5000:5000"
    
    environment:
      # These variables configure the backend
      # In production (Kubernetes), these come from ConfigMaps and Secrets
      - NODE_ENV=production
      - PORT=5000
      # MongoDB connection string — use the service name 'mongo' as hostname
      - MONGODB_URI=mongodb://mongo:27017/merndb
    
    # Define health check so frontend knows when backend is ready
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:5000/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1) })"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    
    depends_on:
      mongo:
        condition: service_healthy
    
    networks:
      - mern-network
    
    restart: unless-stopped

  # --------------------------------------------------------
  # Database Service: MongoDB
  # --------------------------------------------------------
  mongo:
    # Use official MongoDB 7.0 image
    image: mongo:7.0
    
    container_name: mern-mongo
    
    # Map host port 27017 to container port 27017
    ports:
      - "27017:27017"
    
    environment:
      # Set MongoDB admin credentials
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: mongopassword
      MONGO_INITDB_DATABASE: merndb
    
    # Persist MongoDB data even if container is removed
    # Without this, all data is lost when container stops
    volumes:
      - mongo-data:/data/db
    
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    
    networks:
      - mern-network
    
    restart: unless-stopped

# ============================================================
# VOLUMES: Persistent storage
# ============================================================
volumes:
  # Named volume for MongoDB data persistence
  # Stored at /var/lib/docker/volumes/mongo-data
  mongo-data:
    driver: local

# ============================================================
# NETWORKS: Custom network for container communication
# ============================================================
networks:
  # Containers on the same network can communicate using service names
  # e.g., backend can reach MongoDB using "mongo" as the hostname
  mern-network:
    driver: bridge
```

---

# 18. BUILD AND PUSH DOCKER IMAGES

## Build Images Locally (Manual Method — for testing)

```bash
# Navigate to project root
cd /path/to/your/project

# Login to ACR
az acr login --name merncicdacr

# Build frontend image
docker build \
  -t merncicdacr.azurecr.io/mern-frontend:latest \
  -t merncicdacr.azurecr.io/mern-frontend:v1.0 \
  ./frontend

# Build backend image
docker build \
  -t merncicdacr.azurecr.io/mern-backend:latest \
  -t merncicdacr.azurecr.io/mern-backend:v1.0 \
  ./backend

# Push frontend to ACR
docker push merncicdacr.azurecr.io/mern-frontend:latest
docker push merncicdacr.azurecr.io/mern-frontend:v1.0

# Push backend to ACR
docker push merncicdacr.azurecr.io/mern-backend:latest
docker push merncicdacr.azurecr.io/mern-backend:v1.0
```

## Verify Images in ACR

```bash
# List all repositories in your ACR
az acr repository list \
  --name merncicdacr \
  --output table

# List tags for frontend
az acr repository show-tags \
  --name merncicdacr \
  --repository mern-frontend \
  --output table
```

Expected output:
```
Name
---------
mern-backend
mern-frontend

Tags for mern-frontend:
Tag
------
latest
v1.0
```

---

# 19. KUBERNETES MANIFESTS

## Kubernetes Concepts (Quick Reference)

| Resource | What It Does |
|---|---|
| **Namespace** | Logical isolation — like folders for Kubernetes resources |
| **Deployment** | Declares how many pods to run and what image to use |
| **Pod** | The smallest unit — runs your Docker container |
| **Service** | Network endpoint that routes traffic to pods |
| **Ingress** | Routes external HTTP/HTTPS traffic to services |
| **ConfigMap** | Stores non-sensitive configuration (env vars) |
| **Secret** | Stores sensitive data (passwords, tokens) — base64 encoded |

---

## k8s/namespace.yaml

```yaml
# ============================================================
# NAMESPACE
# A namespace logically separates resources in Kubernetes.
# All our MERN app resources will live in this namespace.
# This prevents conflicts with other applications on the same cluster.
# ============================================================

apiVersion: v1           # The Kubernetes API version for this resource
kind: Namespace          # The type of resource we're creating
metadata:
  name: mern-app         # Name of the namespace — used in all other manifests
  labels:
    # Labels are key-value pairs for organization and filtering
    app: mern-app
    environment: production
    team: devops
```

---

## k8s/configmap.yaml

```yaml
# ============================================================
# CONFIGMAP
# Stores non-sensitive configuration data.
# These are injected into pods as environment variables.
# Never put passwords, tokens, or private keys in a ConfigMap.
# ============================================================

apiVersion: v1
kind: ConfigMap
metadata:
  name: mern-config        # Name used to reference this ConfigMap
  namespace: mern-app      # Must match our namespace
  labels:
    app: mern-app
data:
  # These key-value pairs become environment variables in the pod
  
  NODE_ENV: "production"
  # Tells Node.js to run in production mode (enables optimizations, disables debug)
  
  PORT: "5000"
  # The port number the backend API listens on
  
  FRONTEND_PORT: "80"
  # The port Nginx serves the React app on
  
  REACT_APP_API_URL: "http://backend-service:5000"
  # URL that the React frontend uses to call the backend API
  # "backend-service" is the Kubernetes Service name for the backend
  # This works because both are in the same cluster/namespace
  
  LOG_LEVEL: "info"
  # Application log verbosity level
  
  CORS_ORIGIN: "http://frontend-service"
  # CORS allowed origin for the backend API
```

---

## k8s/secret.yaml

```yaml
# ============================================================
# SECRET
# Stores sensitive data encoded in base64.
# IMPORTANT: base64 is NOT encryption — just encoding.
# For production, use Azure Key Vault with External Secrets Operator.
# For this project/demo, this is sufficient.
# ============================================================

apiVersion: v1
kind: Secret
metadata:
  name: mern-secrets
  namespace: mern-app
  labels:
    app: mern-app
type: Opaque   # "Opaque" means generic key-value secret (most common type)
data:
  # All values MUST be base64 encoded
  # To encode: echo -n "your-value" | base64
  # To decode: echo "encoded-value" | base64 --decode
  
  # MongoDB connection string
  # echo -n "mongodb://admin:mongopassword@mongo-service:27017/merndb" | base64
  MONGODB_URI: bW9uZ29kYjovL2FkbWluOm1vbmdvcGFzc3dvcmRAbW9uZ28tc2VydmljZToyNzAxNy9tZXJuZGI=
  
  # JWT Secret key for authentication tokens
  # echo -n "your-super-secret-jwt-key-change-this-in-production" | base64
  JWT_SECRET: eW91ci1zdXBlci1zZWNyZXQtand0LWtleS1jaGFuZ2UtdGhpcy1pbi1wcm9kdWN0aW9u
  
  # ACR credentials (for pulling images)
  # These are used by the imagePullSecret in Deployments
  # (Created separately using kubectl create secret docker-registry)

---
# ============================================================
# DOCKER REGISTRY SECRET FOR ACR
# Allows Kubernetes to pull images from your private ACR
# Run this command to create it:
#
# kubectl create secret docker-registry acr-secret \
#   --docker-server=merncicdacr.azurecr.io \
#   --docker-username=merncicdacr \
#   --docker-password=<ACR-PASSWORD> \
#   --namespace=mern-app
#
# We don't create it as a YAML file because the password would
# be stored in plain text in your Git repository.
# ============================================================
```

> **Important:** After applying secret.yaml, run this command to create the ACR pull secret:
```bash
kubectl create secret docker-registry acr-secret \
  --docker-server=merncicdacr.azurecr.io \
  --docker-username=merncicdacr \
  --docker-password=<YOUR-ACR-PASSWORD> \
  --namespace=mern-app
```

---

## k8s/deployment.yaml

```yaml
# ============================================================
# DEPLOYMENTS — Frontend and Backend
# A Deployment manages a set of identical pods.
# It ensures the desired number of replicas is always running.
# If a pod crashes, the Deployment automatically creates a new one.
# ============================================================

# ============================================================
# FRONTEND DEPLOYMENT
# ============================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment    # Unique name for this deployment
  namespace: mern-app
  labels:
    app: mern-frontend
    version: v1.0
    component: frontend
spec:
  # ---- REPLICAS ----
  replicas: 2   
  # Run 2 copies (pods) of the frontend
  # If one crashes, the other still serves traffic (high availability)
  # For demo with 1 AKS node and cost constraints, 2 is sufficient
  
  # ---- SELECTOR ----
  selector:
    matchLabels:
      app: mern-frontend   # This deployment manages pods with this label
  
  # ---- UPDATE STRATEGY ----
  strategy:
    type: RollingUpdate    # Deploy new version without downtime
    rollingUpdate:
      maxSurge: 1          # Maximum extra pods during update (25% of replicas)
      maxUnavailable: 0    # Zero pods down during update (zero-downtime deployment)
  
  # ---- POD TEMPLATE ----
  template:
    metadata:
      labels:
        app: mern-frontend  # Labels on each pod — must match selector above
        version: v1.0
    
    spec:
      # imagePullSecrets tells Kubernetes which secret to use to pull from ACR
      imagePullSecrets:
        - name: acr-secret  # The docker-registry secret we created earlier
      
      # ---- CONTAINERS ----
      containers:
        - name: frontend-container
          
          # Full image path in ACR
          # Jenkins will update this tag during each deployment
          image: merncicdacr.azurecr.io/mern-frontend:latest
          
          # Always pull the latest version — don't use cached image
          imagePullPolicy: Always
          
          ports:
            - containerPort: 80  # Port that Nginx listens on inside the container
          
          # ---- ENVIRONMENT VARIABLES ----
          envFrom:
            - configMapRef:
                name: mern-config  # Load all key-value pairs from our ConfigMap
          
          # ---- RESOURCE LIMITS ----
          # Requests: Minimum guaranteed resources for this container
          # Limits: Maximum resources this container can use
          # If a container exceeds memory limit, it is killed and restarted
          resources:
            requests:
              memory: "64Mi"   # Minimum 64MB RAM guaranteed
              cpu: "50m"       # Minimum 50 millicores (0.05 CPU cores) guaranteed
            limits:
              memory: "256Mi"  # Maximum 256MB RAM
              cpu: "200m"      # Maximum 200 millicores (0.2 CPU cores)
          
          # ---- LIVENESS PROBE ----
          # Kubernetes runs this check periodically.
          # If it fails, Kubernetes KILLS and RESTARTS the container.
          # Use for: detecting a deadlocked container that cannot recover.
          livenessProbe:
            httpGet:
              path: /          # HTTP GET request to the root path
              port: 80
            initialDelaySeconds: 15  # Wait 15s before first check (let app start)
            periodSeconds: 20        # Check every 20 seconds
            failureThreshold: 3      # Restart after 3 consecutive failures
          
          # ---- READINESS PROBE ----
          # Kubernetes runs this check periodically.
          # If it fails, Kubernetes REMOVES the pod from the service's endpoint list.
          # Traffic STOPS going to this pod, but it is NOT restarted.
          # Use for: detecting when a pod is not ready to receive traffic.
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
            failureThreshold: 3

---
# ============================================================
# BACKEND DEPLOYMENT
# ============================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: mern-app
  labels:
    app: mern-backend
    version: v1.0
    component: backend
spec:
  replicas: 2
  
  selector:
    matchLabels:
      app: mern-backend
  
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  
  template:
    metadata:
      labels:
        app: mern-backend
        version: v1.0
    
    spec:
      imagePullSecrets:
        - name: acr-secret
      
      containers:
        - name: backend-container
          image: merncicdacr.azurecr.io/mern-backend:latest
          imagePullPolicy: Always
          
          ports:
            - containerPort: 5000  # Port the Node.js app listens on
          
          # Load non-sensitive config from ConfigMap
          envFrom:
            - configMapRef:
                name: mern-config
          
          # Load sensitive config from Secret
          env:
            - name: MONGODB_URI
              valueFrom:
                secretKeyRef:
                  name: mern-secrets   # Secret name
                  key: MONGODB_URI     # Key inside the secret
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: mern-secrets
                  key: JWT_SECRET
          
          resources:
            requests:
              memory: "128Mi"   # Node.js needs more memory than Nginx
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          
          # Liveness probe hits our /health endpoint
          livenessProbe:
            httpGet:
              path: /health    # Your backend must have a GET /health endpoint
              port: 5000       # returning 200 OK
            initialDelaySeconds: 20
            periodSeconds: 15
            failureThreshold: 3
          
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 10
            failureThreshold: 3
```

---

## k8s/service.yaml

```yaml
# ============================================================
# SERVICES
# A Service provides a stable network endpoint to access pods.
# Pods have dynamic IPs — Services give a stable DNS name.
# ============================================================

# ============================================================
# FRONTEND SERVICE
# ============================================================
apiVersion: v1
kind: Service
metadata:
  name: frontend-service     # This name is used as a DNS hostname inside the cluster
  namespace: mern-app
  labels:
    app: mern-frontend
spec:
  # ClusterIP: Only accessible within the cluster (internal traffic)
  # NodePort: Also accessible via node IP:port
  # LoadBalancer: Creates an Azure Load Balancer with public IP
  # We use ClusterIP here — Ingress will handle external access
  type: ClusterIP
  
  # Match pods with this label
  selector:
    app: mern-frontend
  
  ports:
    - name: http
      protocol: TCP
      port: 80         # Port the Service listens on
      targetPort: 80   # Port on the pod to forward to (Nginx port)

---
# ============================================================
# BACKEND SERVICE
# ============================================================
apiVersion: v1
kind: Service
metadata:
  name: backend-service   # Used as hostname by frontend: http://backend-service:5000
  namespace: mern-app
  labels:
    app: mern-backend
spec:
  type: ClusterIP
  
  selector:
    app: mern-backend
  
  ports:
    - name: http
      protocol: TCP
      port: 5000
      targetPort: 5000
```

---

## k8s/ingress.yaml

```yaml
# ============================================================
# INGRESS
# Routes external HTTP/HTTPS traffic to internal Services.
# The Ingress Controller (Nginx) runs as pods in the cluster.
# It creates a single Azure Load Balancer with one public IP.
# Multiple applications can share this one IP using path/host routing.
# ============================================================

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
  namespace: mern-app
  labels:
    app: mern-app
  
  # Annotations configure the Nginx Ingress Controller behavior
  annotations:
    # Which Ingress Controller class to use
    kubernetes.io/ingress.class: "nginx"
    
    # Redirect HTTP to HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    # Set to "true" when you add TLS/SSL certificates
    
    # Maximum upload size (important for file uploads)
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    
    # Timeout settings
    nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "120"
    
    # Enable CORS headers at Ingress level
    nginx.ingress.kubernetes.io/enable-cors: "true"

spec:
  rules:
    # Rule 1: Route all traffic for our domain to the frontend
    - host: mern-app.yourdomain.com  
      # Replace with your actual domain OR
      # Use the Ingress external IP directly (see below)
      
      http:
        paths:
          # Route /api/* to backend
          - path: /api
            pathType: Prefix    # Prefix means: match any path starting with /api
            backend:
              service:
                name: backend-service
                port:
                  number: 5000
          
          # Route everything else to frontend
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

# ============================================================
# NOTE: For demo without a custom domain, use the external IP:
# 1. Apply this Ingress
# 2. Run: kubectl get ingress -n mern-app
# 3. Find the EXTERNAL-IP column
# 4. Access your app at http://<EXTERNAL-IP>
# ============================================================
```

---

# 20. DEPLOY TO AKS

## Install Nginx Ingress Controller

Before deploying your app, install the Nginx Ingress Controller on AKS. This controller watches for Ingress resources and configures routing.

```bash
# Install Nginx Ingress Controller using Helm
# If Helm is not installed, install it first:
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add the ingress-nginx Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install Nginx Ingress Controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=1 \
  --set controller.nodeSelector."kubernetes\.io/os"=linux \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
```

### Wait for External IP

```bash
# Watch until EXTERNAL-IP column shows an IP (not <pending>)
kubectl get service ingress-nginx-controller \
  --namespace ingress-nginx \
  --watch
```

This can take 2-5 minutes. Once you see:
```
NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)
ingress-nginx-controller   LoadBalancer   10.0.0.100   20.10.20.30   80:31234/TCP,443:31235/TCP
```

**Save the EXTERNAL-IP (e.g., `20.10.20.30`) — this is your app's public IP.**

## Apply All Kubernetes Manifests

```bash
# Create namespace first
kubectl apply -f k8s/namespace.yaml

# Create ConfigMap and Secrets
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml

# Create ACR pull secret (replace password with yours)
kubectl create secret docker-registry acr-secret \
  --docker-server=merncicdacr.azurecr.io \
  --docker-username=merncicdacr \
  --docker-password=<YOUR-ACR-PASSWORD> \
  --namespace=mern-app

# Deploy applications
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

## Verify Deployment

```bash
# Check all resources in the namespace
kubectl get all -n mern-app

# Expected output:
# NAME                                        READY   STATUS    RESTARTS   AGE
# pod/frontend-deployment-5f7d9-xxxxx         1/1     Running   0          2m
# pod/frontend-deployment-5f7d9-yyyyy         1/1     Running   0          2m
# pod/backend-deployment-6b8c5-xxxxx          1/1     Running   0          2m
# pod/backend-deployment-6b8c5-yyyyy          1/1     Running   0          2m
#
# NAME                       TYPE        CLUSTER-IP      PORT(S)
# service/backend-service    ClusterIP   10.0.100.1      5000/TCP
# service/frontend-service   ClusterIP   10.0.100.2      80/TCP
#
# NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/backend-deployment     2/2     2            2           2m
# deployment.apps/frontend-deployment    2/2     2            2           2m

# Check pod logs (replace pod name)
kubectl logs frontend-deployment-5f7d9-xxxxx -n mern-app

# Check Ingress
kubectl get ingress -n mern-app
```

## Access Your Application

```bash
# Get the Ingress external IP
EXTERNAL_IP=$(kubectl get ingress mern-ingress -n mern-app -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Your app is running at: http://$EXTERNAL_IP"
```

Open `http://<EXTERNAL-IP>` in your browser — you should see your React app.

---

**Running Cost After AKS Deployment:**

| Resource | Monthly Cost |
|---|---|
| ACR Basic | $5.00 |
| AKS (1x Standard_B2s) | $34.85 |
| Load Balancer | $3.65 |
| Public IP | $3.00 |
| OS Disk | $1.20 |
| Jenkins VM (B2s) | $34.85 |
| **Total** | **~$82.55** |

> IMPORTANT: Stop Jenkins VM when not running pipelines. This saves ~$34.85/month.

---
*End of Part 1 — Continues in Part 2*
