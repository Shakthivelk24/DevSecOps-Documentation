# Azure DevSecOps Migration Guide
## MERN Stack "DevOps Pipeline Dashboard" — Phase 3: Azure Deployment

> **Author:** Senior Azure Cloud Engineer / DevSecOps Architect  
> **Version:** 1.0.0  
> **Subscription:** Azure for Students ($100 Credit)  
> **Assumes:** Phase 1 (App built) ✅ | Phase 2 (Local pipeline tested) ✅

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Azure Cost Optimization Plan](#2-azure-cost-optimization-plan)
3. [Azure Resource Group](#3-azure-resource-group)
4. [Azure Virtual Machine (Jenkins Host)](#4-azure-virtual-machine-jenkins-host)
5. [Azure Container Registry (ACR)](#5-azure-container-registry-acr)
6. [Azure Kubernetes Service (AKS)](#6-azure-kubernetes-service-aks)
7. [Jenkins on Azure VM](#7-jenkins-on-azure-vm)
8. [SonarQube on Azure VM (Docker)](#8-sonarqube-on-azure-vm-docker)
9. [Trivy on Azure VM](#9-trivy-on-azure-vm)
10. [OWASP ZAP on Azure VM (Docker)](#10-owasp-zap-on-azure-vm-docker)
11. [Migration Process (10 Steps)](#11-migration-process-10-steps)
12. [Kubernetes Configuration Files](#12-kubernetes-configuration-files)
13. [Prometheus on AKS](#13-prometheus-on-aks)
14. [Grafana on AKS](#14-grafana-on-aks)
15. [Complete Jenkinsfile](#15-complete-jenkinsfile)
16. [Verification Procedures](#16-verification-procedures)
17. [Troubleshooting Guide](#17-troubleshooting-guide)
18. [Best Practices](#18-best-practices)
19. [README.md Template](#19-readmemd-template)

---

## 1. Architecture Overview

### 1.1 Azure Cloud Architecture

```
╔══════════════════════════════════════════════════════════════════════════╗
║                     AZURE FOR STUDENTS SUBSCRIPTION                      ║
║                                                                          ║
║  ┌─────────────────────────────────────────────────────────────────┐    ║
║  │              Resource Group: rg-devops-pipeline-prod             │    ║
║  │                        Region: East US                           │    ║
║  │                                                                  │    ║
║  │  ┌──────────────────┐      ┌──────────────────────────────────┐ │    ║
║  │  │   Azure VM       │      │   Azure Container Registry (ACR)│ │    ║
║  │  │  Standard_B2ms   │      │   devopspipelineacr              │ │    ║
║  │  │  Ubuntu 22.04    │      │   SKU: Basic                     │ │    ║
║  │  │                  │      └──────────────────────────────────┘ │    ║
║  │  │  ┌────────────┐  │                                            │    ║
║  │  │  │  Jenkins   │  │      ┌──────────────────────────────────┐ │    ║
║  │  │  │  :8080     │  │      │   Azure Kubernetes Service (AKS) │ │    ║
║  │  │  └────────────┘  │      │   aks-devops-cluster             │ │    ║
║  │  │  ┌────────────┐  │      │                                  │ │    ║
║  │  │  │ SonarQube  │  │      │  ┌──────────┐  ┌─────────────┐  │ │    ║
║  │  │  │  :9000     │  │      │  │ Frontend │  │   Backend   │  │ │    ║
║  │  │  └────────────┘  │      │  │  Pod(s)  │  │   Pod(s)    │  │ │    ║
║  │  │  ┌────────────┐  │      │  └──────────┘  └─────────────┘  │ │    ║
║  │  │  │   Docker   │  │      │  ┌──────────┐  ┌─────────────┐  │ │    ║
║  │  │  │  Engine    │  │      │  │  MongoDB │  │   Ingress   │  │ │    ║
║  │  │  └────────────┘  │      │  │  Pod(s)  │  │ Controller  │  │ │    ║
║  │  │  ┌────────────┐  │      │  └──────────┘  └─────────────┘  │ │    ║
║  │  │  │   Trivy    │  │      │  ┌──────────┐  ┌─────────────┐  │ │    ║
║  │  │  │  Scanner   │  │      │  │Prometheus│  │   Grafana   │  │ │    ║
║  │  │  └────────────┘  │      │  │  :9090   │  │   :3000     │  │ │    ║
║  │  │  ┌────────────┐  │      │  └──────────┘  └─────────────┘  │ │    ║
║  │  │  │ OWASP ZAP  │  │      └──────────────────────────────────┘ │    ║
║  │  │  │  Docker    │  │                                            │    ║
║  │  │  └────────────┘  │      ┌──────────────────────────────────┐ │    ║
║  │  └──────────────────┘      │   Azure Virtual Network (VNet)   │ │    ║
║  │                            │   10.0.0.0/16                    │ │    ║
║  │                            │   ┌────────┐  ┌──────────────┐  │ │    ║
║  │                            │   │Subnet-1│  │  Subnet-AKS  │  │ │    ║
║  │                            │   │10.0.1.0│  │  10.0.2.0    │  │ │    ║
║  │                            │   └────────┘  └──────────────┘  │ │    ║
║  │                            └──────────────────────────────────┘ │    ║
║  └─────────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### 1.2 CI/CD Pipeline Flow

```
┌────────────┐
│ Developer  │
│  Git Push  │
└─────┬──────┘
      │  GitHub Webhook
      ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        JENKINS PIPELINE                             │
│                                                                     │
│  [1] Checkout Code         [2] npm install        [3] npm test      │
│       GitHub                  Dependencies          Unit Tests      │
│                                                                     │
│  [4] SonarQube Scan       [5] Quality Gate        [6] npm build     │
│       SAST Analysis          Pass/Fail             Production Build  │
│                                                                     │
│  [7] Docker Build         [8] Trivy Scan          [9] Push to ACR   │
│       Image Build             CVE Scan             Azure Registry    │
│                                                                     │
│  [10] Deploy to AKS       [11] Verify Deploy      [12] OWASP ZAP   │
│        kubectl apply          Pod Health            DAST Scan        │
│                                                                     │
│  [13] Archive Reports     [14] Notify              [15] Monitor      │
│        HTML/JSON              Slack/Email           Prometheus+Graf  │
└─────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────┐    ┌──────────────┐    ┌────────────────┐
│    ACR      │───▶│     AKS      │───▶│  Prometheus    │
│  Images     │    │  Kubernetes  │    │  + Grafana     │
└─────────────┘    └──────────────┘    └────────────────┘
```

### 1.3 Security Flow

```
CODE COMMIT
    │
    ▼
┌─────────────────────────────────────────────────────┐
│                  SECURITY LAYERS                     │
│                                                      │
│  Layer 1: SAST (Static Analysis)                    │
│  ├── SonarQube Code Quality Scan                    │
│  ├── Vulnerability Detection in Source Code         │
│  └── Quality Gate Enforcement                       │
│                                                      │
│  Layer 2: SCA (Software Composition Analysis)       │
│  ├── Trivy Docker Image Scanning                    │
│  ├── OS Package CVE Detection                       │
│  └── Application Dependency Scanning                │
│                                                      │
│  Layer 3: DAST (Dynamic Analysis)                   │
│  ├── OWASP ZAP Automated Scanning                   │
│  ├── Runtime Vulnerability Testing                  │
│  └── HTML Report Generation                         │
│                                                      │
│  Layer 4: Infrastructure Security                   │
│  ├── Azure NSG (Network Security Groups)            │
│  ├── AKS RBAC (Role-Based Access Control)           │
│  ├── ACR Private Access                             │
│  └── Kubernetes Secrets Management                  │
│                                                      │
│  Layer 5: Runtime Monitoring                        │
│  ├── Prometheus Metrics Collection                  │
│  └── Grafana Dashboard Alerting                     │
└─────────────────────────────────────────────────────┘
```

### 1.4 Monitoring Flow

```
APPLICATION PODS
      │  Metrics (/metrics endpoint)
      ▼
┌─────────────┐
│  Prometheus │  ← Scrapes every 15s
│  :9090      │
└──────┬──────┘
       │  Query (PromQL)
       ▼
┌─────────────┐
│   Grafana   │  ← Dashboards + Alerts
│   :3000     │
└──────┬──────┘
       │  Alerts
       ▼
┌─────────────┐
│  Alertmanag │  (optional: email/Slack)
│  er         │
└─────────────┘
```

---

## 2. Azure Cost Optimization Plan

### 2.1 Resource Cost Summary

| Resource | SKU/Tier | Est. Monthly Cost | Credit Impact | Stoppable | Delete After Test |
|---|---|---|---|---|---|
| Azure VM (Jenkins) | Standard_B2ms | ~$30–35/mo | $30–35 | ✅ Yes | ❌ Keep for CI/CD |
| Azure Container Registry | Basic | ~$5/mo | $5 | ❌ No | ⚠️ If done |
| Azure Kubernetes Service | 1×Standard_B2s node | ~$30–35/mo | $30–35 | ✅ Stop node | ⚠️ If done |
| Public IP (VM) | Static | ~$3–4/mo | $3–4 | ✅ Release | ⚠️ If done |
| Managed Disk (VM) | 30 GB P4 | ~$2–3/mo | $2–3 | ❌ No | ⚠️ With VM |
| VNet + NSG | Free tier | $0 | $0 | — | — |
| **Total (running)** | | **~$70–80/mo** | **~$70–80** | | |

> **💡 With $100 student credit:** You have roughly **1–1.5 months** of full operation. Stop the VM and scale AKS to 0 nodes when not in use to extend credit significantly.

### 2.2 Cost-Saving Recommendations

```
COST SAVING STRATEGIES
══════════════════════

1. Stop VM when not using Jenkins
   Azure Portal → VM → Stop
   Savings: ~$1/day

2. Scale AKS node pool to 0 when idle
   az aks nodepool scale --node-count 0
   Savings: ~$1/day

3. Use ACR Basic (not Standard/Premium)
   Basic is sufficient for student projects
   Savings: ~$15/mo vs Premium

4. Use B-series VMs (Burstable)
   Standard_B2ms is 70–80% cheaper than D-series
   for intermittent workloads

5. Set Azure Budget Alerts
   Azure Portal → Cost Management → Budgets
   Set alert at $50, $80, $90

6. Delete test resources immediately
   After evaluation, delete the entire resource group
   az group delete --name rg-devops-pipeline-prod
```

---

## 3. Azure Resource Group

### 3.1 What Is a Resource Group?

A Resource Group is a logical container that holds all related Azure resources for a solution. All resources in this project live inside one resource group for easy management and deletion.

### 3.2 Create Resource Group via Azure Portal

```
Step-by-Step Navigation:

Azure Portal (portal.azure.com)
    │
    ▼
Search bar → type "Resource Groups" → Click
    │
    ▼
Click [+ Create]
    │
    ▼
Fill in the form:
    ┌─────────────────────────────────────────────────┐
    │ Subscription:     Azure for Students            │
    │ Resource group:   rg-devops-pipeline-prod       │
    │ Region:           (US) East US                  │
    └─────────────────────────────────────────────────┘
    │
    ▼
Click [Review + Create]
    │
    ▼
Click [Create]
    │
    ▼
✅ Resource group created
```

**Field Explanations:**

| Field | Value | Why |
|---|---|---|
| Subscription | Azure for Students | Your student account |
| Resource group | `rg-devops-pipeline-prod` | `rg-` prefix is industry convention |
| Region | East US | Low latency, lowest cost for most US/global projects |

### 3.3 Naming Convention

```
Format: rg-{project}-{environment}

Examples:
  rg-devops-pipeline-prod    ← Production
  rg-devops-pipeline-dev     ← Development
  rg-devops-pipeline-test    ← Testing
```

---

## 4. Azure Virtual Machine (Jenkins Host)

### 4.1 Why This VM?

The `Standard_B2ms` SKU is selected because:

- **2 vCPUs / 8 GB RAM** — sufficient for Jenkins + Docker + SonarQube + Trivy + OWASP ZAP simultaneously
- **Burstable CPU** — handles CI pipeline spikes without paying for always-on high CPU
- **Cost** — ~$30–35/month vs ~$90+ for equivalent D-series
- **Ubuntu 22.04 LTS** — stable, long-term support, best Docker/Kubernetes compatibility

### 4.2 Create VM via Azure Portal

```
Azure Portal
    │
    ▼
Search bar → type "Virtual Machines" → Click
    │
    ▼
Click [+ Create] → [Azure Virtual Machine]
    │
    ▼
BASICS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Subscription:          Azure for Students            │
    │ Resource group:        rg-devops-pipeline-prod       │
    │ Virtual machine name:  vm-jenkins-devops             │
    │ Region:                East US                       │
    │ Availability options:  No infrastructure redundancy  │
    │ Security type:         Standard                      │
    │ Image:                 Ubuntu Server 22.04 LTS       │
    │                        (x64 Gen2) — click Browse     │
    │ VM Architecture:       x64                           │
    │ Size:                  Standard_B2ms                 │
    │                        (click "See all sizes",       │
    │                        filter "B2ms", select)        │
    └──────────────────────────────────────────────────────┘
    │
    ▼
ADMINISTRATOR ACCOUNT:
    ┌──────────────────────────────────────────────────────┐
    │ Authentication type:   SSH public key                │
    │ Username:              azureuser                     │
    │ SSH public key source: Generate new key pair         │
    │ Key pair name:         vm-jenkins-devops-key         │
    └──────────────────────────────────────────────────────┘
    │
    ▼
INBOUND PORT RULES:
    ┌──────────────────────────────────────────────────────┐
    │ Public inbound ports:  Allow selected ports          │
    │ Select inbound ports:  SSH (22)                      │
    │                        HTTP (80)                     │
    │                        HTTPS (443)                   │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Next: Disks]
```

### 4.3 Disk Configuration

```
DISKS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ OS disk type:          Standard SSD (LRS)            │
    │ OS disk size:          30 GiB (default)              │
    │ Delete with VM:        ✅ Checked                    │
    │ Encryption type:       (Default) Platform-managed    │
    └──────────────────────────────────────────────────────┘

    ⚠️ Do NOT use Premium SSD — unnecessary cost for this workload
    ⚠️ Check "Delete with VM" to avoid orphan disk costs
```

### 4.4 Networking Configuration

```
NETWORKING TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Virtual network:       (new) vnet-devops-pipeline    │
    │ Subnet:                (new) subnet-jenkins          │
    │                        (10.0.1.0/24)                 │
    │ Public IP:             (new) pip-vm-jenkins-devops   │
    │ NIC NSG:               Advanced                      │
    │ Configure NSG:         (new) nsg-vm-jenkins-devops   │
    │ Delete NIC when VM     ✅ Checked                    │
    │ is deleted:                                          │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Review + Create]
    │
    ▼
Click [Create]
    │
    ▼
⬇ DOWNLOAD PRIVATE KEY PROMPT APPEARS
    │
    ▼
Click [Download private key and create resource]
    │
    ▼
✅ Download vm-jenkins-devops-key.pem to your local machine
   Store it safely — you cannot download it again!
```

### 4.5 Add NSG Rules (After VM Creation)

After the VM is created, add custom port rules for Jenkins, SonarQube:

```
Azure Portal
    │
    ▼
Resource Groups → rg-devops-pipeline-prod
    │
    ▼
Click: nsg-vm-jenkins-devops
    │
    ▼
Settings → Inbound security rules → [+ Add]
```

Add the following rules one by one:

| Priority | Name | Port | Protocol | Source | Action | Purpose |
|---|---|---|---|---|---|---|
| 100 | Allow-SSH | 22 | TCP | My IP | Allow | SSH access |
| 110 | Allow-HTTP | 80 | TCP | Any | Allow | HTTP traffic |
| 120 | Allow-HTTPS | 443 | TCP | Any | Allow | HTTPS traffic |
| 130 | Allow-Jenkins | 8080 | TCP | Any | Allow | Jenkins UI |
| 140 | Allow-SonarQube | 9000 | TCP | Any | Allow | SonarQube UI |
| 150 | Allow-Grafana | 3000 | TCP | Any | Allow | Grafana UI |
| 160 | Allow-NodeApp | 3001 | TCP | Any | Allow | MERN App test |

> **Security Tip:** For production, replace "Any" source with your specific IP address for Jenkins (8080) and SonarQube (9000).

### 4.6 Connect to VM via SSH

```bash
# Step 1: Set correct permissions on your key
chmod 400 ~/Downloads/vm-jenkins-devops-key.pem

# Step 2: Get your VM's Public IP
# Azure Portal → Virtual Machines → vm-jenkins-devops → Overview
# Copy the Public IP address

# Step 3: SSH into VM
ssh -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<YOUR-VM-PUBLIC-IP>

# Expected output:
# Welcome to Ubuntu 22.04.x LTS (GNU/Linux 5.15.x-azure x86_64)
# azureuser@vm-jenkins-devops:~$
```

### 4.7 VM Initial Setup

```bash
# Update package lists
sudo apt-get update -y

# Upgrade installed packages
sudo apt-get upgrade -y

# Install essential tools
sudo apt-get install -y \
    curl \
    wget \
    git \
    unzip \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release \
    htop \
    nano

# Verify installations
git --version
curl --version
```

---

## 5. Azure Container Registry (ACR)

### 5.1 What Is ACR?

Azure Container Registry is a managed Docker registry for storing and managing your Docker container images. It integrates directly with AKS, eliminating the need for Docker Hub credentials.

### 5.2 Create ACR via Azure Portal

```
Azure Portal
    │
    ▼
Search bar → type "Container Registries" → Click
    │
    ▼
Click [+ Create]
    │
    ▼
BASICS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Subscription:      Azure for Students                │
    │ Resource group:    rg-devops-pipeline-prod           │
    │ Registry name:     devopspipelineacr                 │
    │                    (must be globally unique,         │
    │                     3-50 chars, alphanumeric only)   │
    │ Location:          East US                           │
    │ SKU:               Basic                             │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Review + Create]
    │
    ▼
Click [Create]
    │
    ▼
✅ ACR created
   Login server: devopspipelineacr.azurecr.io
```

**SKU Comparison:**

| SKU | Storage | Webhooks | Cost | Recommendation |
|---|---|---|---|---|
| Basic | 10 GB | 2 | ~$5/mo | ✅ Use for student |
| Standard | 100 GB | 10 | ~$20/mo | ❌ Overkill |
| Premium | 500 GB | 500 | ~$50/mo | ❌ Too expensive |

### 5.3 Enable Admin Access

```
Azure Portal
    │
    ▼
Container Registries → devopspipelineacr
    │
    ▼
Settings → Access keys
    │
    ▼
Toggle: Admin user → Enabled
    │
    ▼
Note the credentials:
    ┌──────────────────────────────────────────────────┐
    │ Registry name:  devopspipelineacr                │
    │ Login server:   devopspipelineacr.azurecr.io     │
    │ Username:       devopspipelineacr                │
    │ Password:       <password-1>  (save this!)       │
    │ Password2:      <password-2>  (backup)           │
    └──────────────────────────────────────────────────┘
```

### 5.4 Docker Login to ACR (From VM)

```bash
# SSH into VM first, then:

# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login --use-device-code
# Follow the browser instructions

# Login to ACR via Azure CLI (recommended)
az acr login --name devopspipelineacr

# OR login via Docker directly
docker login devopspipelineacr.azurecr.io \
  --username devopspipelineacr \
  --password <YOUR-ACR-PASSWORD>

# Expected output:
# Login Succeeded
```

### 5.5 Docker Tag and Push Commands

```bash
# Tag your existing local image for ACR
# Format: <registry-name>.azurecr.io/<image-name>:<tag>

# Tag frontend image
docker tag devops-dashboard-frontend:latest \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Tag backend image
docker tag devops-dashboard-backend:latest \
  devopspipelineacr.azurecr.io/devops-dashboard-backend:latest

# Push frontend image to ACR
docker push devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Push backend image to ACR
docker push devopspipelineacr.azurecr.io/devops-dashboard-backend:latest

# Verify images in ACR
az acr repository list --name devopspipelineacr --output table

# Expected output:
# Result
# ──────────────────────────────────
# devops-dashboard-frontend
# devops-dashboard-backend
```

---

## 6. Azure Kubernetes Service (AKS)

### 6.1 Why AKS?

AKS is a managed Kubernetes service that handles the Kubernetes control plane automatically. You only pay for the worker nodes (VMs), not the master nodes.

### 6.2 Create AKS via Azure Portal

```
Azure Portal
    │
    ▼
Search bar → type "Kubernetes Services" → Click
    │
    ▼
Click [+ Create] → [Create a Kubernetes cluster]
    │
    ▼
BASICS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Subscription:       Azure for Students               │
    │ Resource group:     rg-devops-pipeline-prod          │
    │ Cluster preset:     Dev/Test                         │
    │ Kubernetes cluster  aks-devops-cluster               │
    │ name:                                                │
    │ Region:             East US                          │
    │ Kubernetes version: 1.29.x (latest stable)           │
    │ Automatic upgrades: Enabled (patch)                  │
    │ Authentication:     Local accounts with Kubernetes   │
    │ method:             RBAC                             │
    └──────────────────────────────────────────────────────┘
    │
    ▼
NODE POOLS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ agentpool (system pool):                             │
    │   Node size:     Standard_B2s                        │
    │                  (click change size → filter B2s)    │
    │   Scale method:  Manual                              │
    │   Node count:    1                                   │
    │                                                      │
    │ ⚠️ Do NOT enable autoscale — costs more credits      │
    └──────────────────────────────────────────────────────┘
    │
    ▼
NETWORKING TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Network configuration: kubenet (simpler, cheaper)    │
    │ DNS name prefix:       aks-devops-cluster-dns        │
    │ Network policy:        None (for student project)    │
    └──────────────────────────────────────────────────────┘
    │
    ▼
INTEGRATIONS TAB:
    ┌──────────────────────────────────────────────────────┐
    │ Container registry:  devopspipelineacr               │
    │                      (select from dropdown)          │
    │ Azure Monitor:       Disabled (saves cost)           │
    │ Azure Policy:        Disabled                        │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Review + Create]
    │
    ▼
Click [Create]

⏳ Wait 5–10 minutes for cluster to deploy
    │
    ▼
✅ AKS cluster created
```

> **Cost Note:** `Standard_B2s` (2 vCPU, 4 GB) = ~$30–35/month for 1 node. This is the minimum for running MERN + Prometheus + Grafana.

### 6.3 Connect kubectl to AKS

```bash
# On the Azure VM (SSH into it first):

# Install kubectl
sudo az aks install-cli

# OR install manually
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Download AKS credentials to kubeconfig
az aks get-credentials \
  --resource-group rg-devops-pipeline-prod \
  --name aks-devops-cluster \
  --overwrite-existing

# Verify connection
kubectl get nodes

# Expected output:
# NAME                                STATUS   ROLES   AGE   VERSION
# aks-agentpool-12345678-vmss000000   Ready    agent   5m    v1.29.x

# Get cluster info
kubectl cluster-info
```

### 6.4 Attach ACR to AKS

```bash
# Allow AKS to pull images from ACR (one-time setup)
az aks update \
  --name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --attach-acr devopspipelineacr

# Verify the attachment
az aks show \
  --name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --query "identityProfile"
```

---

## 7. Jenkins on Azure VM

### 7.1 Install Java (Required for Jenkins)

```bash
# SSH into VM
ssh -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<VM-PUBLIC-IP>

# Install Java 17
sudo apt-get install -y openjdk-17-jdk

# Verify
java -version
# Expected: openjdk version "17.0.x" 2023-xx-xx
```

### 7.2 Install Jenkins

```bash
# Add Jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins apt repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt-get update -y
sudo apt-get install -y jenkins

# Start Jenkins
sudo systemctl start jenkins

# Enable Jenkins to start on boot
sudo systemctl enable jenkins

# Check Jenkins status
sudo systemctl status jenkins

# Expected output:
# ● jenkins.service - Jenkins Automation Server
#    Loaded: loaded (/lib/systemd/system/jenkins.service; enabled)
#    Active: active (running) since ...
```

### 7.3 Initial Jenkins Setup

```bash
# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Copy the 32-character alphanumeric password
# Example output: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
```

```
Browser: http://<VM-PUBLIC-IP>:8080
    │
    ▼
Unlock Jenkins
    ┌─────────────────────────────────────────┐
    │ Paste the initialAdminPassword here     │
    └─────────────────────────────────────────┘
    │
    ▼
Install Suggested Plugins (click this)
    │
    ▼  (wait 2-3 minutes)
Create First Admin User
    ┌─────────────────────────────────────────┐
    │ Username: admin                         │
    │ Password: <your-strong-password>        │
    │ Full name: DevOps Admin                 │
    │ Email: admin@devops.local               │
    └─────────────────────────────────────────┘
    │
    ▼
Jenkins URL: http://<VM-PUBLIC-IP>:8080/
    │
    ▼
Click [Start using Jenkins]
✅ Jenkins is ready
```

### 7.4 Install Jenkins Plugins

```
Jenkins Dashboard
    │
    ▼
Manage Jenkins → Plugins → Available Plugins
    │
    ▼
Search and install each plugin (check box → Install):
```

| Plugin Name | Purpose |
|---|---|
| Docker Pipeline | Docker build/push in pipeline |
| Docker Commons | Shared Docker utilities |
| Kubernetes | Deploy to K8s from Jenkins |
| Kubernetes CLI | kubectl commands |
| SonarQube Scanner | SonarQube integration |
| OWASP Dependency-Check | Dependency vulnerability scan |
| Blue Ocean | Modern pipeline UI |
| GitHub Integration | Webhook trigger |
| Credentials Binding | Secure credential injection |
| Pipeline Stage View | Pipeline visualization |
| Workspace Cleanup | Clean workspace between builds |
| AnsiColor | Colored console output |
| Timestamper | Timestamps in console |

```
After selecting all plugins:
    │
    ▼
Click [Install]
    │
    ▼  (wait 3-5 minutes)
Check: Restart Jenkins when installation is complete
    │
    ▼
✅ Plugins installed
```

### 7.5 Install Docker on VM

```bash
# Remove old Docker versions if any
sudo apt-get remove -y docker docker-engine docker.io containerd runc 2>/dev/null

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add jenkins user to docker group (critical!)
sudo usermod -aG docker jenkins
sudo usermod -aG docker azureuser

# Add current user to docker group (apply immediately)
newgrp docker

# Verify
docker --version
# Expected: Docker version 26.x.x, build xxxxx

# Test
docker run hello-world
```

### 7.6 Configure Jenkins Credentials

```
Jenkins Dashboard
    │
    ▼
Manage Jenkins → Credentials → System → Global credentials
    │
    ▼
Click [+ Add Credentials]
```

Add the following credentials:

**Credential 1: ACR Credentials**
```
Kind:          Username with password
Scope:         Global
Username:      devopspipelineacr
Password:      <ACR-PASSWORD-FROM-PORTAL>
ID:            acr-credentials
Description:   Azure Container Registry Credentials
Click [Create]
```

**Credential 2: GitHub Token (if private repo)**
```
Kind:          Secret text
Scope:         Global
Secret:        <GITHUB-PERSONAL-ACCESS-TOKEN>
ID:            github-token
Description:   GitHub Personal Access Token
Click [Create]
```

**Credential 3: SonarQube Token**
```
Kind:          Secret text
Scope:         Global
Secret:        <SONARQUBE-TOKEN> (generate in SonarQube UI after install)
ID:            sonarqube-token
Description:   SonarQube Authentication Token
Click [Create]
```

**Credential 4: Azure Service Principal (for kubectl)**
```
Kind:          Secret file
Scope:         Global
File:          Upload your kubeconfig file (~/.kube/config)
ID:            aks-kubeconfig
Description:   AKS Kubeconfig File
Click [Create]
```

### 7.7 Configure SonarQube in Jenkins

```
Jenkins Dashboard
    │
    ▼
Manage Jenkins → System
    │
    ▼
Scroll to: SonarQube servers section
    │
    ▼
Click [Add SonarQube]
    ┌──────────────────────────────────────────────────────┐
    │ Name:            SonarQube-Server                    │
    │ Server URL:      http://localhost:9000                │
    │ Server auth      sonarqube-token (select from list)  │
    │ token:                                               │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Save]
```

### 7.8 Configure Jenkins Tools

```
Jenkins Dashboard
    │
    ▼
Manage Jenkins → Tools
    │
    ▼
NodeJS installations:
    ┌──────────────────────────────────────────────────────┐
    │ Click [Add NodeJS]                                   │
    │ Name:     NodeJS-18                                  │
    │ Version:  NodeJS 18.x (LTS)                          │
    │ ✅ Install automatically                             │
    └──────────────────────────────────────────────────────┘

SonarQube Scanner installations:
    ┌──────────────────────────────────────────────────────┐
    │ Click [Add SonarQube Scanner]                        │
    │ Name:     SonarScanner                               │
    │ ✅ Install automatically                             │
    │ Version:  Latest                                     │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Save]
```

### 7.9 Configure GitHub Webhook

```
GitHub Repository
    │
    ▼
Settings → Webhooks → Add webhook
    ┌──────────────────────────────────────────────────────┐
    │ Payload URL:  http://<VM-PUBLIC-IP>:8080/            │
    │               github-webhook/                        │
    │ Content type: application/json                       │
    │ Which events: Just the push event                    │
    │ Active:       ✅ Checked                             │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Add webhook]
```

---

## 8. SonarQube on Azure VM (Docker)

### 8.1 Why Docker for SonarQube?

Deploying SonarQube in Docker on the same VM avoids needing a separate Azure resource, saving $15–20/month.

### 8.2 Install and Run SonarQube

```bash
# SSH into VM
ssh -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<VM-PUBLIC-IP>

# Set required kernel parameter for Elasticsearch (SonarQube dependency)
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

# Create a Docker network for DevOps tools
docker network create devops-network

# Create directories for SonarQube data persistence
mkdir -p ~/sonarqube/data ~/sonarqube/logs ~/sonarqube/extensions

# Run SonarQube container
docker run -d \
  --name sonarqube \
  --network devops-network \
  --restart unless-stopped \
  -p 9000:9000 \
  -v ~/sonarqube/data:/opt/sonarqube/data \
  -v ~/sonarqube/logs:/opt/sonarqube/logs \
  -v ~/sonarqube/extensions:/opt/sonarqube/extensions \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:lts-community

# Verify container is running
docker ps

# Check logs (wait ~60 seconds for startup)
docker logs sonarqube --follow

# Look for: SonarQube is operational
# Then press Ctrl+C to stop following logs
```

### 8.3 SonarQube Initial Configuration

```
Browser: http://<VM-PUBLIC-IP>:9000
    │
    ▼
Default credentials:
    Username: admin
    Password: admin
    │
    ▼
Change password when prompted (required)
New password: <your-strong-password>
    │
    ▼
Create Project:
    Administration → Projects → Create Project
    ┌──────────────────────────────────────────────────────┐
    │ Project key:     devops-pipeline-dashboard           │
    │ Display name:    DevOps Pipeline Dashboard           │
    │ Visibility:      Public                              │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Generate Token for Jenkins:
    My Account (top right) → Security → Tokens
    ┌──────────────────────────────────────────────────────┐
    │ Token name:  jenkins-pipeline-token                  │
    │ Type:        Global Analysis Token                   │
    │ Expires:     No expiration                           │
    └──────────────────────────────────────────────────────┘
    │
    ▼
Click [Generate]
Copy the token: sqp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
    │
    ▼
⚠️ Save this token — it won't be shown again!
   Add it to Jenkins credentials as 'sonarqube-token'
```

### 8.4 sonar-project.properties

Create this file in your project root:

```properties
# sonar-project.properties
sonar.projectKey=devops-pipeline-dashboard
sonar.projectName=DevOps Pipeline Dashboard
sonar.projectVersion=1.0

# Source directories
sonar.sources=src,server
sonar.exclusions=**/node_modules/**,**/coverage/**,**/*.test.js,**/dist/**

# Language
sonar.language=js

# Encoding
sonar.sourceEncoding=UTF-8

# Coverage reports (if using Jest)
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# Test reports
sonar.testExecutionReportPaths=coverage/test-reporter.xml
```

---

## 9. Trivy on Azure VM

### 9.1 Install Trivy

```bash
# Add Trivy repository
sudo apt-get install -y wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
  sudo apt-key add -

echo "deb https://aquasecurity.github.io/trivy-repo/deb \
  $(lsb_release -sc) main" | \
  sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update -y
sudo apt-get install -y trivy

# Verify
trivy --version
# Expected: Version: 0.5x.x
```

### 9.2 Trivy Scan Commands

```bash
# Scan a Docker image (Full scan)
trivy image devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Scan with severity filter (CRITICAL and HIGH only)
trivy image \
  --severity CRITICAL,HIGH \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Scan and output to JSON (for CI/CD pipeline)
trivy image \
  --format json \
  --output trivy-report.json \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Scan and output to HTML (for archiving)
trivy image \
  --format template \
  --template "@contrib/html.tpl" \
  --output trivy-report.html \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Fail build on CRITICAL vulnerabilities
trivy image \
  --exit-code 1 \
  --severity CRITICAL \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

# Scan filesystem (dependencies)
trivy fs \
  --security-checks vuln,secret \
  .

# Download template for HTML reports
mkdir -p ~/.trivy/templates
curl -sSL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl \
  -o ~/.trivy/templates/html.tpl
```

### 9.3 Trivy Configuration File

```yaml
# trivy.yaml (place in project root)
severity:
  - CRITICAL
  - HIGH
  - MEDIUM

exit-code: 0   # Set to 1 to fail pipeline on findings

format: table

output: ""

ignore-unfixed: false

vuln-type:
  - os
  - library

security-checks:
  - vuln
  - secret

timeout: "5m0s"
```

---

## 10. OWASP ZAP on Azure VM (Docker)

### 10.1 Run OWASP ZAP via Docker

```bash
# Create directory for ZAP reports
mkdir -p ~/zap-reports

# Pull ZAP Docker image
docker pull ghcr.io/zaproxy/zaproxy:stable

# Run ZAP baseline scan (passive scan only)
docker run --rm \
  -v ~/zap-reports:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://<YOUR-APP-URL> \
  -r zap-baseline-report.html \
  -J zap-baseline-report.json \
  -x zap-baseline-report.xml \
  -I

# Run ZAP full scan (active scan — more thorough)
docker run --rm \
  -v ~/zap-reports:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
  -t http://<YOUR-APP-URL> \
  -r zap-full-report.html \
  -J zap-full-report.json \
  -I

# Run API scan (for backend REST API)
docker run --rm \
  -v ~/zap-reports:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-api-scan.py \
  -t http://<YOUR-APP-URL>/api \
  -f openapi \
  -r zap-api-report.html \
  -I

# View reports
ls -la ~/zap-reports/
```

### 10.2 ZAP Automation Framework Config

```yaml
# zap-config.yaml
env:
  contexts:
    - name: "DevOps Dashboard Context"
      urls:
        - "http://<YOUR-APP-URL>"
      includePaths:
        - "http://<YOUR-APP-URL>/.*"
      excludePaths:
        - "http://<YOUR-APP-URL>/logout.*"

jobs:
  - type: passiveScan-config
    parameters:
      maxAlertsPerRule: 10
      scanOnlyInScope: true

  - type: spider
    parameters:
      maxDuration: 2
      maxDepth: 5

  - type: passiveScan-wait
    parameters:
      maxDuration: 5

  - type: activeScan
    parameters:
      maxRuleDurationInMins: 2
      maxScanDurationInMins: 10

  - type: report
    parameters:
      template: "traditional-html"
      reportDir: "/zap/wrk"
      reportFile: "zap-automation-report.html"
```

---

## 11. Migration Process (10 Steps)

### Step 1: Clone Existing GitHub Repository

```bash
# SSH into Azure VM
ssh -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<VM-PUBLIC-IP>

# Clone your existing repository
git clone https://github.com/<YOUR-USERNAME>/devops-pipeline-dashboard.git

# Navigate to project
cd devops-pipeline-dashboard

# Verify all files exist
ls -la

# Expected files:
# Dockerfile (frontend)
# Dockerfile.backend (or similar)
# docker-compose.yml
# Jenkinsfile
# k8s/
# package.json
# src/ (React frontend)
# server/ (Express backend)
# sonar-project.properties
```

### Step 2: Verify Project

```bash
# Verify project structure
tree -L 2 .

# Check Node.js version compatibility
node --version  # Should be 18.x or 20.x

# Install dependencies locally for verification
npm install

# Run tests
npm test -- --watchAll=false

# Check frontend build
npm run build

# Verify Dockerfiles exist
cat Dockerfile
cat Dockerfile.backend  # or your backend Dockerfile name

# Verify Jenkinsfile
cat Jenkinsfile

# Verify Kubernetes manifests
ls -la k8s/
cat k8s/deployment.yaml
cat k8s/service.yaml
```

### Step 3: Build Docker Images

```bash
# Login to ACR first
az acr login --name devopspipelineacr

# Build frontend Docker image
docker build \
  -t devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest \
  -f Dockerfile \
  .

# Build backend Docker image
docker build \
  -t devopspipelineacr.azurecr.io/devops-dashboard-backend:latest \
  -f Dockerfile.backend \
  .

# Verify images were built
docker images | grep devopspipelineacr

# Expected output:
# devopspipelineacr.azurecr.io/devops-dashboard-frontend   latest   abc123   2 min ago   250MB
# devopspipelineacr.azurecr.io/devops-dashboard-backend    latest   def456   1 min ago   200MB

# Test images locally before pushing
docker run -d --name test-frontend -p 3000:3000 \
  devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

docker run -d --name test-backend -p 5000:5000 \
  devopspipelineacr.azurecr.io/devops-dashboard-backend:latest

# Verify containers are running
docker ps

# Stop test containers
docker stop test-frontend test-backend
docker rm test-frontend test-backend
```

### Step 4: Push Docker Images to ACR

```bash
# Push frontend image
docker push devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest
# Progress bars will show as layers are pushed

# Push backend image
docker push devopspipelineacr.azurecr.io/devops-dashboard-backend:latest

# Verify images in ACR
az acr repository list \
  --name devopspipelineacr \
  --output table

# View image tags
az acr repository show-tags \
  --name devopspipelineacr \
  --repository devops-dashboard-frontend \
  --output table

# View image details
az acr repository show \
  --name devopspipelineacr \
  --image devops-dashboard-frontend:latest
```

### Step 5: Update Kubernetes Manifests

Update all Kubernetes manifests to use ACR image URLs instead of local/Docker Hub images:

```bash
# Navigate to your k8s directory
cd ~/devops-pipeline-dashboard/k8s

# Replace local image references with ACR references
# (Do this for each manifest file that has image: field)

# Example: Change from:
#   image: devops-dashboard-frontend:latest
# Change to:
#   image: devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest

sed -i 's|image: devops-dashboard-frontend:latest|image: devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest|g' deployment.yaml

sed -i 's|image: devops-dashboard-backend:latest|image: devopspipelineacr.azurecr.io/devops-dashboard-backend:latest|g' deployment.yaml

# Verify the changes
grep "image:" deployment.yaml
```

### Step 6: Deploy to AKS

```bash
# Ensure kubectl is connected to AKS
kubectl get nodes

# Create namespace for the application
kubectl create namespace devops-app

# Apply all Kubernetes manifests
kubectl apply -f k8s/configmap.yaml -n devops-app
kubectl apply -f k8s/secret.yaml -n devops-app
kubectl apply -f k8s/deployment.yaml -n devops-app
kubectl apply -f k8s/service.yaml -n devops-app
kubectl apply -f k8s/ingress.yaml -n devops-app

# OR apply all at once
kubectl apply -f k8s/ -n devops-app

# Watch deployment progress
kubectl rollout status deployment/frontend-deployment -n devops-app
kubectl rollout status deployment/backend-deployment -n devops-app
```

### Step 7: Verify Deployment

```bash
# Check all pods
kubectl get pods -n devops-app -o wide

# Expected output:
# NAME                                  READY   STATUS    RESTARTS   AGE
# frontend-deployment-7d4b9c8f6-xjk2p   1/1     Running   0          2m
# backend-deployment-5f8c7b9d4-lmn3q    1/1     Running   0          2m
# mongodb-deployment-6b7d8e9f2-pqr4s    1/1     Running   0          2m

# Check deployments
kubectl get deployments -n devops-app

# Check services
kubectl get services -n devops-app

# Expected output includes EXTERNAL-IP for LoadBalancer type:
# NAME               TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
# frontend-service   LoadBalancer   10.0.200.1     20.xxx.xxx.xxx   80:31234/TCP
# backend-service    ClusterIP      10.0.200.2     <none>           5000/TCP

# Check ingress
kubectl get ingress -n devops-app

# Describe pods if they fail
kubectl describe pod <pod-name> -n devops-app

# View pod logs
kubectl logs <pod-name> -n devops-app

# Check all resources
kubectl get all -n devops-app
```

### Step 8: Run OWASP ZAP Scan

```bash
# Get the application URL (from service EXTERNAL-IP)
APP_URL=$(kubectl get service frontend-service -n devops-app \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo "App URL: http://$APP_URL"

# Create reports directory
mkdir -p ~/zap-reports

# Run ZAP baseline scan
docker run --rm \
  -v ~/zap-reports:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://$APP_URL \
  -r zap-report-$(date +%Y%m%d-%H%M%S).html \
  -J zap-report-$(date +%Y%m%d-%H%M%S).json \
  -I

# View report
ls -la ~/zap-reports/
```

### Step 9: Verify Prometheus Metrics

```bash
# After Prometheus is deployed (see Section 13):

# Port-forward Prometheus to localhost
kubectl port-forward \
  svc/prometheus-kube-prometheus-prometheus \
  9090:9090 \
  -n monitoring &

# Open in browser (from VM, use curl)
curl http://localhost:9090/metrics

# Or from local machine (forward through SSH)
ssh -i ~/Downloads/vm-jenkins-devops-key.pem \
  -L 9090:localhost:9090 \
  azureuser@<VM-PUBLIC-IP>

# Then open: http://localhost:9090 in your local browser

# Check targets
curl http://localhost:9090/api/v1/targets | python3 -m json.tool

# Query example
curl 'http://localhost:9090/api/v1/query?query=up'
```

### Step 10: Verify Grafana Dashboards

```bash
# After Grafana is deployed (see Section 14):

# Port-forward Grafana to localhost
kubectl port-forward \
  svc/grafana \
  3000:3000 \
  -n monitoring &

# Get Grafana admin password
kubectl get secret \
  --namespace monitoring grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode

# Access Grafana from local machine via SSH tunnel
ssh -i ~/Downloads/vm-jenkins-devops-key.pem \
  -L 3000:localhost:3000 \
  azureuser@<VM-PUBLIC-IP>

# Then open: http://localhost:3000 in your local browser
# Login: admin / <decoded-password>
```

---

## 12. Kubernetes Configuration Files

### 12.1 Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops-app
  labels:
    name: devops-app
    environment: production
```

### 12.2 ConfigMap

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config          # Name referenced by deployments
  namespace: devops-app     # Must match namespace
  labels:
    app: devops-dashboard
    environment: production
data:
  # Non-sensitive configuration values
  NODE_ENV: "production"
  PORT: "5000"                          # Backend port
  FRONTEND_PORT: "3000"                 # Frontend port
  REACT_APP_API_URL: "http://backend-service:5000"  # Internal K8s service DNS
  MONGO_PORT: "27017"                   # MongoDB default port
  APP_NAME: "DevOps Pipeline Dashboard"
  LOG_LEVEL: "info"
  
  # CORS configuration
  CORS_ORIGIN: "*"                      # Tighten in production
  
  # Prometheus metrics
  METRICS_ENABLED: "true"
  METRICS_PORT: "9100"
```

**Field Explanations:**
- `apiVersion: v1` — ConfigMaps are in the core API group
- `kind: ConfigMap` — The resource type
- `metadata.name` — How other resources reference this ConfigMap
- `data` — Key-value pairs injected as environment variables into pods

### 12.3 Secret

```yaml
# k8s/secret.yaml
# ⚠️ IMPORTANT: Base64 encode your values before adding them here
# Command: echo -n "your-value" | base64
# For Azure: Use Key Vault in production instead of plain Secrets

apiVersion: v1
kind: Secret
metadata:
  name: app-secrets          # Name referenced by deployments
  namespace: devops-app
  labels:
    app: devops-dashboard
type: Opaque                 # Generic secret type
data:
  # All values MUST be base64 encoded
  # echo -n "mongodb://mongodb-service:27017/devops_db" | base64
  MONGO_URI: bW9uZ29kYjovL21vbmdvZGItc2VydmljZToyNzAxNy9kZXZvcHNfZGI=
  
  # echo -n "your-jwt-secret-key-min-32-chars" | base64
  JWT_SECRET: eW91ci1qd3Qtc2VjcmV0LWtleS1taW4tMzItY2hhcnM=
  
  # echo -n "devopspipelineacr" | base64
  ACR_USERNAME: ZGV2b3BzcGlwZWxpbmVhY3I=
  
  # echo -n "your-acr-password" | base64
  ACR_PASSWORD: eW91ci1hY3ItcGFzc3dvcmQ=
```

**How to create proper base64 values:**
```bash
# Encode your actual MongoDB URI
echo -n "mongodb://mongodb-service:27017/devops_db" | base64

# Encode JWT secret
echo -n "your-actual-jwt-secret-minimum-32-characters" | base64

# Decode to verify
echo "bW9uZ29kYjovL21vbmdvZGItc2VydmljZToyNzAxNy9kZXZvcHNfZGI=" | base64 --decode
```

### 12.4 Deployment

```yaml
# k8s/deployment.yaml
---
# ═══════════════════════════════════════════
# FRONTEND DEPLOYMENT
# ═══════════════════════════════════════════
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: devops-app
  labels:
    app: frontend
    tier: frontend
    version: "1.0"
spec:
  replicas: 2              # 2 replicas for high availability
  selector:
    matchLabels:
      app: frontend        # Must match template labels
  strategy:
    type: RollingUpdate    # Zero-downtime deployments
    rollingUpdate:
      maxSurge: 1          # Create 1 new pod before killing old
      maxUnavailable: 0    # Never have less than desired pods
  template:
    metadata:
      labels:
        app: frontend
        version: "1.0"
    spec:
      # Use imagePullSecrets if ACR is NOT attached to AKS
      # imagePullSecrets:
      #   - name: acr-secret
      containers:
        - name: frontend
          # ACR image URL (replace with your registry)
          image: devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest
          imagePullPolicy: Always    # Always pull latest (for development)
          ports:
            - containerPort: 3000    # Port your React app serves on
              name: http
          env:
            # Inject from ConfigMap
            - name: NODE_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: NODE_ENV
            - name: REACT_APP_API_URL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: REACT_APP_API_URL
          resources:
            requests:
              memory: "128Mi"        # Minimum memory required
              cpu: "100m"            # 100 millicores = 0.1 CPU
            limits:
              memory: "256Mi"        # Maximum memory allowed
              cpu: "250m"            # Maximum CPU allowed
          # Liveness probe: restart pod if app crashes
          livenessProbe:
            httpGet:
              path: /                # Root path to check
              port: 3000
            initialDelaySeconds: 30  # Wait 30s before first check
            periodSeconds: 10        # Check every 10s
            timeoutSeconds: 5        # Timeout after 5s
            failureThreshold: 3      # Restart after 3 failures
          # Readiness probe: send traffic only when ready
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

---
# ═══════════════════════════════════════════
# BACKEND DEPLOYMENT
# ═══════════════════════════════════════════
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: devops-app
  labels:
    app: backend
    tier: backend
    version: "1.0"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: backend
        version: "1.0"
    spec:
      containers:
        - name: backend
          image: devopspipelineacr.azurecr.io/devops-dashboard-backend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 5000
              name: http
          env:
            - name: NODE_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: NODE_ENV
            - name: PORT
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: PORT
            - name: MONGO_URI
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: MONGO_URI
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: JWT_SECRET
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /api/health      # Your health check endpoint
              port: 5000
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /api/health
              port: 5000
            initialDelaySeconds: 15
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

---
# ═══════════════════════════════════════════
# MONGODB DEPLOYMENT
# ═══════════════════════════════════════════
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  namespace: devops-app
  labels:
    app: mongodb
    tier: database
spec:
  replicas: 1              # Only 1 replica for stateful DB
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:6.0            # Official MongoDB 6.0 image
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 27017
              name: mongodb
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: "admin"
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: JWT_SECRET    # Use a dedicated DB password in production
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          volumeMounts:
            - name: mongodb-storage
              mountPath: /data/db    # MongoDB data directory
      volumes:
        - name: mongodb-storage
          emptyDir: {}               # ⚠️ Use PersistentVolume in production!
```

### 12.5 Service

```yaml
# k8s/service.yaml
---
# Frontend Service — External access via LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: devops-app
  labels:
    app: frontend
  annotations:
    # Azure-specific: Use Standard load balancer (required for AKS)
    service.beta.kubernetes.io/azure-load-balancer-sku: "standard"
spec:
  type: LoadBalancer          # Creates Azure Load Balancer with public IP
  selector:
    app: frontend             # Route traffic to pods with this label
  ports:
    - name: http
      port: 80                # External port (what users access)
      targetPort: 3000        # Internal pod port
      protocol: TCP

---
# Backend Service — Internal only (ClusterIP)
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: devops-app
  labels:
    app: backend
spec:
  type: ClusterIP             # Internal only, not exposed externally
  selector:
    app: backend
  ports:
    - name: http
      port: 5000              # Internal service port
      targetPort: 5000        # Pod port
      protocol: TCP

---
# MongoDB Service — Internal only
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: devops-app
  labels:
    app: mongodb
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - name: mongodb
      port: 27017
      targetPort: 27017
      protocol: TCP
```

**Service Type Explanations:**

| Type | When To Use | External Access |
|---|---|---|
| `ClusterIP` | Internal communication only | ❌ No |
| `NodePort` | Testing/development | ✅ Via node IP |
| `LoadBalancer` | Production external access | ✅ Public IP |

### 12.6 Ingress

```yaml
# k8s/ingress.yaml
# First install NGINX Ingress Controller:
# kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: devops-app
  labels:
    app: devops-dashboard
  annotations:
    # Use NGINX as the ingress controller
    kubernetes.io/ingress.class: "nginx"
    
    # Rewrite target — strip /api prefix when routing to backend
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    
    # Enable CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    
    # Rate limiting (security best practice)
    nginx.ingress.kubernetes.io/limit-rps: "100"
    
    # Request body size
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    
    # SSL redirect (uncomment if you have TLS configured)
    # nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
spec:
  # Uncomment and configure for HTTPS with cert-manager
  # tls:
  #   - hosts:
  #       - devops-dashboard.example.com
  #     secretName: tls-secret
  
  rules:
    # If you have a domain, replace with your domain
    # - host: devops-dashboard.example.com
    - http:
        paths:
          # Route /api/* to backend service
          - path: /api(/|$)(.*)
            pathType: Prefix
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
```

**Install NGINX Ingress Controller:**
```bash
# Install NGINX Ingress Controller for AKS
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

# Verify installation
kubectl get pods -n ingress-nginx

# Get external IP of Ingress
kubectl get service -n ingress-nginx ingress-nginx-controller
# Wait for EXTERNAL-IP to be assigned (2-5 minutes)
```

---

## 13. Prometheus on AKS

### 13.1 Install Prometheus using Helm

```bash
# Install Helm if not already installed
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version

# Add Prometheus Helm repository
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install kube-prometheus-stack (includes Prometheus + Grafana + Alertmanager)
helm install prometheus \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.retention=7d \
  --set grafana.enabled=true \
  --set alertmanager.enabled=false \
  --set nodeExporter.enabled=true \
  --set kubeStateMetrics.enabled=true

# Verify installation (wait 2-3 minutes)
kubectl get pods -n monitoring

# Expected output:
# NAME                                                  READY   STATUS    RESTARTS
# prometheus-grafana-xxxx                               3/3     Running   0
# prometheus-kube-prometheus-operator-xxxx              1/1     Running   0
# prometheus-kube-prometheus-prometheus-0               2/2     Running   0
# prometheus-kube-state-metrics-xxxx                    1/1     Running   0
# prometheus-prometheus-node-exporter-xxxx              1/1     Running   0
```

### 13.2 Custom Prometheus Configuration

```yaml
# prometheus-values.yaml (use with helm upgrade)
prometheus:
  prometheusSpec:
    retention: 7d                    # Keep 7 days of metrics
    retentionSize: "5GB"             # Max storage size
    
    # Scrape interval
    scrapeInterval: 15s
    evaluationInterval: 15s
    
    # Additional scrape configs for MERN app
    additionalScrapeConfigs:
      - job_name: 'devops-dashboard-backend'
        static_configs:
          - targets: ['backend-service.devops-app.svc.cluster.local:5000']
        metrics_path: '/metrics'     # Your backend metrics endpoint
        scrape_interval: 15s
      
      - job_name: 'devops-dashboard-frontend'
        static_configs:
          - targets: ['frontend-service.devops-app.svc.cluster.local:3000']
        metrics_path: '/metrics'
        scrape_interval: 30s

grafana:
  enabled: true
  adminPassword: "AdminP@ssw0rd!"   # Change this!
  
  # Persistence (saves dashboards)
  persistence:
    enabled: false                   # Disable for cost savings

resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

```bash
# Apply custom values
helm upgrade prometheus \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f prometheus-values.yaml
```

### 13.3 Access Prometheus

```bash
# Port-forward Prometheus UI
kubectl port-forward \
  --namespace monitoring \
  svc/prometheus-kube-prometheus-prometheus \
  9090:9090

# In another terminal (or use SSH tunnel):
# Access: http://localhost:9090

# Useful PromQL queries:
# CPU usage by pod:
# sum(rate(container_cpu_usage_seconds_total{namespace="devops-app"}[5m])) by (pod)

# Memory usage:
# sum(container_memory_usage_bytes{namespace="devops-app"}) by (pod)

# HTTP request rate:
# rate(http_requests_total{job="devops-dashboard-backend"}[5m])

# Pod up/down status:
# up{namespace="devops-app"}
```

---

## 14. Grafana on AKS

### 14.1 Access Grafana

```bash
# Grafana is included in the kube-prometheus-stack

# Get admin password
kubectl get secret \
  --namespace monitoring \
  prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
echo ""  # New line

# Port-forward Grafana
kubectl port-forward \
  --namespace monitoring \
  svc/prometheus-grafana \
  3000:80

# Access: http://localhost:3000
# Username: admin
# Password: <decoded above>
```

### 14.2 Configure Prometheus Data Source

```
Grafana UI: http://localhost:3000
    │
    ▼
Left sidebar → Connections → Data sources
    │
    ▼
Add data source → Prometheus
    ┌──────────────────────────────────────────────────┐
    │ Name:        Prometheus                          │
    │ URL:         http://prometheus-kube-prometheus-  │
    │              prometheus.monitoring.svc.cluster   │
    │              .local:9090                         │
    │ Access:      Server (default)                    │
    └──────────────────────────────────────────────────┘
    │
    ▼
Click [Save & test]
    │
    ▼
✅ "Data source is working"
```

### 14.3 Import Pre-built Dashboards

```
Grafana UI
    │
    ▼
Left sidebar → Dashboards → Import
    │
    ▼
Import by ID (paste these Grafana.com IDs):
```

| Dashboard ID | Name | Purpose |
|---|---|---|
| 3119 | Kubernetes Cluster Monitoring | Cluster overview |
| 6417 | Kubernetes Pods | Pod metrics |
| 1860 | Node Exporter Full | Node CPU/Memory/Disk |
| 13770 | 1 K8s for Prometheus Dashboard | K8s comprehensive |

```bash
# Import via Grafana API (automated)
GRAFANA_URL="http://localhost:3000"
GRAFANA_PASS=$(kubectl get secret --namespace monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode)

# Import Kubernetes dashboard (ID: 3119)
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"dashboard":{"id":null,"uid":null},"folderId":0,"overwrite":true,"inputs":[],"pluginId":"grafana-dashboards","gnetId":3119}' \
  http://admin:${GRAFANA_PASS}@localhost:3000/api/dashboards/import
```

---

## 15. Complete Jenkinsfile

```groovy
// Jenkinsfile
// Production-ready CI/CD Pipeline for DevOps Pipeline Dashboard
// Target: Azure Container Registry + Azure Kubernetes Service

pipeline {
    agent any

    // ═══════════════════════════════════════
    // ENVIRONMENT VARIABLES
    // ═══════════════════════════════════════
    environment {
        // Azure Container Registry
        ACR_NAME        = 'devopspipelineacr'
        ACR_URL         = 'devopspipelineacr.azurecr.io'
        
        // Docker image names
        FRONTEND_IMAGE  = "${ACR_URL}/devops-dashboard-frontend"
        BACKEND_IMAGE   = "${ACR_URL}/devops-dashboard-backend"
        
        // Build tag (use build number for uniqueness)
        IMAGE_TAG       = "${BUILD_NUMBER}"
        LATEST_TAG      = 'latest'
        
        // Kubernetes
        K8S_NAMESPACE   = 'devops-app'
        
        // SonarQube
        SONAR_PROJECT   = 'devops-pipeline-dashboard'
        
        // Kubernetes manifests directory
        K8S_DIR         = 'k8s'
        
        // Reports directory
        REPORTS_DIR     = 'security-reports'
    }

    // ═══════════════════════════════════════
    // PIPELINE OPTIONS
    // ═══════════════════════════════════════
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
        disableConcurrentBuilds()
    }

    // ═══════════════════════════════════════
    // PIPELINE TRIGGERS
    // ═══════════════════════════════════════
    triggers {
        githubPush()        // Trigger on GitHub push (webhook required)
    }

    // ═══════════════════════════════════════
    // TOOLS
    // ═══════════════════════════════════════
    tools {
        nodejs 'NodeJS-18'  // Must match name in Jenkins Tools config
    }

    // ═══════════════════════════════════════
    // PIPELINE STAGES
    // ═══════════════════════════════════════
    stages {

        // ───────────────────────────────────
        // STAGE 1: CHECKOUT
        // ───────────────────────────────────
        stage('Checkout Source Code') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║     Stage 1: Checkout Source       ║'
                echo '╚════════════════════════════════════╝'
                
                cleanWs()   // Clean workspace before checkout
                
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/YOUR-USERNAME/devops-pipeline-dashboard.git',
                        credentialsId: 'github-token'
                    ]]
                ])
                
                script {
                    env.GIT_COMMIT_HASH = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    
                    env.GIT_BRANCH = sh(
                        script: 'git rev-parse --abbrev-ref HEAD',
                        returnStdout: true
                    ).trim()
                    
                    echo "Branch: ${env.GIT_BRANCH}"
                    echo "Commit: ${env.GIT_COMMIT_HASH}"
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 2: INSTALL DEPENDENCIES
        // ───────────────────────────────────
        stage('Install Dependencies') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 2: Install Dependencies    ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    echo "Node version: $(node --version)"
                    echo "NPM version: $(npm --version)"
                    
                    # Install root/frontend dependencies
                    echo "Installing frontend dependencies..."
                    npm ci --prefer-offline
                    
                    # Install backend dependencies
                    echo "Installing backend dependencies..."
                    cd server && npm ci --prefer-offline && cd ..
                '''
            }
        }

        // ───────────────────────────────────
        // STAGE 3: RUN TESTS
        // ───────────────────────────────────
        stage('Run Unit Tests') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║     Stage 3: Unit Tests            ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    # Run tests with coverage
                    npm test -- \
                        --watchAll=false \
                        --coverage \
                        --coverageReporters=lcov \
                        --coverageReporters=text-summary \
                        2>&1 | tee test-results.log
                    
                    echo "Test results:"
                    cat test-results.log
                '''
            }
            post {
                always {
                    // Archive test results
                    junit '**/test-results.xml'
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 4: SONARQUBE SCAN
        // ───────────────────────────────────
        stage('SonarQube Analysis') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 4: SonarQube SAST Scan     ║'
                echo '╚════════════════════════════════════╝'
                
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT} \
                            -Dsonar.projectName="DevOps Pipeline Dashboard" \
                            -Dsonar.projectVersion=${BUILD_NUMBER} \
                            -Dsonar.sources=src,server \
                            -Dsonar.exclusions="**/node_modules/**,**/coverage/**,**/*.test.js" \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                            -Dsonar.host.url=http://localhost:9000
                    '''
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 5: QUALITY GATE
        // ───────────────────────────────────
        stage('Quality Gate') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 5: Quality Gate Check      ║'
                echo '╚════════════════════════════════════╝'
                
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 6: BUILD APPLICATION
        // ───────────────────────────────────
        stage('Build Application') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 6: Production Build        ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    echo "Building frontend for production..."
                    npm run build
                    
                    echo "Build artifacts:"
                    ls -la build/
                    du -sh build/
                '''
            }
        }

        // ───────────────────────────────────
        // STAGE 7: DOCKER BUILD
        // ───────────────────────────────────
        stage('Docker Build') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 7: Docker Image Build      ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    echo "Building Docker images..."
                    
                    # Build frontend image
                    docker build \
                        --tag ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                        --tag ${FRONTEND_IMAGE}:${LATEST_TAG} \
                        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                        --build-arg VCS_REF=${GIT_COMMIT_HASH} \
                        --build-arg VERSION=${IMAGE_TAG} \
                        --file Dockerfile \
                        --no-cache \
                        .
                    
                    # Build backend image
                    docker build \
                        --tag ${BACKEND_IMAGE}:${IMAGE_TAG} \
                        --tag ${BACKEND_IMAGE}:${LATEST_TAG} \
                        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                        --build-arg VCS_REF=${GIT_COMMIT_HASH} \
                        --build-arg VERSION=${IMAGE_TAG} \
                        --file Dockerfile.backend \
                        --no-cache \
                        .
                    
                    echo "Images built successfully:"
                    docker images | grep devopspipelineacr
                '''
            }
        }

        // ───────────────────────────────────
        // STAGE 8: TRIVY SECURITY SCAN
        // ───────────────────────────────────
        stage('Trivy Image Scan') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 8: Trivy CVE Scan          ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    mkdir -p ${REPORTS_DIR}
                    
                    echo "Scanning frontend image..."
                    trivy image \
                        --format table \
                        --severity CRITICAL,HIGH,MEDIUM \
                        --output ${REPORTS_DIR}/trivy-frontend-report.txt \
                        --no-progress \
                        ${FRONTEND_IMAGE}:${IMAGE_TAG}
                    
                    echo "Scanning backend image..."
                    trivy image \
                        --format table \
                        --severity CRITICAL,HIGH,MEDIUM \
                        --output ${REPORTS_DIR}/trivy-backend-report.txt \
                        --no-progress \
                        ${BACKEND_IMAGE}:${IMAGE_TAG}
                    
                    echo "Generating HTML reports..."
                    trivy image \
                        --format template \
                        --template "@~/.trivy/templates/html.tpl" \
                        --output ${REPORTS_DIR}/trivy-frontend-report.html \
                        ${FRONTEND_IMAGE}:${IMAGE_TAG}
                    
                    trivy image \
                        --format template \
                        --template "@~/.trivy/templates/html.tpl" \
                        --output ${REPORTS_DIR}/trivy-backend-report.html \
                        ${BACKEND_IMAGE}:${IMAGE_TAG}
                    
                    echo "Trivy Scan Results:"
                    cat ${REPORTS_DIR}/trivy-frontend-report.txt
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: "${REPORTS_DIR}/trivy-*.html", fingerprint: true
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 9: PUSH TO ACR
        // ───────────────────────────────────
        stage('Push to Azure Container Registry') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 9: Push Images to ACR      ║'
                echo '╚════════════════════════════════════╝'
                
                withCredentials([usernamePassword(
                    credentialsId: 'acr-credentials',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh '''
                        # Login to ACR
                        echo $ACR_PASS | docker login ${ACR_URL} \
                            --username $ACR_USER \
                            --password-stdin
                        
                        echo "Pushing frontend image..."
                        docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}
                        docker push ${FRONTEND_IMAGE}:${LATEST_TAG}
                        
                        echo "Pushing backend image..."
                        docker push ${BACKEND_IMAGE}:${IMAGE_TAG}
                        docker push ${BACKEND_IMAGE}:${LATEST_TAG}
                        
                        echo "Images pushed successfully!"
                        
                        # Logout from ACR
                        docker logout ${ACR_URL}
                    '''
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 10: DEPLOY TO AKS
        // ───────────────────────────────────
        stage('Deploy to AKS') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 10: Deploy to AKS          ║'
                echo '╚════════════════════════════════════╝'
                
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        # Set KUBECONFIG
                        export KUBECONFIG=$KUBECONFIG
                        
                        # Verify kubectl connection
                        kubectl cluster-info
                        kubectl get nodes
                        
                        # Create namespace if not exists
                        kubectl create namespace ${K8S_NAMESPACE} \
                            --dry-run=client -o yaml | kubectl apply -f -
                        
                        # Apply ConfigMap and Secrets first
                        kubectl apply -f ${K8S_DIR}/configmap.yaml -n ${K8S_NAMESPACE}
                        kubectl apply -f ${K8S_DIR}/secret.yaml -n ${K8S_NAMESPACE}
                        
                        # Update image tags in deployment files
                        sed -i "s|:latest|:${IMAGE_TAG}|g" ${K8S_DIR}/deployment.yaml
                        
                        # Apply deployments
                        kubectl apply -f ${K8S_DIR}/deployment.yaml -n ${K8S_NAMESPACE}
                        kubectl apply -f ${K8S_DIR}/service.yaml -n ${K8S_NAMESPACE}
                        kubectl apply -f ${K8S_DIR}/ingress.yaml -n ${K8S_NAMESPACE}
                        
                        echo "Deployment applied successfully!"
                    '''
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 11: VERIFY DEPLOYMENT
        // ───────────────────────────────────
        stage('Verify Deployment') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 11: Verify Deployment      ║'
                echo '╚════════════════════════════════════╝'
                
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        
                        echo "Waiting for frontend rollout..."
                        kubectl rollout status \
                            deployment/frontend-deployment \
                            -n ${K8S_NAMESPACE} \
                            --timeout=300s
                        
                        echo "Waiting for backend rollout..."
                        kubectl rollout status \
                            deployment/backend-deployment \
                            -n ${K8S_NAMESPACE} \
                            --timeout=300s
                        
                        echo "Current pod status:"
                        kubectl get pods -n ${K8S_NAMESPACE} -o wide
                        
                        echo "Services:"
                        kubectl get services -n ${K8S_NAMESPACE}
                        
                        echo "Ingress:"
                        kubectl get ingress -n ${K8S_NAMESPACE}
                        
                        # Get app URL
                        APP_IP=$(kubectl get service frontend-service \
                            -n ${K8S_NAMESPACE} \
                            -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                        echo "Application URL: http://${APP_IP}"
                        
                        # Health check
                        if [ -n "$APP_IP" ]; then
                            sleep 30  # Wait for pods to be fully ready
                            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://${APP_IP} || echo "000")
                            echo "HTTP Status Code: ${HTTP_CODE}"
                            if [ "$HTTP_CODE" = "200" ]; then
                                echo "✅ Application is healthy!"
                            else
                                echo "⚠️ Application returned HTTP ${HTTP_CODE}"
                            fi
                        fi
                    '''
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 12: OWASP ZAP DAST SCAN
        // ───────────────────────────────────
        stage('OWASP ZAP DAST Scan') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 12: OWASP ZAP DAST Scan   ║'
                echo '╚════════════════════════════════════╝'
                
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        
                        # Get application IP
                        APP_IP=$(kubectl get service frontend-service \
                            -n ${K8S_NAMESPACE} \
                            -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                        
                        if [ -z "$APP_IP" ]; then
                            echo "⚠️ Could not get app IP, skipping ZAP scan"
                            exit 0
                        fi
                        
                        echo "Running OWASP ZAP scan against: http://${APP_IP}"
                        mkdir -p ${REPORTS_DIR}
                        
                        # Run ZAP Baseline scan
                        docker run --rm \
                            -v ${WORKSPACE}/${REPORTS_DIR}:/zap/wrk/:rw \
                            -t ghcr.io/zaproxy/zaproxy:stable \
                            zap-baseline.py \
                            -t http://${APP_IP} \
                            -r zap-report-${BUILD_NUMBER}.html \
                            -J zap-report-${BUILD_NUMBER}.json \
                            -I \
                            2>&1 | tee zap-scan-output.log || true
                        
                        echo "ZAP scan completed."
                        echo "Report saved to: ${REPORTS_DIR}/zap-report-${BUILD_NUMBER}.html"
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: "${REPORTS_DIR}/zap-report-*.html", fingerprint: true
                }
            }
        }

        // ───────────────────────────────────
        // STAGE 13: ARCHIVE REPORTS
        // ───────────────────────────────────
        stage('Archive Security Reports') {
            steps {
                echo '╔════════════════════════════════════╗'
                echo '║   Stage 13: Archive Reports        ║'
                echo '╚════════════════════════════════════╝'
                
                sh '''
                    echo "All security reports:"
                    ls -la ${REPORTS_DIR}/
                    
                    echo "Report sizes:"
                    du -sh ${REPORTS_DIR}/*
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: "${REPORTS_DIR}/**/*", fingerprint: true
                    
                    // Publish HTML reports in Jenkins UI
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: "${REPORTS_DIR}",
                        reportFiles: 'trivy-frontend-report.html',
                        reportName: 'Trivy Frontend Security Report'
                    ])
                    
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: "${REPORTS_DIR}",
                        reportFiles: "zap-report-${BUILD_NUMBER}.html",
                        reportName: 'OWASP ZAP Security Report'
                    ])
                }
            }
        }

    } // end stages

    // ═══════════════════════════════════════
    // POST-PIPELINE ACTIONS
    // ═══════════════════════════════════════
    post {
        always {
            echo '╔════════════════════════════════════╗'
            echo '║       Pipeline Completed           ║'
            echo '╚════════════════════════════════════╝'
            
            // Clean up Docker images to free space
            sh '''
                echo "Cleaning up local Docker images..."
                docker rmi ${FRONTEND_IMAGE}:${IMAGE_TAG} || true
                docker rmi ${BACKEND_IMAGE}:${IMAGE_TAG} || true
                docker system prune -f --volumes || true
                echo "Cleanup done. Remaining images:"
                docker images
            '''
        }
        
        success {
            echo '✅ Pipeline completed SUCCESSFULLY!'
            echo "✅ Build: #${BUILD_NUMBER}"
            echo "✅ Commit: ${GIT_COMMIT_HASH}"
            echo "✅ Images pushed to: ${ACR_URL}"
            echo "✅ Deployed to AKS namespace: ${K8S_NAMESPACE}"
        }
        
        failure {
            echo '❌ Pipeline FAILED!'
            echo "❌ Build: #${BUILD_NUMBER}"
            echo "❌ Check the logs above for details"
        }
        
        unstable {
            echo '⚠️ Pipeline is UNSTABLE (tests may have failed)'
        }
    }

} // end pipeline
```

---

## 16. Verification Procedures

### 16.1 Verify Azure VM

```bash
# From local machine:
ssh -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<VM-PUBLIC-IP>

# Expected: SSH prompt
# azureuser@vm-jenkins-devops:~$

# Check VM resources
free -h          # Memory usage
df -h            # Disk usage
nproc            # CPU count
htop             # Interactive resource monitor (q to quit)

# Expected:
# Memory: ~8GB total, <4GB used normally
# Disk: 30GB total
# CPU: 2 cores
```

### 16.2 Verify Jenkins

```bash
# Check Jenkins service status
sudo systemctl status jenkins

# Expected output:
# ● jenkins.service - Jenkins Automation Server
#    Active: active (running)

# Check Jenkins is accessible
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080
# Expected: 200

# Browser: http://<VM-PUBLIC-IP>:8080
# Should see Jenkins Dashboard
```

### 16.3 Verify Docker

```bash
# Check Docker daemon
sudo systemctl status docker

# Test Docker works
docker run --rm hello-world

# Expected:
# Hello from Docker!
# This message shows that your installation appears to be working correctly.

# Check Docker info
docker info | grep -E "Server Version|Storage Driver|Containers|Images"

# Check Docker is accessible to Jenkins user
sudo su - jenkins -s /bin/bash -c "docker ps"
# Expected: CONTAINER ID   IMAGE ...  (no permission error)
```

### 16.4 Verify SonarQube

```bash
# Check SonarQube container
docker ps | grep sonarqube

# Expected:
# abc123def456   sonarqube:lts-community   "..."   Up X hours   0.0.0.0:9000->9000/tcp   sonarqube

# Check SonarQube API
curl http://localhost:9000/api/system/status

# Expected:
# {"status":"UP","version":"10.x.x"}

# Browser: http://<VM-PUBLIC-IP>:9000
# Should see SonarQube dashboard
```

### 16.5 Verify Trivy

```bash
# Check Trivy version
trivy --version

# Expected:
# Version: 0.5x.x

# Run test scan
trivy image --quiet alpine:latest

# Expected: Scan results table (or "No vulnerabilities found")
```

### 16.6 Verify ACR

```bash
# List repositories
az acr repository list --name devopspipelineacr --output table

# Expected:
# Result
# devops-dashboard-frontend
# devops-dashboard-backend

# Check image tags
az acr repository show-tags \
  --name devopspipelineacr \
  --repository devops-dashboard-frontend \
  --output table

# Expected:
# Result
# latest
# 1
# 2
# ...
```

### 16.7 Verify AKS

```bash
# Check nodes
kubectl get nodes

# Expected:
# NAME                              STATUS   ROLES   AGE   VERSION
# aks-agentpool-12345678-vmss000000   Ready    agent   Xd    v1.29.x

# Check all resources in devops-app namespace
kubectl get all -n devops-app

# Expected:
# NAME                                      READY   STATUS    RESTARTS   AGE
# pod/frontend-deployment-xxx-yyy           1/1     Running   0          Xh
# pod/backend-deployment-xxx-yyy            1/1     Running   0          Xh
# pod/mongodb-deployment-xxx-yyy            1/1     Running   0          Xh
#
# NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
# service/frontend-service   LoadBalancer   10.0.200.1     20.xxx.xxx.xxx   80:31234/TCP
# service/backend-service    ClusterIP      10.0.200.2     <none>           5000/TCP
# service/mongodb-service    ClusterIP      10.0.200.3     <none>           27017/TCP
```

### 16.8 Verify Pods

```bash
# Get pod details
kubectl get pods -n devops-app -o wide

# Describe a specific pod
kubectl describe pod <pod-name> -n devops-app

# View pod logs
kubectl logs <pod-name> -n devops-app

# View pod logs (follow/tail)
kubectl logs <pod-name> -n devops-app --follow

# Check pod resource usage
kubectl top pods -n devops-app
```

### 16.9 Verify Services

```bash
# Get all services
kubectl get services -n devops-app

# Get service details
kubectl describe service frontend-service -n devops-app

# Test service internally
kubectl run test-pod --rm -it \
  --image=curlimages/curl \
  --restart=Never \
  -n devops-app \
  -- curl http://frontend-service:80

# Get external IP
kubectl get service frontend-service \
  -n devops-app \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### 16.10 Verify Ingress

```bash
# Get ingress
kubectl get ingress -n devops-app

# Expected:
# NAME          CLASS   HOSTS   ADDRESS          PORTS   AGE
# app-ingress   nginx   *       20.xxx.xxx.xxx   80      Xh

# Describe ingress
kubectl describe ingress app-ingress -n devops-app

# Test ingress
INGRESS_IP=$(kubectl get ingress app-ingress -n devops-app \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  
curl -v http://$INGRESS_IP/        # Frontend
curl -v http://$INGRESS_IP/api/    # Backend API
```

### 16.11 Verify Prometheus

```bash
# Check Prometheus pods
kubectl get pods -n monitoring | grep prometheus

# Port-forward and check
kubectl port-forward svc/prometheus-kube-prometheus-prometheus \
  9090:9090 -n monitoring &

sleep 3

# Check targets
curl http://localhost:9090/api/v1/targets 2>/dev/null | \
  python3 -c "import sys,json; d=json.load(sys.stdin); \
  [print(t['labels']['job'], t['health']) for t in d['data']['activeTargets']]"

# Expected: list of targets all showing "up"

# Kill port-forward
kill %1
```

### 16.12 Verify Grafana

```bash
# Check Grafana pods
kubectl get pods -n monitoring | grep grafana

# Expected:
# prometheus-grafana-xxx-yyy   3/3     Running   0   Xd

# Get admin password
GRAFANA_PASS=$(kubectl get secret --namespace monitoring \
  prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode)

# Port-forward
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring &

sleep 3

# Test login via API
curl -s -o /dev/null -w "%{http_code}" \
  http://admin:${GRAFANA_PASS}@localhost:3000/api/health

# Expected: 200

kill %1
```

---

## 17. Troubleshooting Guide

### 17.1 Azure Login Issues

```bash
# Issue: az login fails or times out
# Solution 1: Use device code flow
az login --use-device-code
# Go to https://microsoft.com/devicelogin and enter the code

# Solution 2: Check subscription
az account show
az account list --output table

# Solution 3: Set correct subscription
az account set --subscription "Azure for Students"

# Solution 4: Clear Azure CLI cache
rm -rf ~/.azure
az login
```

### 17.2 VM SSH Issues

```bash
# Issue: Permission denied (publickey)
# Solution 1: Fix key permissions
chmod 400 ~/Downloads/vm-jenkins-devops-key.pem

# Solution 2: Verify the key name matches VM
ls -la ~/Downloads/*.pem

# Solution 3: Use verbose mode to debug
ssh -v -i ~/Downloads/vm-jenkins-devops-key.pem azureuser@<VM-IP>

# Issue: Connection refused on port 22
# Solution: Check NSG rule allows port 22 from your IP
# Azure Portal → VM → Networking → Inbound port rules

# Issue: Connection timeout
# Solution: Check VM is Running (not Stopped)
# Azure Portal → VM → Overview → Status: Running

# Solution: Verify public IP
az vm list-ip-addresses \
  --resource-group rg-devops-pipeline-prod \
  --name vm-jenkins-devops \
  --output table
```

### 17.3 Docker Issues

```bash
# Issue: Permission denied on Docker socket
# Solution: Add user to docker group and re-login
sudo usermod -aG docker $USER
newgrp docker
# OR logout and login again

# For Jenkins specifically:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# Issue: Docker daemon not running
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker

# Issue: Docker build fails (out of disk)
df -h                           # Check disk
docker system prune -af         # Remove all unused images/containers
docker volume prune -f          # Remove unused volumes

# Issue: Cannot pull from ACR
# Solution: Re-login to ACR
az acr login --name devopspipelineacr
# OR
docker login devopspipelineacr.azurecr.io
```

### 17.4 Jenkins Issues

```bash
# Issue: Jenkins not starting
sudo systemctl status jenkins
sudo journalctl -u jenkins -n 50 --no-pager

# Common fix: Java not found
which java
sudo update-alternatives --config java

# Issue: Jenkins workspace permissions
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/
sudo chmod -R 755 /var/lib/jenkins/workspace/

# Issue: Jenkins plugin not working
# Go to: Manage Jenkins → Plugins → Installed
# Verify plugin is installed and enabled
# Restart Jenkins:
sudo systemctl restart jenkins

# Issue: Git checkout fails
# Verify GitHub token in credentials
# Test manually:
git clone https://<GITHUB-TOKEN>@github.com/<USER>/<REPO>.git /tmp/test-clone

# Issue: Pipeline hangs on Quality Gate
# Check SonarQube webhook is configured:
# SonarQube → Administration → Webhooks
# Add: http://localhost:8080/sonarqube-webhook/

# Issue: kubectl not found in Jenkins
sudo apt-get install -y kubectl
which kubectl
# OR
sudo ln -s /usr/local/bin/kubectl /usr/bin/kubectl
```

### 17.5 SonarQube Issues

```bash
# Issue: SonarQube container won't start
docker logs sonarqube 2>&1 | tail -50

# Common fix: vm.max_map_count too low
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

# Common fix: Elasticsearch memory
docker update --memory="2g" sonarqube
docker restart sonarqube

# Issue: Cannot login to SonarQube
# Default credentials: admin/admin
# If locked out, restart container:
docker restart sonarqube

# Issue: Quality Gate always fails
# Check SonarQube conditions:
# SonarQube → Quality Gates → Sonar way → Conditions
# Adjust thresholds for first-time setup

# Issue: sonar-scanner not found
# Install manually:
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip -d /opt/
sudo ln -s /opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner /usr/local/bin/sonar-scanner
```

### 17.6 Trivy Issues

```bash
# Issue: Trivy database update fails
trivy image --download-db-only
# OR
export TRIVY_SKIP_DB_UPDATE=true
trivy image --skip-update <image-name>

# Issue: Trivy not finding vulnerabilities (expected some)
trivy image --severity ALL <image-name>

# Issue: Trivy times out on large images
trivy image --timeout 10m <image-name>

# Issue: Cannot scan ACR images
# Login to ACR first
az acr login --name devopspipelineacr
docker pull devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest
trivy image devopspipelineacr.azurecr.io/devops-dashboard-frontend:latest
```

### 17.7 ACR Authentication Issues

```bash
# Issue: Unauthorized error when pushing
# Solution: Re-login
az acr login --name devopspipelineacr

# Issue: ACR credentials expired
# Solution: Generate new credentials
az acr credential renew \
  --name devopspipelineacr \
  --password-name password

# Issue: AKS cannot pull from ACR
# Solution: Re-attach ACR to AKS
az aks update \
  --name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --attach-acr devopspipelineacr

# Verify attachment
az aks check-acr \
  --name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --acr devopspipelineacr.azurecr.io
```

### 17.8 AKS Deployment Issues

```bash
# Issue: Pods stuck in Pending
kubectl describe pod <pod-name> -n devops-app
# Look for: Events section at bottom

# Common causes:
# 1. Insufficient CPU/Memory
kubectl describe node | grep -A 5 "Conditions:"

# 2. Image pull failure
kubectl get pod <pod-name> -n devops-app -o jsonpath='{.status.containerStatuses}'

# 3. Check events
kubectl get events -n devops-app --sort-by='.lastTimestamp'

# Issue: Pods in CrashLoopBackOff
# Check logs
kubectl logs <pod-name> -n devops-app --previous

# Issue: Service has no external IP
# LoadBalancer takes 2-5 minutes
kubectl get service frontend-service -n devops-app --watch
# Wait for EXTERNAL-IP to change from <pending>

# Issue: Cannot connect to pod
kubectl exec -it <pod-name> -n devops-app -- /bin/sh
# OR
kubectl exec -it <pod-name> -n devops-app -- /bin/bash
```

### 17.9 kubectl Issues

```bash
# Issue: kubectl: command not found
sudo apt-get install -y kubectl
# OR
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Issue: connection refused / no server configured
az aks get-credentials \
  --resource-group rg-devops-pipeline-prod \
  --name aks-devops-cluster \
  --overwrite-existing

# Issue: Unauthorized
az login
az aks get-credentials \
  --resource-group rg-devops-pipeline-prod \
  --name aks-devops-cluster \
  --overwrite-existing

# Issue: kubectl context wrong
kubectl config get-contexts
kubectl config use-context aks-devops-cluster
```

### 17.10 Prometheus Issues

```bash
# Issue: Prometheus pods not running
kubectl describe pod -n monitoring | grep -A 10 Events

# Issue: Targets not showing up
# Check scrape configuration
kubectl get configmap \
  prometheus-kube-prometheus-prometheus \
  -n monitoring -o yaml

# Issue: No metrics from application
# Verify backend exposes /metrics endpoint
curl http://<backend-service-ip>:5000/metrics

# Add prom-client to your Express backend:
# npm install prom-client
# Then expose /metrics route

# Issue: Out of disk on Prometheus
kubectl exec -it prometheus-kube-prometheus-prometheus-0 \
  -n monitoring -c prometheus -- df -h /prometheus
```

### 17.11 Grafana Issues

```bash
# Issue: Cannot login to Grafana
# Get current password
kubectl get secret --namespace monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode

# Issue: No data in dashboards
# Check Prometheus data source is configured
# Grafana → Connections → Data sources → Prometheus → Test

# Issue: Dashboard shows "No data"
# Verify PromQL query is correct
# Open Prometheus UI: http://localhost:9090
# Test query in Prometheus before using in Grafana

# Issue: Port-forward drops
# Use nohup to keep it running
nohup kubectl port-forward svc/prometheus-grafana \
  3000:80 -n monitoring > /dev/null 2>&1 &
```

---

## 18. Best Practices

### 18.1 Security Best Practices

```
SECURITY CHECKLIST
══════════════════

✅ Never commit .env files to Git
✅ Use Kubernetes Secrets for sensitive data (not ConfigMaps)
✅ Enable RBAC on AKS
✅ Restrict NSG rules to minimum required ports
✅ Use imagePullPolicy: Always to prevent stale images
✅ Run containers as non-root user
✅ Set resource limits on all containers
✅ Enable readiness and liveness probes
✅ Use network policies to restrict pod communication
✅ Rotate ACR credentials regularly
✅ Set budget alerts in Azure Cost Management
✅ Enable Azure Defender (if budget allows)
✅ Store secrets in Azure Key Vault (production)
✅ Use least-privilege service accounts
✅ Scan images before pushing with Trivy
✅ Review SonarQube quality gate before every release
✅ Archive and review OWASP ZAP reports
```

### 18.2 Azure Cost Best Practices

```bash
# Stop VM when not in use
az vm deallocate \
  --resource-group rg-devops-pipeline-prod \
  --name vm-jenkins-devops

# Start VM when needed
az vm start \
  --resource-group rg-devops-pipeline-prod \
  --name vm-jenkins-devops

# Scale AKS to 0 nodes when not testing
az aks nodepool scale \
  --cluster-name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --name agentpool \
  --node-count 0

# Scale back up
az aks nodepool scale \
  --cluster-name aks-devops-cluster \
  --resource-group rg-devops-pipeline-prod \
  --name agentpool \
  --node-count 1

# Set budget alert
az consumption budget create \
  --budget-name "StudentBudget" \
  --amount 80 \
  --time-grain Monthly \
  --category Cost

# Delete entire resource group when done
az group delete \
  --name rg-devops-pipeline-prod \
  --yes --no-wait
```

### 18.3 Kubernetes Best Practices

```yaml
# Always include these in deployments:

spec:
  template:
    spec:
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
      
      containers:
        - name: app
          # Resource limits (required)
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "250m"
          
          # Health probes (required)
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          
          # Container security
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
```

### 18.4 CI/CD Best Practices

```
PIPELINE BEST PRACTICES
════════════════════════

1. Build once, deploy many
   - Build Docker image once
   - Use same image across environments

2. Use specific image tags (not :latest in production)
   - Tag with build number: :${BUILD_NUMBER}
   - Use :latest for development only

3. Fail fast
   - Run fast tests first (unit tests)
   - Run slow scans after (OWASP ZAP)

4. Security gates
   - Block on CRITICAL Trivy findings
   - Block on Quality Gate failures
   - Review (don't necessarily block) ZAP findings

5. Clean workspace
   - Always clean before build
   - Remove Docker images after push

6. Archive all reports
   - Keep security reports for audit trail
   - Set retention (10 builds recommended)

7. Notifications
   - Alert on failure via email/Slack
   - Include build number and commit hash
```

---

## 19. README.md Template

```markdown
# DevOps Pipeline Dashboard

A full-stack MERN application with complete DevSecOps CI/CD pipeline on Microsoft Azure.

## Architecture

- **Frontend:** React.js (deployed on AKS)
- **Backend:** Node.js + Express (deployed on AKS)
- **Database:** MongoDB (deployed on AKS)
- **CI/CD:** Jenkins (on Azure VM)
- **Container Registry:** Azure Container Registry
- **Orchestration:** Azure Kubernetes Service
- **Security:** SonarQube + Trivy + OWASP ZAP
- **Monitoring:** Prometheus + Grafana

## Quick Start

### Prerequisites

- Azure CLI installed
- kubectl installed
- Docker installed
- Access to Azure subscription

### Clone Repository

git clone https://github.com/YOUR-USERNAME/devops-pipeline-dashboard.git
cd devops-pipeline-dashboard

### Local Development

npm install
npm start

### Deploy to Azure

1. Follow Azure Resource Group setup (Section 3)
2. Create VM (Section 4)
3. Create ACR (Section 5)
4. Create AKS (Section 6)
5. Configure Jenkins (Section 7)
6. Run migration steps (Section 11)

## Pipeline Stages

| Stage | Tool | Purpose |
|---|---|---|
| Checkout | Git | Pull source code |
| Test | Jest | Unit tests + coverage |
| SAST | SonarQube | Static code analysis |
| Quality Gate | SonarQube | Block on failures |
| Build | npm | Production build |
| Docker Build | Docker | Container image |
| CVE Scan | Trivy | Image vulnerability scan |
| Push | ACR | Store images |
| Deploy | kubectl | AKS deployment |
| DAST | OWASP ZAP | Dynamic security scan |

## Azure Resources

| Resource | Name | Purpose |
|---|---|---|
| Resource Group | rg-devops-pipeline-prod | Container for all resources |
| Virtual Machine | vm-jenkins-devops | Jenkins + DevSecOps tools |
| Container Registry | devopspipelineacr | Docker image storage |
| Kubernetes Service | aks-devops-cluster | Application hosting |

## Cost Estimate

Monthly: ~$70-80 (Azure for Students: $100 credit)

To reduce costs:
- Stop VM when not in use
- Scale AKS to 0 nodes when idle

## Security

- SonarQube SAST scan on every commit
- Trivy CVE scan on every Docker image
- OWASP ZAP DAST scan on every deployment
- All reports archived in Jenkins

## Monitoring

- Prometheus: Metrics collection (port 9090)
- Grafana: Dashboards and alerting (port 3000)

## License

MIT
```

---

## Appendix A: Quick Reference Commands

```bash
# ════════════════════════════════════════
# AZURE CLI QUICK REFERENCE
# ════════════════════════════════════════

# Login
az login --use-device-code

# List resources
az resource list \
  --resource-group rg-devops-pipeline-prod \
  --output table

# Start/Stop VM
az vm start --resource-group rg-devops-pipeline-prod --name vm-jenkins-devops
az vm deallocate --resource-group rg-devops-pipeline-prod --name vm-jenkins-devops

# ACR operations
az acr login --name devopspipelineacr
az acr repository list --name devopspipelineacr

# AKS operations
az aks get-credentials --resource-group rg-devops-pipeline-prod --name aks-devops-cluster
az aks show --resource-group rg-devops-pipeline-prod --name aks-devops-cluster

# Cost check
az consumption usage list --output table

# ════════════════════════════════════════
# KUBECTL QUICK REFERENCE
# ════════════════════════════════════════

# Namespace operations
kubectl get namespaces
kubectl create namespace devops-app

# Pod operations
kubectl get pods -n devops-app
kubectl describe pod <name> -n devops-app
kubectl logs <name> -n devops-app
kubectl exec -it <name> -n devops-app -- /bin/bash
kubectl delete pod <name> -n devops-app

# Deployment operations
kubectl get deployments -n devops-app
kubectl rollout status deployment/<name> -n devops-app
kubectl rollout restart deployment/<name> -n devops-app
kubectl scale deployment/<name> --replicas=3 -n devops-app

# Service operations
kubectl get services -n devops-app
kubectl describe service <name> -n devops-app
kubectl port-forward svc/<name> <local-port>:<svc-port> -n devops-app

# Apply manifests
kubectl apply -f k8s/ -n devops-app
kubectl delete -f k8s/ -n devops-app

# Resource monitoring
kubectl top pods -n devops-app
kubectl top nodes

# ════════════════════════════════════════
# DOCKER QUICK REFERENCE
# ════════════════════════════════════════

# Build
docker build -t <image-name>:<tag> -f Dockerfile .

# Tag
docker tag <local-image> <acr-url>/<image>:<tag>

# Push
docker push <acr-url>/<image>:<tag>

# Run
docker run -d --name <name> -p <host>:<container> <image>

# Logs
docker logs <name> --follow

# Shell
docker exec -it <name> /bin/bash

# Cleanup
docker system prune -af
docker volume prune -f
```

---

## Appendix B: Environment Variables Reference

```bash
# Create .env.example file (commit this, NOT .env)

# ─── Application ───────────────────────
NODE_ENV=production
PORT=5000
FRONTEND_PORT=3000

# ─── Database ──────────────────────────
MONGO_URI=mongodb://mongodb-service:27017/devops_db

# ─── Authentication ────────────────────
JWT_SECRET=your-jwt-secret-minimum-32-characters

# ─── Azure ─────────────────────────────
ACR_NAME=devopspipelineacr
ACR_URL=devopspipelineacr.azurecr.io
AKS_CLUSTER_NAME=aks-devops-cluster
RESOURCE_GROUP=rg-devops-pipeline-prod
AKS_REGION=eastus

# ─── SonarQube ─────────────────────────
SONAR_HOST_URL=http://localhost:9000
SONAR_PROJECT_KEY=devops-pipeline-dashboard

# ─── Monitoring ────────────────────────
PROMETHEUS_URL=http://prometheus:9090
GRAFANA_URL=http://grafana:3000
```

---

*Document Version: 1.0.0 | Last Updated: 2024 | Azure for Students Optimized*

*This guide covers migration of the Phase 2 local DevSecOps pipeline to Microsoft Azure, including all tools, configurations, verification steps, and cost optimization for the $100 Azure for Students credit.*
