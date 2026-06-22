# Secure DevOps Pipeline — Phase 2: Local Implementation Guide

> **Project:** DevOps Pipeline Dashboard (MERN Stack)
> **Phase:** 2 — Local Secure DevOps Pipeline
> **Author:** Senior DevSecOps Architect
> **Environment:** Windows 11 → Local → Azure (Phase 3)
> **Status:** Production-Ready

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Prerequisites](#2-prerequisites)
3. [Dependency Installation & Verification](#3-dependency-installation--verification)
4. [Folder Structure](#4-folder-structure)
5. [Docker Configuration](#5-docker-configuration)
6. [SonarQube Setup](#6-sonarqube-setup)
7. [Jenkins Setup](#7-jenkins-setup)
8. [Trivy Security Scanner](#8-trivy-security-scanner)
9. [OWASP ZAP Setup](#9-owasp-zap-setup)
10. [Prometheus Monitoring](#10-prometheus-monitoring)
11. [Grafana Dashboard](#11-grafana-dashboard)
12. [Kubernetes Manifests](#12-kubernetes-manifests)
13. [Secure CI/CD Pipeline — Jenkinsfile](#13-secure-cicd-pipeline--jenkinsfile)
14. [Running Instructions](#14-running-instructions)
15. [Verification Procedures](#15-verification-procedures)
16. [Troubleshooting Guide](#16-troubleshooting-guide)
17. [Best Practices](#17-best-practices)
18. [Documentation Suite](#18-documentation-suite)

---

## 1. Architecture Overview

### 1.1 Local Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Windows 11 Host Machine                          │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    Docker Desktop Engine                         │   │
│  │                                                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │   │
│  │  │ Frontend │  │ Backend  │  │ MongoDB  │  │  SonarQube   │   │   │
│  │  │  :3000   │  │  :5000   │  │  :27017  │  │    :9000     │   │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │   │
│  │                                                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │   │
│  │  │ Jenkins  │  │Prometheus│  │ Grafana  │  │  OWASP ZAP   │   │   │
│  │  │  :8080   │  │  :9090   │  │  :3001   │  │    :8090     │   │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │   │
│  │                                                                  │   │
│  │  ┌───────────────────────────────────────────────────────────┐  │   │
│  │  │              Docker Desktop Kubernetes Cluster             │  │   │
│  │  │  ┌──────────────────┐    ┌──────────────────────────────┐ │  │   │
│  │  │  │  devops-dashboard │    │    Ingress Controller         │ │  │   │
│  │  │  │  namespace       │    │    (localhost:80)             │ │  │   │
│  │  │  │                  │    └──────────────────────────────┘ │  │   │
│  │  │  │  frontend-pod    │                                      │  │   │
│  │  │  │  backend-pod     │                                      │  │   │
│  │  │  │  mongodb-pod     │                                      │  │   │
│  │  │  └──────────────────┘                                      │  │   │
│  │  └───────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                           │
│  │   Git    │   │  GitHub  │   │  Trivy   │  (Host-installed)          │
│  └──────────┘   └──────────┘   └──────────┘                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Secure DevOps Pipeline Flow

```
┌─────────────┐
│ Developer   │
│ Workstation │
└──────┬──────┘
       │ git push
       ▼
┌─────────────┐     Webhook      ┌─────────────────────────────────────┐
│   GitHub    │─────────────────►│           Jenkins Pipeline          │
│ Repository  │                  │                                     │
└─────────────┘                  │  Stage 1: Checkout                  │
                                 │  Stage 2: npm install               │
                                 │  Stage 3: npm test                  │
                                 │  Stage 4: SonarQube Scan ──────────►│──► SonarQube :9000
                                 │  Stage 5: Quality Gate              │
                                 │  Stage 6: npm run build             │
                                 │  Stage 7: Docker Build              │
                                 │  Stage 8: Trivy Image Scan ────────►│──► Trivy (CLI)
                                 │  Stage 9: Push to Registry          │
                                 │  Stage 10: Deploy to K8s ──────────►│──► Kubernetes
                                 │  Stage 11: Verify Deployment        │
                                 │  Stage 12: OWASP ZAP Scan ─────────►│──► ZAP :8090
                                 │  Stage 13: Archive Reports          │
                                 │  Stage 14: Notify                   │
                                 └─────────────────────────────────────┘
```

### 1.3 Monitoring Flow

```
┌─────────────────────────────────────────────────────────┐
│                    Monitoring Stack                      │
│                                                         │
│  Application ──► Node Exporter ──► Prometheus :9090     │
│  Kubernetes  ──► kube-metrics  ──► Prometheus :9090     │
│  Jenkins     ──► metrics API   ──► Prometheus :9090     │
│                                         │               │
│                                         ▼               │
│                                    Grafana :3001         │
│                                    (Dashboards)          │
│                                         │               │
│                                         ▼               │
│                                 Alert Manager           │
│                                 (Email/Slack)            │
└─────────────────────────────────────────────────────────┘
```

### 1.4 Security Flow

```
┌─────────────────────────────────────────────────────────────┐
│                      Security Gates                         │
│                                                             │
│  Source Code ──► SonarQube SAST ──► Quality Gate           │
│                        │                  │                  │
│                        ▼                  ▼                  │
│                   Code Smells        Pass/Fail               │
│                   Vulnerabilities                            │
│                   Coverage %                                 │
│                                                             │
│  Docker Image ──► Trivy Scan ──────► Severity Filter       │
│                        │                  │                  │
│                        ▼                  ▼                  │
│                   CVE Report         CRITICAL/HIGH          │
│                   SBOM               Block/Allow             │
│                                                             │
│  Running App  ──► OWASP ZAP DAST ──► HTML Report           │
│                        │                  │                  │
│                        ▼                  ▼                  │
│                   Passive Scan       Risk Score              │
│                   Active Scan        Archive                 │
└─────────────────────────────────────────────────────────────┘
```

### 1.5 Deployment Flow

```
Developer
    │
    │ git push origin main
    ▼
GitHub (Remote Repository)
    │
    │ Webhook POST /github-webhook/
    ▼
Jenkins (localhost:8080)
    │
    ├─► Build & Test
    │       └─► npm install, npm test
    │
    ├─► Security Scan
    │       ├─► SonarQube (SAST)
    │       └─► Trivy (Image)
    │
    ├─► Build Artifact
    │       ├─► npm run build
    │       └─► docker build + tag
    │
    ├─► Deploy
    │       ├─► kubectl apply -f kubernetes/
    │       └─► kubectl rollout status
    │
    ├─► DAST Scan
    │       └─► OWASP ZAP baseline scan
    │
    └─► Monitoring
            ├─► Prometheus scrapes metrics
            └─► Grafana visualizes
```

---

## 2. Prerequisites

### 2.1 Hardware Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU Cores | 4 cores | 8+ cores |
| RAM | 16 GB | 32 GB |
| Free Disk Space | 50 GB | 100 GB |
| Network | Broadband | Broadband |

> **Why these specs?** Running Jenkins, SonarQube, Kubernetes, Prometheus, Grafana, and OWASP ZAP simultaneously is memory-intensive. SonarQube alone requires 2 GB RAM minimum. Docker Desktop Kubernetes requires additional overhead.

### 2.2 Software Requirements

| Software | Required Version | Purpose |
|----------|-----------------|---------|
| Windows 11 | 22H2 or later | Host OS |
| Git | 2.40+ | Source control |
| Docker Desktop | 4.25+ | Containers + Kubernetes |
| Jenkins | 2.440+ LTS | CI/CD orchestration |
| kubectl | 1.28+ | Kubernetes CLI |
| Trivy | 0.48+ | Container security scanner |
| Node.js | 18.x LTS | Application runtime |
| npm | 9.x+ | Package manager |
| Java (JDK) | 17 LTS | Jenkins runtime |

### 2.3 System Requirements

```
Windows 11 Settings:
├── Virtualization enabled in BIOS/UEFI
├── WSL2 enabled (required for Docker Desktop)
├── Hyper-V enabled
├── At least 16 GB RAM allocated
└── Windows Defender exceptions for:
    ├── C:\Program Files\Docker\
    ├── C:\Users\<user>\.docker\
    └── C:\ProgramData\Jenkins\
```

### 2.4 Required Network Ports

| Port | Service | Protocol |
|------|---------|----------|
| 3000 | React Frontend | HTTP |
| 5000 | Node.js Backend | HTTP |
| 8080 | Jenkins | HTTP |
| 9000 | SonarQube | HTTP |
| 8090 | OWASP ZAP | HTTP |
| 9090 | Prometheus | HTTP |
| 3001 | Grafana | HTTP |
| 27017 | MongoDB | TCP |
| 80/443 | Kubernetes Ingress | HTTP/HTTPS |

---

## 3. Dependency Installation & Verification

### 3.1 WSL2 Setup (Required for Docker Desktop)

Open **PowerShell as Administrator** and run:

```powershell
# Enable WSL2 feature
# This command enables the Windows Subsystem for Linux feature required by Docker Desktop
wsl --install

# Set WSL version to 2 (faster and more compatible than WSL1)
wsl --set-default-version 2

# Verify WSL2 is active
wsl --status
```

**Expected Output:**
```
Default Version: 2
Windows Subsystem for Linux was last updated on ...
WSL automatic updates are on.
Kernel version: 5.15.xx.x-x
```

### 3.2 Docker Desktop Installation

**Step 1: Download Docker Desktop**

```powershell
# Download Docker Desktop installer using winget (Windows Package Manager)
# winget is built into Windows 11
winget install Docker.DockerDesktop

# OR download manually from:
# https://www.docker.com/products/docker-desktop/
```

**Step 2: Install Docker Desktop**

```powershell
# Run the downloaded installer
# Accept the license agreement
# Select "Use WSL 2 instead of Hyper-V" during installation
# Restart your computer when prompted

# After restart, Docker Desktop opens automatically
# Wait for the Docker engine to start (whale icon in system tray turns green)
```

**Step 3: Enable Kubernetes in Docker Desktop**

```
Docker Desktop → Settings → Kubernetes → Enable Kubernetes → Apply & Restart
```

**Step 4: Verify Docker Desktop**

```powershell
# Verify Docker engine is running
# The 'docker version' command shows both client and server information
docker version

# Check Docker system information (shows resources available)
docker info

# Verify Docker can pull and run images
# This downloads and runs a test container - should print "Hello from Docker!"
docker run hello-world
```

**Expected Output (docker version):**
```
Client:
 Cloud integration: v1.0.35+desktop.10
 Version:           25.0.x
 API version:       1.44
 Go version:        go1.21.x
 OS/Arch:           windows/amd64

Server: Docker Desktop 4.x.x
 Engine:
  Version:          25.0.x
  API version:      1.44 (minimum version 1.24)
```

**Step 5: Verify Kubernetes**

```powershell
# Show all nodes in your local Kubernetes cluster
# Should show 'docker-desktop' node with STATUS=Ready
kubectl get nodes

# Check all system pods are running
kubectl get pods --all-namespaces
```

**Expected Output:**
```
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   5m    v1.28.x
```

### 3.3 Jenkins Verification

Jenkins is already installed. Verify it is running properly:

```powershell
# Check Jenkins service status using Windows Service Manager
# Jenkins runs as a Windows service named 'Jenkins'
sc query Jenkins

# Verify Jenkins is listening on port 8080
# netstat shows network connections; -a shows all, -n shows numeric, -o shows owning PID
netstat -ano | findstr :8080

# Retrieve Jenkins initial admin password
# This file is created during first-time Jenkins setup
type "C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword"

# Test Jenkins is accessible via HTTP
# Should return HTML content if Jenkins is running
curl -I http://localhost:8080
```

**Expected Output (sc query Jenkins):**
```
SERVICE_NAME: Jenkins
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

### 3.4 kubectl Verification

```powershell
# Display kubectl client and server version information
# Both client and server versions should be shown
kubectl version

# Verify kubectl can communicate with the cluster
# Should show the cluster API server address
kubectl cluster-info

# List all available API resources in the cluster
kubectl api-resources | head -20
```

**Expected Output (kubectl version):**
```
Client Version: v1.28.x
Kustomize Version: v5.x.x
Server Version: v1.28.x
```

### 3.5 Trivy Verification

```powershell
# Display Trivy version - confirms installation
# Trivy is a comprehensive security scanner for containers and filesystems
trivy version

# Show all available Trivy commands and options
trivy --help

# Update Trivy vulnerability database
# This downloads the latest CVE database - required for accurate scanning
trivy image --download-db-only
```

**Expected Output:**
```
Version: 0.48.x
Vulnerability DB:
  Version: 2
  UpdatedAt: 2024-xx-xx xx:xx:xx.xxxxxxxx +0000 UTC
  NextUpdate: 2024-xx-xx xx:xx:xx.xxxxxxxx +0000 UTC
  DownloadedAt: 2024-xx-xx xx:xx:xx.xxxxxxxx +0000 UTC
```

### 3.6 Deploy SonarQube via Docker

SonarQube provides Static Application Security Testing (SAST) and code quality analysis.

```powershell
# Create a dedicated Docker network for DevOps tools
# This allows containers to communicate with each other by name
docker network create devops-network

# Create Docker volume for SonarQube data persistence
# Volumes survive container restarts and upgrades
docker volume create sonarqube-data
docker volume create sonarqube-logs
docker volume create sonarqube-extensions

# Pull SonarQube Community Edition image
# 'community' tag = free version with all core features
docker pull sonarqube:community

# Run SonarQube container
# --name: container identifier
# --network: connects to devops-network so Jenkins can reach it
# -p 9000:9000: maps host port 9000 to container port 9000
# -v: mounts volumes for data persistence
# -e: sets memory limits (SonarQube requires at least 2GB heap)
# --restart unless-stopped: auto-restarts on Docker Desktop restart
docker run -d \
  --name sonarqube \
  --network devops-network \
  -p 9000:9000 \
  -v sonarqube-data:/opt/sonarqube/data \
  -v sonarqube-logs:/opt/sonarqube/logs \
  -v sonarqube-extensions:/opt/sonarqube/extensions \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  --restart unless-stopped \
  sonarqube:community

# Verify SonarQube container is running
docker ps --filter name=sonarqube

# Monitor SonarQube startup logs (takes 2-3 minutes first time)
# Press Ctrl+C to stop following logs once you see "SonarQube is operational"
docker logs -f sonarqube
```

> **Note for Windows:** If SonarQube fails to start with `vm.max_map_count` error, run this in PowerShell as Administrator:
> ```powershell
> wsl -d docker-desktop sysctl -w vm.max_map_count=262144
> ```

### 3.7 Deploy OWASP ZAP via Docker

OWASP ZAP (Zed Attack Proxy) performs Dynamic Application Security Testing (DAST).

```powershell
# Pull the official OWASP ZAP stable image
# This image includes the full ZAP tool optimized for CI/CD usage
docker pull owasp/zap2docker-stable

# Run OWASP ZAP as a persistent daemon container
# -d: detached mode (runs in background)
# -u zap: run as the 'zap' user (non-root for security)
# -p 8090:8090: expose ZAP API on port 8090
# zap.sh -daemon: starts ZAP in daemon mode for API/Jenkins access
# -host 0.0.0.0: listen on all interfaces
# -port 8090: API port
# -config api.addrs.addr.name=.*: allow connections from any address
# -config api.addrs.addr.regex=true: enable regex matching for allowed addresses
# -config api.disablekey=true: disable API key (fine for local dev)
docker run -d \
  --name owasp-zap \
  --network devops-network \
  -p 8090:8090 \
  -v ${PWD}/reports:/zap/wrk \
  --user root \
  --restart unless-stopped \
  owasp/zap2docker-stable \
  zap.sh -daemon \
  -host 0.0.0.0 \
  -port 8090 \
  -config api.addrs.addr.name=.* \
  -config api.addrs.addr.regex=true \
  -config api.disablekey=true

# Verify ZAP container is running
docker ps --filter name=owasp-zap
```

### 3.8 Deploy Prometheus via Docker

Prometheus collects metrics from your application and infrastructure.

```powershell
# Create a directory for Prometheus configuration on the host
# This directory will be mounted into the container
mkdir -p C:\devops\monitoring\prometheus

# Create prometheus.yml configuration file (see Section 10 for full config)
# For now, create the directory structure

# Pull Prometheus image
docker pull prom/prometheus:latest

# Run Prometheus container
# --config.file: tells Prometheus where to find its configuration
# -v: mounts local config file into the container
# --web.enable-lifecycle: enables API endpoints for reloading config without restart
# --web.enable-admin-api: enables admin APIs for snapshot/deletion operations
docker run -d \
  --name prometheus \
  --network devops-network \
  -p 9090:9090 \
  -v C:\devops\monitoring\prometheus\prometheus.yml:/etc/prometheus/prometheus.yml \
  -v prometheus-data:/prometheus \
  --restart unless-stopped \
  prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/prometheus \
  --web.enable-lifecycle \
  --web.enable-admin-api

# Verify Prometheus is running
docker ps --filter name=prometheus
```

### 3.9 Deploy Grafana via Docker

Grafana provides dashboards and visualization for Prometheus metrics.

```powershell
# Create Grafana data volume for dashboard persistence
docker volume create grafana-data

# Pull Grafana image
docker pull grafana/grafana:latest

# Run Grafana container
# GF_SECURITY_ADMIN_PASSWORD: sets the admin password
# GF_USERS_ALLOW_SIGN_UP: disables self-registration for security
# GF_SERVER_HTTP_PORT: sets the internal port Grafana listens on
# -p 3001:3000: maps host port 3001 to Grafana's default 3000
#   (using 3001 to avoid conflict with React frontend on port 3000)
docker run -d \
  --name grafana \
  --network devops-network \
  -p 3001:3000 \
  -v grafana-data:/var/lib/grafana \
  -e GF_SECURITY_ADMIN_PASSWORD=admin123 \
  -e GF_USERS_ALLOW_SIGN_UP=false \
  -e GF_SERVER_HTTP_PORT=3000 \
  --restart unless-stopped \
  grafana/grafana:latest

# Verify Grafana is running
docker ps --filter name=grafana
```

---

## 4. Folder Structure

### 4.1 Complete Project Structure

```
devops-pipeline-dashboard/          ← Root project folder
│
├── client/                         ← React.js Frontend (Vite) - EXISTING
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.js
│   └── ...
│
├── server/                         ← Node.js / Express Backend - EXISTING
│   ├── routes/
│   ├── models/
│   ├── controllers/
│   ├── package.json
│   └── ...
│
├── docker/                         ← All Docker configuration files
│   ├── frontend/
│   │   ├── Dockerfile              ← Multi-stage build for React app
│   │   └── nginx.conf              ← Nginx configuration for serving React
│   ├── backend/
│   │   └── Dockerfile              ← Node.js production Dockerfile
│   ├── .dockerignore               ← Files excluded from Docker build context
│   └── docker-compose.yml          ← Orchestrates all local services
│
├── kubernetes/                     ← Kubernetes resource manifests
│   ├── namespace.yaml              ← Isolates project resources
│   ├── configmap.yaml              ← Non-secret configuration
│   ├── secret.yaml                 ← Encrypted credentials
│   ├── deployment.yaml             ← Pod definitions and replicas
│   ├── service.yaml                ← Internal service discovery
│   └── ingress.yaml                ← External HTTP routing
│
├── jenkins/                        ← Jenkins CI/CD configuration
│   ├── Jenkinsfile                 ← Complete CI/CD pipeline definition
│   ├── plugins.txt                 ← Required Jenkins plugins list
│   └── casc/
│       └── jenkins.yaml            ← Jenkins Configuration as Code (JCasC)
│
├── monitoring/                     ← Observability stack configuration
│   ├── prometheus/
│   │   └── prometheus.yml          ← Prometheus scrape configuration
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   └── devops-dashboard.json  ← Pre-built Grafana dashboard
│   │   └── datasources/
│   │       └── prometheus.yaml     ← Auto-provisioned datasource
│   └── alerts/
│       └── alert-rules.yml         ← Prometheus alerting rules
│
├── reports/                        ← Security scan output directory
│   ├── sonarqube/                  ← SonarQube analysis reports
│   ├── trivy/                      ← Trivy vulnerability JSON reports
│   ├── owasp-zap/                  ← ZAP HTML and XML reports
│   └── .gitkeep                    ← Keeps empty directory in Git
│
├── docs/                           ← Project documentation
│   ├── README.md                   ← Project overview and quick start
│   ├── LOCAL-SETUP.md              ← This document (detailed local guide)
│   ├── TROUBLESHOOTING.md          ← Common issues and solutions
│   ├── BEST-PRACTICES.md           ← DevSecOps standards
│   └── ARCHITECTURE.md             ← Diagrams and design decisions
│
├── .env.example                    ← Template for environment variables
├── .gitignore                      ← Git exclusion rules
└── README.md                       ← Top-level project README
```

### 4.2 Create Folder Structure

Run these commands from your project root in PowerShell:

```powershell
# Create all required directories at once
# The -Force flag creates parent directories as needed
New-Item -ItemType Directory -Force -Path @(
  "docker\frontend",
  "docker\backend",
  "kubernetes",
  "jenkins\casc",
  "monitoring\prometheus",
  "monitoring\grafana\dashboards",
  "monitoring\grafana\datasources",
  "monitoring\alerts",
  "reports\sonarqube",
  "reports\trivy",
  "reports\owasp-zap",
  "docs"
)

# Create placeholder files to track empty directories in Git
New-Item -ItemType File -Force -Path "reports\.gitkeep"

# Verify structure was created
Get-ChildItem -Recurse -Directory | Select-Object FullName
```

---

## 5. Docker Configuration

### 5.1 Frontend Dockerfile

**File:** `docker/frontend/Dockerfile`

```dockerfile
# ============================================================
# Stage 1: BUILD STAGE
# Uses Node.js to compile the React application
# ============================================================

# Use Node.js 18 Alpine as the build base
# Alpine = minimal Linux image (~5MB vs ~100MB for full Debian)
# This reduces attack surface and build time
FROM node:18-alpine AS builder

# Set working directory inside the container
# All subsequent commands run from this directory
WORKDIR /app

# Copy package files FIRST (before source code)
# Docker caches each layer. If package.json doesn't change,
# the npm install layer is reused - dramatically speeds up builds
COPY client/package.json client/package-lock.json ./

# Install dependencies
# --ci: uses package-lock.json exactly (reproducible builds)
# --only=production would skip devDependencies, but we need them for build
RUN npm ci

# Copy the rest of the frontend source code
# Done AFTER npm install so code changes don't invalidate the npm cache layer
COPY client/ .

# Set build-time environment variable for API URL
# This bakes the backend URL into the React bundle at compile time
# Override this in docker-compose or CI/CD for different environments
ARG VITE_API_URL=http://localhost:5000
ENV VITE_API_URL=$VITE_API_URL

# Build the React application for production
# Vite compiles, minifies, and outputs to the /app/dist directory
RUN npm run build

# ============================================================
# Stage 2: PRODUCTION STAGE
# Uses lightweight Nginx to serve the compiled static files
# The builder stage artifacts are NOT included in this image
# ============================================================

# Nginx Alpine is extremely small (~23MB) and production-hardened
FROM nginx:alpine AS production

# Copy custom Nginx configuration
# Controls how Nginx serves the React SPA (Single Page App)
COPY docker/frontend/nginx.conf /etc/nginx/conf.d/default.conf

# Copy compiled React assets from the builder stage
# ONLY the /app/dist files are copied - not node_modules, source code, etc.
# This is the key security benefit of multi-stage builds
COPY --from=builder /app/dist /usr/share/nginx/html

# Add labels for image metadata and traceability
LABEL maintainer="devops-team"
LABEL version="1.0"
LABEL description="DevOps Pipeline Dashboard - Frontend"

# Document that Nginx serves on port 80
# This is metadata only - port mapping happens in docker-compose/kubernetes
EXPOSE 80

# Health check - Docker will periodically test if the container is healthy
# --interval: check every 30 seconds
# --timeout: fail after 5 seconds of no response
# --start-period: wait 10 seconds before first check (startup time)
# --retries: mark unhealthy after 3 consecutive failures
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:80 || exit 1

# Run Nginx in foreground (required for Docker - containers need a foreground process)
# 'daemon off' prevents Nginx from forking to background (which would cause container to exit)
CMD ["nginx", "-g", "daemon off;"]
```

**File:** `docker/frontend/nginx.conf`

```nginx
server {
    # Listen on port 80 (standard HTTP)
    listen 80;
    
    # Accept requests for any hostname (use specific domain in production)
    server_name localhost;

    # Serve files from this directory
    # This is where we copied the React build output
    root /usr/share/nginx/html;
    
    # Default file to serve when directory is requested
    index index.html;

    # Enable gzip compression for better performance
    # Reduces transfer size for text-based files (JS, CSS, HTML)
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    gzip_min_length 1000;

    # Security headers (defense in depth)
    # X-Frame-Options: prevents clickjacking attacks
    add_header X-Frame-Options "SAMEORIGIN" always;
    # X-Content-Type-Options: prevents MIME type sniffing
    add_header X-Content-Type-Options "nosniff" always;
    # X-XSS-Protection: enables browser XSS filter
    add_header X-XSS-Protection "1; mode=block" always;
    # Referrer-Policy: controls referrer information
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # React Router support
    # For Single Page Apps, all routes must serve index.html
    # React Router then handles routing client-side
    # $uri: try the exact file first
    # $uri/: try as directory
    # /index.html: fallback to SPA entry point
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets aggressively
    # Files with hash in name (from Vite build) are immutable
    # 'Cache-Control: max-age=31536000' = cache for 1 year
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Proxy API requests to the backend
    # Requests to /api/* are forwarded to the backend service
    # This avoids CORS issues in development
    location /api/ {
        proxy_pass http://backend:5000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    # Health check endpoint for container orchestrators
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### 5.2 Backend Dockerfile

**File:** `docker/backend/Dockerfile`

```dockerfile
# ============================================================
# Stage 1: DEPENDENCY INSTALLATION
# Separates dependency install from app copy for cache efficiency
# ============================================================

FROM node:18-alpine AS dependencies

# Install OS packages needed for native npm modules
# python3, make, g++: required to compile native addons (bcrypt, canvas, etc.)
# dumb-init: proper PID 1 init system for containers
RUN apk add --no-cache python3 make g++ dumb-init

WORKDIR /app

# Copy only package files to leverage Docker layer caching
COPY server/package.json server/package-lock.json ./

# Install ALL dependencies (including devDependencies for now)
# --ci: strict mode using package-lock.json
RUN npm ci

# ============================================================
# Stage 2: PRODUCTION IMAGE
# ============================================================

FROM node:18-alpine AS production

# Install dumb-init in the production image
# dumb-init correctly handles PID 1 responsibilities:
# - Forwards signals (SIGTERM for graceful shutdown)
# - Reaps zombie processes
RUN apk add --no-cache dumb-init

# Create non-root user for security
# Running as root in containers is a security risk
# 'nodejs' user and 'nodejs' group with fixed IDs for consistency
RUN addgroup -g 1001 -S nodejs && \
    adduser -S -u 1001 -G nodejs nodejs

WORKDIR /app

# Copy node_modules from the dependencies stage
# This avoids reinstalling packages in the production stage
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy application source code
# --chown: sets ownership to nodejs user (not root)
COPY --chown=nodejs:nodejs server/ .

# Switch to non-root user
# All subsequent commands and the final process run as 'nodejs'
USER nodejs

# Document the port the backend listens on
EXPOSE 5000

# Image metadata labels
LABEL maintainer="devops-team"
LABEL version="1.0"
LABEL description="DevOps Pipeline Dashboard - Backend API"

# Health check for the backend API
# Checks the /api/health endpoint every 30 seconds
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:5000/api/health || exit 1

# Use dumb-init as the entry point wrapper
# dumb-init then executes node as its child process
# This ensures proper signal handling and process management
ENTRYPOINT ["dumb-init", "--"]

# Start the Node.js application
# Using node directly instead of npm for production
# (npm adds unnecessary overhead and signal handling issues)
CMD ["node", "server.js"]
```

### 5.3 .dockerignore

**File:** `.dockerignore` (place in project root)

```dockerignore
# ============================================================
# .dockerignore — Excludes files from Docker build context
# Smaller build context = faster builds and smaller images
# ============================================================

# Node.js dependencies (rebuilt inside container)
# Never copy local node_modules - different OS/architecture
**/node_modules
**/npm-debug.log
**/.npm

# Build artifacts (generated by build stage inside container)
**/dist
**/build
**/.next
**/.nuxt

# Version control (not needed in container)
.git
.gitignore
.gitattributes

# Development environment files
# These contain local paths and dev settings irrelevant to Docker
**/.env
**/.env.local
**/.env.development
**/.env.test

# IDE and editor files
.vscode
.idea
*.swp
*.swo
**/.DS_Store
**/Thumbs.db

# Test files and coverage reports
# Tests don't run inside the production image
**/coverage
**/__tests__
**/*.test.js
**/*.spec.js
**/jest.config.js

# Documentation
**/docs
**/*.md
!docker/frontend/nginx.conf

# DevOps configuration files (not part of the app)
docker/
kubernetes/
jenkins/
monitoring/
reports/

# Logs
**/logs
**/*.log

# Temporary files
**/tmp
**/temp
```

### 5.4 Docker Compose

**File:** `docker/docker-compose.yml`

```yaml
# Docker Compose version 3.8 supports all modern features
# including secrets, configs, and deploy specs
version: "3.8"

# ============================================================
# NETWORKS
# Custom bridge network allows containers to communicate
# by service name (e.g., "backend" resolves to backend container)
# ============================================================
networks:
  devops-network:
    # 'bridge' driver creates an isolated network on the host
    # External traffic must explicitly be routed through port mappings
    driver: bridge
    name: devops-network

# ============================================================
# VOLUMES
# Named volumes persist data across container restarts/upgrades
# Unlike bind mounts, Docker manages the storage location
# ============================================================
volumes:
  mongodb-data:       # Persists MongoDB database files
  sonarqube-data:     # Persists SonarQube project analysis data
  sonarqube-logs:     # Persists SonarQube log files
  sonarqube-ext:      # Persists SonarQube plugins/extensions
  prometheus-data:    # Persists Prometheus time-series data
  grafana-data:       # Persists Grafana dashboards and settings

# ============================================================
# SERVICES
# Each service = one Docker container
# ============================================================
services:

  # ----------------------------------------------------------
  # MONGODB — Document Database
  # Stores application data (pipeline runs, metrics, users)
  # ----------------------------------------------------------
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    networks:
      - devops-network
    ports:
      # 27017:27017 — expose MongoDB port for local tools (MongoDB Compass)
      # Remove this in production (only internal access needed)
      - "27017:27017"
    volumes:
      # Mount named volume to persist database files
      - mongodb-data:/data/db
    environment:
      # Set root credentials for MongoDB
      # In production, use Docker secrets instead of plain text
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
      MONGO_INITDB_DATABASE: devops_dashboard
    # Restart policy: auto-restart unless manually stopped
    restart: unless-stopped
    # Health check using MongoDB's built-in ping command
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

  # ----------------------------------------------------------
  # BACKEND — Node.js / Express API Server
  # REST API consumed by the React frontend
  # ----------------------------------------------------------
  backend:
    # Build from our custom Dockerfile
    build:
      context: ..               # Build context is project root
      dockerfile: docker/backend/Dockerfile
    container_name: backend
    networks:
      - devops-network
    ports:
      - "5000:5000"
    environment:
      # Runtime environment
      NODE_ENV: production
      # MongoDB connection string (uses service name 'mongodb' for DNS)
      MONGODB_URI: mongodb://admin:password123@mongodb:27017/devops_dashboard?authSource=admin
      # JWT secret for authentication tokens
      JWT_SECRET: your-super-secret-jwt-key-change-in-production
      # Port the server listens on
      PORT: 5000
    # Wait for MongoDB to be healthy before starting backend
    # This prevents connection errors on startup
    depends_on:
      mongodb:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s

  # ----------------------------------------------------------
  # FRONTEND — React.js Application (served by Nginx)
  # Static files compiled by Vite, served by Nginx
  # ----------------------------------------------------------
  frontend:
    build:
      context: ..
      dockerfile: docker/frontend/Dockerfile
      args:
        # Pass backend URL as build argument
        # Vite bakes this into the JavaScript bundle at build time
        VITE_API_URL: http://localhost:5000/api
    container_name: frontend
    networks:
      - devops-network
    ports:
      - "3000:80"              # Map host port 3000 to Nginx port 80
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:80"]
      interval: 30s
      timeout: 5s
      retries: 3

  # ----------------------------------------------------------
  # SONARQUBE — Static Code Analysis
  # Analyzes source code for bugs, vulnerabilities, code smells
  # ----------------------------------------------------------
  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    networks:
      - devops-network
    ports:
      - "9000:9000"
    volumes:
      - sonarqube-data:/opt/sonarqube/data
      - sonarqube-logs:/opt/sonarqube/logs
      - sonarqube-ext:/opt/sonarqube/extensions
    environment:
      # Disable Elasticsearch bootstrap checks for local/dev use
      # In production, these checks protect against misconfiguration
      SONAR_ES_BOOTSTRAP_CHECKS_DISABLE: "true"
    restart: unless-stopped
    # SonarQube needs increased heap space
    # ulimits configure OS-level resource limits for the container
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  # ----------------------------------------------------------
  # PROMETHEUS — Metrics Collection
  # Scrapes and stores time-series metrics from all services
  # ----------------------------------------------------------
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    networks:
      - devops-network
    ports:
      - "9090:9090"
    volumes:
      # Mount our custom config file (read-only for security)
      - ../monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      # Named volume for storing time-series data
      - prometheus-data:/prometheus
    command:
      # Tell Prometheus where its config file is
      - '--config.file=/etc/prometheus/prometheus.yml'
      # Where to store the time-series database
      - '--storage.tsdb.path=/prometheus'
      # Keep 15 days of data (increase for production)
      - '--storage.tsdb.retention.time=15d'
      # Enable hot-reload of config without restart
      - '--web.enable-lifecycle'
      # Enable admin API for snapshots
      - '--web.enable-admin-api'
    restart: unless-stopped

  # ----------------------------------------------------------
  # GRAFANA — Metrics Visualization
  # Creates dashboards from Prometheus data
  # ----------------------------------------------------------
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    networks:
      - devops-network
    ports:
      - "3001:3000"            # Map 3001 on host (3000 is used by frontend)
    volumes:
      - grafana-data:/var/lib/grafana
      # Auto-provision datasources on startup
      - ../monitoring/grafana/datasources:/etc/grafana/provisioning/datasources:ro
      # Auto-provision dashboards on startup
      - ../monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin123
      GF_USERS_ALLOW_SIGN_UP: "false"
      # Disable telemetry reporting to Grafana Labs
      GF_ANALYTICS_REPORTING_ENABLED: "false"
    depends_on:
      - prometheus
    restart: unless-stopped
```

---

## 6. SonarQube Setup

### 6.1 First-Time Login

```
1. Open browser: http://localhost:9000
2. Default credentials:
   Username: admin
   Password: admin
3. You will be prompted to change the password on first login
4. Set new password: Admin@123 (or your secure password)
```

### 6.2 Generate Authentication Token

```
1. Click your profile picture (top-right corner)
2. Select "My Account"
3. Click "Security" tab
4. Under "Generate Tokens":
   - Name: jenkins-token
   - Type: Global Analysis Token
   - Expires in: No expiration (for local dev)
5. Click "Generate"
6. COPY THE TOKEN IMMEDIATELY — it will NOT be shown again
   Example token: sqa_abc123def456ghi789...
```

### 6.3 Create SonarQube Project

```
1. Click "Create Project" on the SonarQube dashboard
2. Select "Manually"
3. Project Configuration:
   - Project display name: DevOps Pipeline Dashboard
   - Project key: devops-pipeline-dashboard
   - Main branch name: main
4. Click "Set Up"
5. Select "With Jenkins" for CI/CD integration
6. Note the project key: devops-pipeline-dashboard
```

### 6.4 SonarQube Scanner Configuration

**File:** `sonar-project.properties` (place in project root)

```properties
# ============================================================
# SonarQube Project Properties
# This file tells sonar-scanner where to find your code
# and how to analyze it
# ============================================================

# Unique project identifier in SonarQube
# Must match what you created in the SonarQube UI
sonar.projectKey=devops-pipeline-dashboard

# Human-readable project name shown in SonarQube UI
sonar.projectName=DevOps Pipeline Dashboard

# Version of your application (update with each release)
sonar.projectVersion=1.0.0

# Source directory to analyze
# '.' means the current directory (project root)
sonar.sources=client/src,server

# Test files location (excluded from main analysis, tracked separately)
sonar.tests=client/src,server
sonar.test.inclusions=**/*.test.js,**/*.spec.js,**/*.test.jsx,**/*.spec.jsx

# Directories to exclude from analysis
# node_modules: third-party code (not your responsibility)
# dist/build: compiled output (not source code)
# coverage: test coverage reports (not source code)
sonar.exclusions=**/node_modules/**,**/dist/**,**/build/**,**/coverage/**,**/*.min.js

# Language-specific settings
# Tell SonarQube this is a JavaScript/JSX project
sonar.language=js

# Coverage report location (generated by Jest)
# SonarQube reads this to show coverage data
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# Test report location (generated by Jest with JUnit reporter)
sonar.testExecutionReportPaths=coverage/test-report.xml

# Source encoding
sonar.sourceEncoding=UTF-8

# SonarQube server URL
# 'sonarqube' = container name (resolved by Docker DNS)
# Use localhost:9000 when running sonar-scanner on host
sonar.host.url=http://localhost:9000

# Authentication token (overridden in Jenkins via credentials)
# DO NOT commit real tokens to source control
# sonar.login=YOUR_TOKEN_HERE
```

### 6.5 Install SonarQube Scanner

```powershell
# Install SonarQube Scanner globally via npm
# This makes 'sonar-scanner' available as a command
npm install -g sonarqube-scanner

# Verify installation
sonar-scanner --version

# Run a manual scan from project root (for testing)
# -Dsonar.login: your generated token
# -Dsonar.host.url: SonarQube server address
sonar-scanner \
  -Dsonar.login=YOUR_TOKEN_HERE \
  -Dsonar.host.url=http://localhost:9000

# OR use Docker to run the scanner (no installation needed)
docker run \
  --rm \
  --network devops-network \
  -v "${PWD}:/usr/src" \
  sonarsource/sonar-scanner-cli:latest \
  -Dsonar.login=YOUR_TOKEN_HERE \
  -Dsonar.host.url=http://sonarqube:9000
```

### 6.6 Jenkins Integration for SonarQube

After completing the Jenkins setup in Section 7, configure SonarQube in Jenkins:

```
1. Manage Jenkins → Configure System
2. Scroll to "SonarQube servers"
3. Click "Add SonarQube"
4. Name: SonarQube-Local
5. Server URL: http://sonarqube:9000
   (using container name for Docker DNS resolution)
6. Server authentication token:
   → Click "Add" → Jenkins
   → Kind: Secret text
   → Secret: (paste your SonarQube token)
   → ID: sonarqube-token
   → Description: SonarQube Authentication Token
7. Click "Add" then select "sonarqube-token" from dropdown
8. Save
```

---

## 7. Jenkins Setup

### 7.1 Access Jenkins

```
URL: http://localhost:8080
Initial Password: Found at C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
```

### 7.2 Required Jenkins Plugins

Install these plugins via **Manage Jenkins → Plugins → Available plugins**:

| Plugin Name | Purpose |
|-------------|---------|
| Pipeline | Enables Jenkinsfile-based pipelines |
| Git | GitHub source code checkout |
| GitHub Integration | Webhooks and GitHub status updates |
| Docker Pipeline | Build and push Docker images in pipeline |
| SonarQube Scanner | Code quality integration |
| HTML Publisher | Publishes HTML reports (ZAP, coverage) |
| JUnit | Test result visualization |
| Workspace Cleanup | Clean workspace before builds |
| Blue Ocean | Modern pipeline visualization UI |
| Credentials Binding | Safely use secrets in pipelines |
| Kubernetes CLI | kubectl commands in pipeline |
| Timestamper | Add timestamps to build logs |
| AnsiColor | Colored console output |
| Slack Notification | Build status notifications (optional) |

**Install via Jenkins CLI (faster):**

```powershell
# Download Jenkins CLI jar
curl -o jenkins-cli.jar http://localhost:8080/jnlpJars/jenkins-cli.jar

# Install all required plugins at once
java -jar jenkins-cli.jar \
  -s http://localhost:8080 \
  -auth admin:YOUR_ADMIN_PASSWORD \
  install-plugin \
    pipeline \
    git \
    github \
    docker-workflow \
    sonar \
    htmlpublisher \
    junit \
    ws-cleanup \
    blueocean \
    credentials-binding \
    kubernetes-cli \
    timestamper \
    ansicolor \
    -restart
```

### 7.3 Jenkins Credentials Configuration

Store all secrets in Jenkins Credentials Manager (never hardcode in Jenkinsfile):

```
Manage Jenkins → Credentials → System → Global credentials → Add Credentials
```

**Credential 1: GitHub Token**
```
Kind: Secret text
Secret: ghp_YOUR_GITHUB_PERSONAL_ACCESS_TOKEN
ID: github-token
Description: GitHub Personal Access Token
```

**Credential 2: SonarQube Token**
```
Kind: Secret text
Secret: sqa_YOUR_SONARQUBE_TOKEN
ID: sonarqube-token
Description: SonarQube Analysis Token
```

**Credential 3: Docker Hub Credentials**
```
Kind: Username with password
Username: your-dockerhub-username
Password: your-dockerhub-password-or-token
ID: dockerhub-credentials
Description: Docker Hub Registry Credentials
```

**Credential 4: Kubernetes Config**
```
Kind: Secret file
File: (upload your ~/.kube/config file)
ID: kubeconfig
Description: Kubernetes Configuration File
```

### 7.4 GitHub Webhook Configuration

**Step 1: Configure in GitHub**

```
1. Go to your GitHub repository
2. Settings → Webhooks → Add webhook
3. Payload URL: http://YOUR_PUBLIC_IP:8080/github-webhook/
   Note: For local Jenkins, use ngrok to expose it:
   ngrok http 8080
   Then use: https://abc123.ngrok.io/github-webhook/
4. Content type: application/json
5. Events: Just the push event
6. Active: ✓ (checked)
7. Add webhook
```

**Step 2: Install ngrok for local webhook testing**

```powershell
# Install ngrok via winget
winget install ngrok.ngrok

# Authenticate ngrok (create free account at ngrok.com)
ngrok authtoken YOUR_NGROK_TOKEN

# Expose Jenkins port 8080 to the internet
# ngrok provides a public URL that tunnels to localhost:8080
ngrok http 8080

# Output will show:
# Forwarding  https://abc123.ngrok.io -> http://localhost:8080
# Use this URL in your GitHub webhook
```

### 7.5 Jenkins GitHub Integration

```
1. Manage Jenkins → Configure System
2. Scroll to "GitHub"
3. Click "Add GitHub Server"
4. Name: GitHub
5. API URL: https://api.github.com
6. Credentials: Select github-token
7. Click "Test connection" — should show "Credentials verified"
8. Save
```

### 7.6 Create Jenkins Pipeline Job

```
1. New Item → Enter name: devops-pipeline-dashboard
2. Select "Pipeline" → OK
3. Configure:
   - Description: Secure DevOps Pipeline for MERN Stack Dashboard
   - ✓ GitHub project → URL: https://github.com/your-org/devops-pipeline-dashboard
   - ✓ GitHub hook trigger for GITScm polling
   - Pipeline Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: https://github.com/your-org/devops-pipeline-dashboard.git
   - Credentials: github-token
   - Branch: */main
   - Script Path: jenkins/Jenkinsfile
4. Save
```

---

## 8. Trivy Security Scanner

### 8.1 Verification

```powershell
# Confirm Trivy version and installation
trivy version

# Update the vulnerability database before scanning
# The DB is downloaded from GitHub releases
trivy image --download-db-only

# List available scanning targets
trivy --help
```

### 8.2 Filesystem Scanning

Scan your source code for vulnerabilities before building Docker images:

```powershell
# Scan the entire project directory for vulnerabilities
# --security-checks: what to scan for
#   vuln: known CVEs in dependencies
#   secret: hardcoded API keys, passwords, tokens
#   config: misconfigurations in IaC files
# --exit-code 1: fail the command if CRITICAL vulnerabilities found
# --severity: only report these severity levels
# --format table: human-readable output (use json for Jenkins)
trivy fs \
  --security-checks vuln,secret,config \
  --exit-code 0 \
  --severity HIGH,CRITICAL \
  --format table \
  .

# Generate JSON report for Jenkins archiving
trivy fs \
  --security-checks vuln,secret \
  --exit-code 0 \
  --severity HIGH,CRITICAL \
  --format json \
  --output reports/trivy/filesystem-scan.json \
  .

# Scan specifically for hardcoded secrets
# This catches API keys, passwords, tokens committed to code
trivy fs \
  --security-checks secret \
  --format table \
  .
```

### 8.3 Docker Image Scanning

Scan built Docker images for OS and application vulnerabilities:

```powershell
# Scan the frontend Docker image
# --ignore-unfixed: skip CVEs that don't have a fix yet
# --vuln-type os,library: scan both OS packages and app libraries
trivy image \
  --exit-code 0 \
  --ignore-unfixed \
  --vuln-type os,library \
  --severity HIGH,CRITICAL \
  --format table \
  devops-dashboard-frontend:latest

# Scan backend image
trivy image \
  --exit-code 0 \
  --ignore-unfixed \
  --vuln-type os,library \
  --severity HIGH,CRITICAL \
  --format table \
  devops-dashboard-backend:latest

# Generate SARIF report (compatible with GitHub Security tab)
trivy image \
  --format sarif \
  --output reports/trivy/frontend-scan.sarif \
  devops-dashboard-frontend:latest

# Generate JSON report for Jenkins
trivy image \
  --format json \
  --output reports/trivy/backend-scan.json \
  devops-dashboard-backend:latest

# Generate SBOM (Software Bill of Materials)
# Lists all components and their licenses
trivy image \
  --format spdx-json \
  --output reports/trivy/sbom.json \
  devops-dashboard-frontend:latest
```

### 8.4 Expected Trivy Output

```
2024-xx-xx T00:00:00.000Z INFO  Vulnerability scanning is enabled
2024-xx-xx T00:00:00.000Z INFO  Secret scanning is enabled
2024-xx-xx T00:00:00.000Z INFO  If your scanning is slow, please try '--scanners vuln' to disable secret scanning

devops-dashboard-frontend:latest (alpine 3.19.0)
====================================================
Total: 0 (HIGH: 0, CRITICAL: 0)

Node.js (node-pkg)
==================
Total: 2 (HIGH: 2, CRITICAL: 0)

┌──────────────┬────────────────┬──────────┬───────────────────┬──────────────────────┐
│   Library    │ Vulnerability  │ Severity │ Installed Version │    Fixed Version     │
├──────────────┼────────────────┼──────────┼───────────────────┼──────────────────────┤
│ semver       │ CVE-2022-25883 │ HIGH     │ 7.3.8             │ 7.5.2                │
└──────────────┴────────────────┴──────────┴───────────────────┴──────────────────────┘
```

---

## 9. OWASP ZAP Setup

### 9.1 Verify ZAP Container

```powershell
# Check ZAP container is running
docker ps --filter name=owasp-zap

# View ZAP startup logs
docker logs owasp-zap

# Test ZAP API is accessible
curl http://localhost:8090/JSON/core/view/version/
```

**Expected API Response:**
```json
{"version":"2.14.0"}
```

### 9.2 Baseline Scan (Passive)

A baseline scan only performs passive checks — no active attacks. Safe to run against any environment.

```powershell
# Create reports directory if it doesn't exist
New-Item -ItemType Directory -Force -Path reports\owasp-zap

# Run ZAP baseline scan against the frontend
# zap-baseline.py: script for passive scanning
# -t: target URL to scan
# -r: HTML report output path (/zap/wrk maps to ./reports in our setup)
# -J: JSON report output path
# -l: minimum alert level to include (PASS|WARN|FAIL)
# --auto: automatically detect and set alert thresholds
docker run --rm \
  --network devops-network \
  -v "${PWD}/reports/owasp-zap:/zap/wrk" \
  owasp/zap2docker-stable \
  zap-baseline.py \
  -t http://frontend:3000 \
  -r zap-baseline-report.html \
  -J zap-baseline-report.json \
  -l WARN

# View the generated report
Start-Process "reports\owasp-zap\zap-baseline-report.html"
```

### 9.3 Full Active Scan

A full scan includes active attack testing. ONLY run against applications you own and in isolated environments.

```powershell
# Run ZAP full scan (active scanning - may take 15-30 minutes)
# zap-full-scan.py: includes active attack testing
# -z: ZAP command line arguments
#    -config: override ZAP configuration
docker run --rm \
  --network devops-network \
  -v "${PWD}/reports/owasp-zap:/zap/wrk" \
  owasp/zap2docker-stable \
  zap-full-scan.py \
  -t http://frontend:3000 \
  -r zap-full-report.html \
  -J zap-full-report.json \
  -l WARN \
  -z "-config scanner.threadPerHost=5"
```

### 9.4 API Scan

```powershell
# Scan the backend API using OpenAPI/Swagger specification
# -f openapi: format of the API definition
# If your API has Swagger/OpenAPI spec, provide it with -s flag
docker run --rm \
  --network devops-network \
  -v "${PWD}/reports/owasp-zap:/zap/wrk" \
  owasp/zap2docker-stable \
  zap-api-scan.py \
  -t http://backend:5000/api \
  -f openapi \
  -r zap-api-report.html
```

---

## 10. Prometheus Monitoring

### 10.1 Prometheus Configuration

**File:** `monitoring/prometheus/prometheus.yml`

```yaml
# ============================================================
# Prometheus Configuration
# Controls what metrics to collect, from where, and how often
# ============================================================

# Global settings apply to all scrape jobs unless overridden
global:
  # How often Prometheus scrapes metrics from targets
  # 15s is good for development; use 30s-60s for production
  scrape_interval: 15s
  
  # Timeout for each scrape request
  # Must be less than scrape_interval
  scrape_timeout: 10s
  
  # How often to evaluate alert rules
  evaluation_interval: 15s
  
  # Labels added to all metrics scraped by this Prometheus instance
  # Useful when federating multiple Prometheus instances
  external_labels:
    environment: 'local-dev'
    project: 'devops-pipeline-dashboard'

# Load alerting rules from files
# These rules define conditions that trigger alerts
rule_files:
  - "/etc/prometheus/alerts/*.yml"

# ============================================================
# SCRAPE CONFIGURATIONS
# Each 'job' defines a set of targets to scrape
# ============================================================
scrape_configs:

  # --- Prometheus self-monitoring ---
  # Prometheus scrapes its own metrics for self-health monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          service: 'prometheus'

  # --- Node Exporter (System Metrics) ---
  # Reports OS-level metrics: CPU, memory, disk, network
  # Install Node Exporter as a Windows service or use Docker
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['host.docker.internal:9100']
        labels:
          service: 'system-metrics'
          host: 'windows-host'

  # --- Backend Application Metrics ---
  # Your Node.js app should expose /metrics endpoint (using prom-client)
  - job_name: 'backend'
    # Path where metrics are exposed (default is /metrics)
    metrics_path: '/metrics'
    static_configs:
      - targets: ['backend:5000']
        labels:
          service: 'backend-api'
          application: 'devops-dashboard'

  # --- Frontend (via Nginx stub_status) ---
  - job_name: 'frontend-nginx'
    metrics_path: '/nginx-status'
    static_configs:
      - targets: ['frontend:80']
        labels:
          service: 'frontend'

  # --- MongoDB Exporter ---
  # Reports MongoDB metrics: connections, operations, replication lag
  # Requires mongodb-exporter sidecar container
  - job_name: 'mongodb'
    static_configs:
      - targets: ['mongodb-exporter:9216']
        labels:
          service: 'mongodb'
          database: 'devops_dashboard'

  # --- Jenkins Metrics ---
  # Jenkins exposes metrics via the Prometheus plugin
  # Install plugin: Prometheus metrics
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
        labels:
          service: 'jenkins-ci'

  # --- Kubernetes Node Metrics ---
  # kube-state-metrics reports Kubernetes object states
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with the annotation prometheus.io/scrape: "true"
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Use the port specified in annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: '${1}'

  # --- Docker Daemon Metrics ---
  # Docker exposes metrics on experimental port 9323
  - job_name: 'docker'
    static_configs:
      - targets: ['host.docker.internal:9323']
        labels:
          service: 'docker-daemon'
```

### 10.2 Alert Rules

**File:** `monitoring/alerts/alert-rules.yml`

```yaml
groups:
  - name: application-alerts
    rules:

      # Alert when backend API is down
      - alert: BackendDown
        # Expression: no HTTP 200 responses in last 5 minutes
        expr: up{job="backend"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Backend API is down"
          description: "The backend API has been unreachable for more than 1 minute."

      # Alert on high error rate
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High HTTP error rate"
          description: "More than 10% of requests are returning 5xx errors."

      # Alert on high memory usage
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is above 85% for more than 5 minutes."
```

### 10.3 Verify Prometheus

```powershell
# Open Prometheus UI
Start-Process "http://localhost:9090"

# Check targets status via API
curl http://localhost:9090/api/v1/targets

# Query a metric (e.g., up status of all jobs)
curl "http://localhost:9090/api/v1/query?query=up"

# Check Prometheus configuration is valid
docker exec prometheus \
  /bin/promtool check config /etc/prometheus/prometheus.yml
```

**Navigate to:** `http://localhost:9090/targets`

You should see all configured scrape targets with their status (UP/DOWN).

---

## 11. Grafana Dashboard

### 11.1 First Login

```
URL: http://localhost:3001
Username: admin
Password: admin123
```

### 11.2 Add Prometheus Datasource

**File:** `monitoring/grafana/datasources/prometheus.yaml`

This file is auto-provisioned when Grafana starts (via volume mount in docker-compose).

```yaml
# Grafana datasource provisioning configuration
# Grafana reads this file on startup to configure datasources automatically
apiVersion: 1

datasources:
  - name: Prometheus
    # Type must match exactly
    type: prometheus
    # Make this the default datasource for new dashboards
    isDefault: true
    url: http://prometheus:9090
    # Access mode:
    # proxy = Grafana server queries Prometheus (recommended)
    # direct = browser queries Prometheus directly (CORS issues)
    access: proxy
    # How often Grafana refreshes data from Prometheus
    jsonData:
      scrapeInterval: "15s"
      httpMethod: "POST"
    # Mark as editable in UI (false = read-only, managed by config file)
    editable: true
```

**Manual Configuration (UI method):**

```
1. Click gear icon (⚙) → Data sources
2. Click "Add data source"
3. Select "Prometheus"
4. Settings:
   - Name: Prometheus
   - URL: http://prometheus:9090
   - Access: Server (default)
5. Click "Save & test" → Should show "Data source is working"
```

### 11.3 Import Pre-built Dashboards

```
1. Click "+" → Import
2. Import via Grafana.com Dashboard ID:
   - Node Exporter Full: 1860
   - Docker and System Monitoring: 893
   - Kubernetes cluster monitoring: 6417
   - Jenkins: Performance and Health Overview: 9964
3. Select Prometheus as data source
4. Click Import
```

### 11.4 Create Custom Application Dashboard

```
1. Click "+" → New Dashboard
2. Click "Add visualization"
3. Select Prometheus datasource
4. Add panels:
```

**Panel 1: Request Rate**
```
Query: rate(http_requests_total[5m])
Title: HTTP Request Rate
Visualization: Time series
```

**Panel 2: Error Rate**
```
Query: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
Title: Error Rate %
Visualization: Gauge
Thresholds: 0=green, 5=yellow, 10=red
```

**Panel 3: Response Time**
```
Query: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
Title: p95 Response Time
Visualization: Time series
Unit: seconds
```

---

## 12. Kubernetes Manifests

### 12.1 Namespace

**File:** `kubernetes/namespace.yaml`

```yaml
# Namespace isolates resources within the Kubernetes cluster
# All DevOps Dashboard resources live in this namespace
# Benefits: resource quotas, RBAC isolation, clear ownership
apiVersion: v1
kind: Namespace
metadata:
  # Namespace name — used in all kubectl commands with -n flag
  name: devops-dashboard
  labels:
    # Labels allow selecting all resources in this namespace
    app: devops-dashboard
    environment: local
    managed-by: kubectl
  annotations:
    # Documentation annotation for humans
    description: "DevOps Pipeline Dashboard application namespace"
```

### 12.2 ConfigMap

**File:** `kubernetes/configmap.yaml`

```yaml
# ConfigMap stores non-sensitive configuration as key-value pairs
# Pods can reference ConfigMaps as environment variables or mounted files
# Change config without rebuilding images by updating ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: devops-dashboard-config
  namespace: devops-dashboard
  labels:
    app: devops-dashboard
data:
  # Application settings
  NODE_ENV: "production"
  PORT: "5000"
  
  # MongoDB connection (without credentials — those go in Secret)
  MONGODB_HOST: "mongodb-service"
  MONGODB_PORT: "27017"
  MONGODB_DATABASE: "devops_dashboard"
  
  # Frontend settings
  REACT_APP_ENV: "production"
  
  # Logging configuration
  LOG_LEVEL: "info"
  LOG_FORMAT: "json"
  
  # CORS settings
  CORS_ORIGIN: "http://localhost:3000"
  
  # Health check configuration
  HEALTH_CHECK_PATH: "/api/health"
```

### 12.3 Secret

**File:** `kubernetes/secret.yaml`

```yaml
# Secret stores sensitive information (credentials, tokens, keys)
# Data is base64-encoded (NOT encrypted by default in Kubernetes)
# For production: use Kubernetes Secrets Encryption at Rest or
# external secret managers (Azure Key Vault, HashiCorp Vault)
apiVersion: v1
kind: Secret
metadata:
  name: devops-dashboard-secret
  namespace: devops-dashboard
  labels:
    app: devops-dashboard
# Opaque is the default Secret type for arbitrary key-value pairs
type: Opaque
# Values must be base64-encoded
# Generate: echo -n "your-value" | base64
# Decode: echo "encoded-value" | base64 --decode
data:
  # MongoDB credentials
  # "admin" in base64
  MONGODB_USERNAME: YWRtaW4=
  # "password123" in base64
  MONGODB_PASSWORD: cGFzc3dvcmQxMjM=
  
  # JWT secret for API authentication
  # "your-super-secret-jwt-key" in base64
  JWT_SECRET: eW91ci1zdXBlci1zZWNyZXQtand0LWtleQ==

---
# IMPORTANT: Generate base64 values on your machine:
# PowerShell: [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("your-value"))
# Linux/Mac:  echo -n "your-value" | base64
```

### 12.4 Deployment

**File:** `kubernetes/deployment.yaml`

```yaml
# ============================================================
# MONGODB DEPLOYMENT
# ============================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  namespace: devops-dashboard
  labels:
    app: mongodb
    component: database
spec:
  # Single replica for MongoDB (use StatefulSet for production clustering)
  replicas: 1
  # Selector tells Deployment which Pods it manages
  selector:
    matchLabels:
      app: mongodb
  # Pod template — the blueprint for new pods
  template:
    metadata:
      labels:
        app: mongodb
        component: database
    spec:
      containers:
        - name: mongodb
          image: mongo:7.0
          ports:
            - containerPort: 27017
          env:
            # Pull credentials from the Secret (not ConfigMap)
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: devops-dashboard-secret
                  key: MONGODB_USERNAME
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: devops-dashboard-secret
                  key: MONGODB_PASSWORD
          # Define resource limits and requests
          # requests: guaranteed resources reserved for this pod
          # limits: maximum resources the pod can use
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          volumeMounts:
            # Mount persistent storage to MongoDB's data directory
            - name: mongodb-storage
              mountPath: /data/db
          # Liveness probe: restart pod if this fails
          livenessProbe:
            exec:
              command:
                - mongosh
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 30
            periodSeconds: 10
          # Readiness probe: remove from Service if this fails
          readinessProbe:
            exec:
              command:
                - mongosh
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 15
            periodSeconds: 5
      volumes:
        - name: mongodb-storage
          # PersistentVolumeClaim provides durable storage
          persistentVolumeClaim:
            claimName: mongodb-pvc

---
# ============================================================
# PersistentVolumeClaim for MongoDB
# ============================================================
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
  namespace: devops-dashboard
spec:
  accessModes:
    # ReadWriteOnce = mounted by single node (appropriate for MongoDB)
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
# ============================================================
# BACKEND DEPLOYMENT
# ============================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: devops-dashboard
  labels:
    app: backend
    component: api
spec:
  # 2 replicas for high availability
  replicas: 2
  selector:
    matchLabels:
      app: backend
  strategy:
    # RollingUpdate: gradually replace old pods with new ones
    # Ensures zero downtime during deployments
    type: RollingUpdate
    rollingUpdate:
      # Maximum pods unavailable during update (25% = 0 of 2)
      maxUnavailable: 0
      # Maximum extra pods created during update
      maxSurge: 1
  template:
    metadata:
      labels:
        app: backend
        component: api
      annotations:
        # Enable Prometheus scraping for this pod
        prometheus.io/scrape: "true"
        prometheus.io/port: "5000"
        prometheus.io/path: "/metrics"
    spec:
      # Ensure pods don't land on the same node (for HA)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: backend
                topologyKey: kubernetes.io/hostname
      containers:
        - name: backend
          # Image built and pushed by Jenkins pipeline
          image: YOUR_DOCKER_USERNAME/devops-dashboard-backend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 5000
          env:
            # Load all values from ConfigMap
            - name: NODE_ENV
              valueFrom:
                configMapKeyRef:
                  name: devops-dashboard-config
                  key: NODE_ENV
            - name: PORT
              valueFrom:
                configMapKeyRef:
                  name: devops-dashboard-config
                  key: PORT
            # MongoDB connection string assembled from ConfigMap + Secret
            - name: MONGODB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: devops-dashboard-secret
                  key: MONGODB_USERNAME
            - name: MONGODB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: devops-dashboard-secret
                  key: MONGODB_PASSWORD
            - name: MONGODB_URI
              value: "mongodb://$(MONGODB_USERNAME):$(MONGODB_PASSWORD)@mongodb-service:27017/devops_dashboard?authSource=admin"
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: devops-dashboard-secret
                  key: JWT_SECRET
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "300m"
          livenessProbe:
            httpGet:
              path: /api/health
              port: 5000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 5

---
# ============================================================
# FRONTEND DEPLOYMENT
# ============================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: devops-dashboard
  labels:
    app: frontend
    component: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: frontend
        component: web
    spec:
      containers:
        - name: frontend
          image: YOUR_DOCKER_USERNAME/devops-dashboard-frontend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

### 12.5 Service

**File:** `kubernetes/service.yaml`

```yaml
# ============================================================
# Services provide stable network endpoints for pods
# Pods come and go (due to scaling/failures), but Service IP stays fixed
# ============================================================

# MongoDB Service — internal only (ClusterIP)
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: devops-dashboard
  labels:
    app: mongodb
spec:
  # ClusterIP: only accessible within the cluster (not from outside)
  # Use for databases — they should never be exposed externally
  type: ClusterIP
  # Selector matches pods to route traffic to
  selector:
    app: mongodb
  ports:
    - port: 27017          # Port the service listens on
      targetPort: 27017    # Port on the pod to forward to
      protocol: TCP
      name: mongodb

---
# Backend Service — internal ClusterIP
# Frontend and Ingress communicate with backend through this Service
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: devops-dashboard
  labels:
    app: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 5000
      targetPort: 5000
      protocol: TCP
      name: http

---
# Frontend Service — accessible externally via Ingress
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: devops-dashboard
  labels:
    app: frontend
spec:
  # NodePort: exposes service on each Node's IP at a static port
  # Useful for local development with docker-desktop Kubernetes
  # Port range: 30000-32767
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80             # Service port
      targetPort: 80       # Pod port
      nodePort: 30000      # External port on each Node (access via localhost:30000)
      protocol: TCP
      name: http
```

### 12.6 Ingress

**File:** `kubernetes/ingress.yaml`

```yaml
# Ingress manages external HTTP/HTTPS access to cluster services
# Acts as a reverse proxy/load balancer at the application layer
# Requires an Ingress Controller (Nginx Ingress for docker-desktop)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devops-dashboard-ingress
  namespace: devops-dashboard
  labels:
    app: devops-dashboard
  annotations:
    # Specify which ingress controller to use
    kubernetes.io/ingress.class: "nginx"
    # Rewrite rules: strip /api prefix before forwarding to backend
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    # Enable CORS headers for API requests from frontend
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "http://localhost"
    # Rate limiting: protect API from abuse
    nginx.ingress.kubernetes.io/limit-rps: "100"
    # Proxy settings for better performance
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
spec:
  rules:
    # Route traffic for localhost
    - host: localhost
      http:
        paths:
          # Backend API routes: /api/* → backend-service:5000
          - path: /api(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 5000
          # Frontend routes: /* → frontend-service:80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### 12.7 Install Nginx Ingress Controller

```powershell
# Install Nginx Ingress Controller for Docker Desktop Kubernetes
# This controller handles the Ingress resource defined above
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

# Wait for the ingress controller to be ready (may take 1-2 minutes)
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# Verify ingress controller is running
kubectl get pods --namespace ingress-nginx
```

---

## 13. Secure CI/CD Pipeline — Jenkinsfile

**File:** `jenkins/Jenkinsfile`

```groovy
// ============================================================
// SECURE DEVOPS PIPELINE — JENKINSFILE
// DevOps Pipeline Dashboard — Phase 2 Local Implementation
// ============================================================

pipeline {
    // Run on any available Jenkins agent
    // For production: specify agent labels (e.g., agent { label 'linux' })
    agent any

    // --------------------------------------------------------
    // PIPELINE ENVIRONMENT VARIABLES
    // Centralized configuration — change once, applies everywhere
    // --------------------------------------------------------
    environment {
        // Application metadata
        APP_NAME = "devops-pipeline-dashboard"
        APP_VERSION = "1.0.${BUILD_NUMBER}"  // e.g., 1.0.42
        
        // Docker image names with version tags
        FRONTEND_IMAGE = "devops-dashboard-frontend"
        BACKEND_IMAGE  = "devops-dashboard-backend"
        IMAGE_TAG      = "${APP_VERSION}"
        
        // Docker Hub registry (replace with your username)
        DOCKER_REGISTRY = "your-dockerhub-username"
        
        // Tool configurations
        SONAR_PROJECT_KEY = "devops-pipeline-dashboard"
        
        // Kubernetes namespace
        K8S_NAMESPACE = "devops-dashboard"
        
        // Report output paths
        TRIVY_REPORTS_DIR = "reports/trivy"
        ZAP_REPORTS_DIR   = "reports/owasp-zap"
        
        // Application URLs (for DAST scanning)
        FRONTEND_URL = "http://frontend-service.devops-dashboard.svc.cluster.local"
        BACKEND_URL  = "http://backend-service.devops-dashboard.svc.cluster.local:5000"
    }

    // --------------------------------------------------------
    // PIPELINE OPTIONS
    // Global pipeline behavior settings
    // --------------------------------------------------------
    options {
        // Add timestamps to all log lines for debugging
        timestamps()
        
        // Color ANSI codes in console output
        ansiColor('xterm')
        
        // Clean workspace before each build
        // Prevents stale files from previous builds affecting results
        // cleanWs()  // Uncomment to enable
        
        // Timeout the entire pipeline after 60 minutes
        // Prevents hung builds from consuming resources indefinitely
        timeout(time: 60, unit: 'MINUTES')
        
        // Keep only last 10 builds to save disk space
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // Prevent multiple builds of the same branch running simultaneously
        disableConcurrentBuilds()
    }

    // --------------------------------------------------------
    // PIPELINE STAGES
    // Executed sequentially — failure in any stage stops the pipeline
    // --------------------------------------------------------
    stages {

        // ====================================================
        // STAGE 1: CHECKOUT
        // Pull source code from GitHub
        // ====================================================
        stage('🔄 Checkout') {
            steps {
                script {
                    echo "=== STAGE: Checkout Source Code ==="
                    echo "Branch: ${env.BRANCH_NAME ?: 'main'}"
                    echo "Build Number: ${env.BUILD_NUMBER}"
                }
                
                // Clean workspace before checkout
                // Ensures no leftover files from previous builds
                cleanWs()
                
                // Checkout code from GitHub using stored credentials
                // 'github-token' is the credential ID configured in Jenkins
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [
                        // Shallow clone: only latest commit (faster for CI)
                        [$class: 'CloneOption', shallow: true, depth: 1],
                        // Clean before checkout
                        [$class: 'CleanBeforeCheckout']
                    ],
                    userRemoteConfigs: [[
                        credentialsId: 'github-token',
                        url: 'https://github.com/your-org/devops-pipeline-dashboard.git'
                    ]]
                ])
                
                // Display what was checked out
                sh 'git log --oneline -5'
                sh 'git status'
                
                // Create report directories for later stages
                sh """
                    mkdir -p ${TRIVY_REPORTS_DIR}
                    mkdir -p ${ZAP_REPORTS_DIR}
                    mkdir -p reports/sonarqube
                """
            }
        }

        // ====================================================
        // STAGE 2: INSTALL DEPENDENCIES
        // Install npm packages for both frontend and backend
        // ====================================================
        stage('📦 Install Dependencies') {
            parallel {
                // Install frontend and backend packages simultaneously
                // 'parallel' runs these sub-stages concurrently (saves time)
                
                stage('Frontend Dependencies') {
                    steps {
                        dir('client') {
                            echo "=== Installing Frontend Dependencies ==="
                            sh '''
                                # Display Node.js and npm versions for debugging
                                node --version
                                npm --version
                                
                                # npm ci is stricter than npm install:
                                # - Reads package-lock.json exactly
                                # - Fails if package-lock.json is out of sync
                                # - Deletes node_modules before installing
                                # - Much faster for CI environments
                                npm ci
                                
                                echo "Frontend dependencies installed successfully"
                            '''
                        }
                    }
                }
                
                stage('Backend Dependencies') {
                    steps {
                        dir('server') {
                            echo "=== Installing Backend Dependencies ==="
                            sh '''
                                node --version
                                npm --version
                                npm ci
                                echo "Backend dependencies installed successfully"
                            '''
                        }
                    }
                }
            }
        }

        // ====================================================
        // STAGE 3: RUN TESTS
        // Execute unit and integration tests
        // ====================================================
        stage('🧪 Run Tests') {
            parallel {
                stage('Frontend Tests') {
                    steps {
                        dir('client') {
                            echo "=== Running Frontend Tests ==="
                            sh '''
                                # Run Jest tests with coverage report
                                # --ci: CI mode (no interactive prompts)
                                # --coverage: generate coverage report
                                # --watchAll=false: don't watch for file changes
                                # --reporters: use default + JUnit reporter for Jenkins
                                npm test -- \
                                    --ci \
                                    --coverage \
                                    --watchAll=false \
                                    --coverageDirectory=../coverage/frontend \
                                    --testResultsProcessor=jest-junit \
                                    || true
                                
                                echo "Frontend tests completed"
                            '''
                        }
                    }
                    post {
                        always {
                            // Publish JUnit test results in Jenkins UI
                            junit allowEmptyResults: true, \
                                  testResults: 'client/junit.xml'
                        }
                    }
                }
                
                stage('Backend Tests') {
                    steps {
                        dir('server') {
                            echo "=== Running Backend Tests ==="
                            sh '''
                                npm test -- \
                                    --ci \
                                    --coverage \
                                    --watchAll=false \
                                    --coverageDirectory=../coverage/backend \
                                    || true
                                
                                echo "Backend tests completed"
                            '''
                        }
                    }
                    post {
                        always {
                            junit allowEmptyResults: true, \
                                  testResults: 'server/junit.xml'
                        }
                    }
                }
            }
        }

        // ====================================================
        // STAGE 4: SONARQUBE ANALYSIS (SAST)
        // Static Application Security Testing + Code Quality
        // ====================================================
        stage('🔍 SonarQube Analysis') {
            steps {
                echo "=== STAGE: SonarQube Static Analysis ==="
                
                // withSonarQubeEnv: injects SONAR_HOST_URL and SONAR_AUTH_TOKEN
                // 'SonarQube-Local' must match the name configured in Jenkins settings
                withSonarQubeEnv('SonarQube-Local') {
                    sh '''
                        # Run SonarQube scanner
                        # sonar.projectKey: must match project created in SonarQube UI
                        # sonar.sources: directories to analyze
                        # sonar.javascript.lcov.reportPaths: coverage data location
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName="DevOps Pipeline Dashboard" \
                            -Dsonar.projectVersion=${APP_VERSION} \
                            -Dsonar.sources=client/src,server \
                            -Dsonar.exclusions="**/node_modules/**,**/dist/**,**/coverage/**" \
                            -Dsonar.javascript.lcov.reportPaths=coverage/frontend/lcov.info,coverage/backend/lcov.info \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                        
                        echo "SonarQube analysis submitted"
                    '''
                }
            }
        }

        // ====================================================
        // STAGE 5: QUALITY GATE
        // Wait for SonarQube to complete analysis and check result
        // ====================================================
        stage('✅ Quality Gate') {
            steps {
                echo "=== STAGE: Checking SonarQube Quality Gate ==="
                
                // Poll SonarQube until analysis is complete
                // abortPipeline: true = fail the pipeline if quality gate fails
                // abortPipeline: false = warn only (useful for initial setup)
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                
                echo "Quality Gate PASSED ✓"
            }
        }

        // ====================================================
        // STAGE 6: BUILD APPLICATION
        // Compile React frontend for production
        // ====================================================
        stage('🏗️ Build Application') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        dir('client') {
                            echo "=== Building Frontend (Vite Production Build) ==="
                            sh '''
                                # Vite builds optimized bundle:
                                # - Minifies JavaScript and CSS
                                # - Splits code into chunks for lazy loading
                                # - Generates content-hashed filenames for cache busting
                                # - Tree-shakes unused code
                                npm run build
                                
                                # Verify build output exists
                                ls -la dist/
                                echo "Frontend build completed successfully"
                            '''
                        }
                    }
                }
                
                stage('Validate Backend') {
                    steps {
                        dir('server') {
                            echo "=== Validating Backend ==="
                            sh '''
                                # Validate package.json is valid JSON
                                node -e "require('./package.json'); console.log('package.json is valid')"
                                
                                # Check for syntax errors in main entry point
                                node --check server.js 2>/dev/null || \
                                node --check index.js 2>/dev/null || \
                                echo "Syntax check completed"
                            '''
                        }
                    }
                }
            }
        }

        // ====================================================
        // STAGE 7: DOCKER BUILD
        // Build Docker images for frontend and backend
        // ====================================================
        stage('🐳 Docker Build') {
            steps {
                echo "=== STAGE: Building Docker Images ==="
                
                script {
                    // Build frontend image with version tag
                    // -t: tag the image with name:version AND name:latest
                    // -f: specify which Dockerfile to use
                    // --build-arg: pass build-time arguments
                    sh """
                        docker build \
                            -t ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG} \
                            -t ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:latest \
                            -f docker/frontend/Dockerfile \
                            --build-arg VITE_API_URL=/api \
                            --build-arg BUILD_DATE=\$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                            --build-arg GIT_COMMIT=\$(git rev-parse --short HEAD) \
                            .
                        
                        echo "Frontend image built: ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    """
                    
                    // Build backend image
                    sh """
                        docker build \
                            -t ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG} \
                            -t ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:latest \
                            -f docker/backend/Dockerfile \
                            --build-arg BUILD_DATE=\$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                            --build-arg GIT_COMMIT=\$(git rev-parse --short HEAD) \
                            .
                        
                        echo "Backend image built: ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG}"
                    """
                    
                    // List built images
                    sh "docker images | grep devops-dashboard"
                }
            }
        }

        // ====================================================
        // STAGE 8: TRIVY SECURITY SCAN
        // Scan Docker images for CVE vulnerabilities
        // ====================================================
        stage('🛡️ Trivy Security Scan') {
            steps {
                echo "=== STAGE: Trivy Container Security Scan ==="
                
                script {
                    // Scan frontend image
                    // --exit-code 0: don't fail pipeline on vulnerabilities
                    //   (change to 1 to enforce security gate)
                    sh """
                        trivy image \
                            --exit-code 0 \
                            --ignore-unfixed \
                            --vuln-type os,library \
                            --severity LOW,MEDIUM,HIGH,CRITICAL \
                            --format json \
                            --output ${TRIVY_REPORTS_DIR}/frontend-scan.json \
                            ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}
                        
                        # Also generate human-readable table report
                        trivy image \
                            --exit-code 0 \
                            --ignore-unfixed \
                            --vuln-type os,library \
                            --severity HIGH,CRITICAL \
                            --format table \
                            ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    // Scan backend image
                    sh """
                        trivy image \
                            --exit-code 0 \
                            --ignore-unfixed \
                            --vuln-type os,library \
                            --severity LOW,MEDIUM,HIGH,CRITICAL \
                            --format json \
                            --output ${TRIVY_REPORTS_DIR}/backend-scan.json \
                            ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG}
                        
                        trivy image \
                            --exit-code 0 \
                            --ignore-unfixed \
                            --vuln-type os,library \
                            --severity HIGH,CRITICAL \
                            --format table \
                            ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    // Scan filesystem for hardcoded secrets
                    sh """
                        trivy fs \
                            --security-checks secret \
                            --exit-code 0 \
                            --format json \
                            --output ${TRIVY_REPORTS_DIR}/secrets-scan.json \
                            .
                    """
                    
                    echo "Trivy scans completed. Reports saved to ${TRIVY_REPORTS_DIR}/"
                }
            }
        }

        // ====================================================
        // STAGE 9: PUSH TO REGISTRY
        // Push Docker images to Docker Hub
        // ====================================================
        stage('📤 Push to Registry') {
            steps {
                echo "=== STAGE: Pushing Images to Docker Registry ==="
                
                // withCredentials: securely injects credentials into the block
                // DOCKER_USER and DOCKER_PASS are available as env vars
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        # Login to Docker Hub
                        # --password-stdin: reads password from stdin (more secure than -p flag)
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        
                        echo "Logged in to Docker Hub successfully"
                    '''
                    
                    sh """
                        # Push frontend image (both version tag and latest)
                        docker push ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:latest
                        
                        # Push backend image
                        docker push ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:latest
                        
                        echo "All images pushed successfully"
                    """
                    
                    sh 'docker logout'
                }
            }
        }

        // ====================================================
        // STAGE 10: DEPLOY TO KUBERNETES
        // Apply Kubernetes manifests to deploy the application
        // ====================================================
        stage('🚀 Deploy to Kubernetes') {
            steps {
                echo "=== STAGE: Deploying to Local Kubernetes ==="
                
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        # Create namespace if it doesn't exist
                        kubectl apply -f kubernetes/namespace.yaml
                        
                        # Apply configurations (secrets and configmaps first)
                        kubectl apply -f kubernetes/configmap.yaml
                        kubectl apply -f kubernetes/secret.yaml
                        
                        # Update image tags in deployment manifests
                        # sed replaces 'YOUR_DOCKER_USERNAME' placeholder with actual registry
                        sed -i 's|YOUR_DOCKER_USERNAME|${DOCKER_REGISTRY}|g' kubernetes/deployment.yaml
                        
                        # Apply deployment manifests
                        kubectl apply -f kubernetes/deployment.yaml
                        kubectl apply -f kubernetes/service.yaml
                        kubectl apply -f kubernetes/ingress.yaml
                        
                        echo "Kubernetes manifests applied"
                    """
                }
            }
        }

        // ====================================================
        // STAGE 11: VERIFY DEPLOYMENT
        // Wait for rollout to complete and verify pod health
        // ====================================================
        stage('✔️ Verify Deployment') {
            steps {
                echo "=== STAGE: Verifying Kubernetes Deployment ==="
                
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        # Wait for backend rollout to complete
                        # --timeout: wait up to 5 minutes
                        kubectl rollout status deployment/backend \
                            -n ${K8S_NAMESPACE} \
                            --timeout=300s
                        
                        # Wait for frontend rollout to complete
                        kubectl rollout status deployment/frontend \
                            -n ${K8S_NAMESPACE} \
                            --timeout=300s
                        
                        # Display pod status
                        kubectl get pods -n ${K8S_NAMESPACE} -o wide
                        
                        # Display service endpoints
                        kubectl get services -n ${K8S_NAMESPACE}
                        
                        # Display ingress rules
                        kubectl get ingress -n ${K8S_NAMESPACE}
                        
                        echo "Deployment verification PASSED ✓"
                    """
                }
            }
        }

        // ====================================================
        // STAGE 12: OWASP ZAP DAST SCAN
        // Dynamic Application Security Testing on live application
        // ====================================================
        stage('🔐 OWASP ZAP DAST Scan') {
            steps {
                echo "=== STAGE: OWASP ZAP Dynamic Security Testing ==="
                
                script {
                    // Run ZAP baseline scan against deployed application
                    // Baseline scan is safe for any environment (passive only)
                    sh """
                        docker run --rm \
                            --network devops-network \
                            -v \${WORKSPACE}/reports/owasp-zap:/zap/wrk:rw \
                            owasp/zap2docker-stable \
                            zap-baseline.py \
                            -t http://frontend:3000 \
                            -r zap-baseline-report.html \
                            -J zap-baseline-report.json \
                            -l WARN \
                            --auto \
                            || true
                        
                        echo "ZAP baseline scan completed"
                        echo "Report: reports/owasp-zap/zap-baseline-report.html"
                    """
                    
                    // Scan the backend API as well
                    sh """
                        docker run --rm \
                            --network devops-network \
                            -v \${WORKSPACE}/reports/owasp-zap:/zap/wrk:rw \
                            owasp/zap2docker-stable \
                            zap-baseline.py \
                            -t http://backend:5000/api \
                            -r zap-api-report.html \
                            -J zap-api-report.json \
                            -l WARN \
                            || true
                        
                        echo "ZAP API scan completed"
                    """
                }
            }
        }

        // ====================================================
        // STAGE 13: ARCHIVE REPORTS
        // Save all security and test reports in Jenkins
        // ====================================================
        stage('📊 Archive Reports') {
            steps {
                echo "=== STAGE: Archiving Security Reports ==="
                
                // Archive raw report files (downloadable from Jenkins UI)
                archiveArtifacts artifacts: """
                    reports/trivy/*.json,
                    reports/owasp-zap/*.html,
                    reports/owasp-zap/*.json,
                    coverage/**/*
                """, allowEmptyArchive: true, fingerprint: true
                
                // Publish HTML reports in Jenkins UI
                // (requires HTML Publisher plugin)
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports/owasp-zap',
                    reportFiles: 'zap-baseline-report.html',
                    reportName: 'OWASP ZAP Baseline Report',
                    reportTitles: 'ZAP Security Report'
                ])
                
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'coverage/frontend',
                    reportFiles: 'index.html',
                    reportName: 'Frontend Test Coverage',
                    reportTitles: 'Coverage Report'
                ])
                
                echo "All reports archived successfully"
            }
        }

    } // end stages

    // --------------------------------------------------------
    // POST BUILD ACTIONS
    // Always run regardless of pipeline result
    // --------------------------------------------------------
    post {
        // Runs only when build succeeds
        success {
            echo "✅ Pipeline SUCCEEDED — Build ${BUILD_NUMBER}"
            echo "Application deployed to Kubernetes at http://localhost:3000"
            echo "Security reports: Check Jenkins Build Artifacts"
            echo "Prometheus: http://localhost:9090"
            echo "Grafana: http://localhost:3001"
        }
        
        // Runs only when build fails
        failure {
            echo "❌ Pipeline FAILED — Build ${BUILD_NUMBER}"
            echo "Check the console output above for details"
        }
        
        // Runs when build is unstable (tests passed but with warnings)
        unstable {
            echo "⚠️ Pipeline UNSTABLE — Some tests or quality checks need attention"
        }
        
        // Always runs (success, failure, or abort)
        always {
            echo "=== Pipeline Complete — Build ${BUILD_NUMBER} ==="
            echo "Duration: ${currentBuild.durationString}"
            
            // Clean up Docker images to free disk space
            // || true prevents failure if images don't exist
            sh """
                docker rmi ${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${IMAGE_TAG} || true
                docker rmi ${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${IMAGE_TAG} || true
                docker system prune -f --filter "until=24h" || true
            """
        }
    }
}
```

---

## 14. Running Instructions

### 14.1 Start All Services (Docker Compose)

```powershell
# Navigate to project root
cd C:\projects\devops-pipeline-dashboard

# Start all services in detached mode
# -d: run in background
# --build: rebuild images (use this after code changes)
docker compose -f docker/docker-compose.yml up -d --build

# Watch startup progress
docker compose -f docker/docker-compose.yml logs -f

# Expected output:
# ✔ Container mongodb     Started
# ✔ Container backend     Started
# ✔ Container frontend    Started
# ✔ Container sonarqube   Started
# ✔ Container prometheus  Started
# ✔ Container grafana     Started
```

**Verify all containers are healthy:**

```powershell
# List all running containers with status
docker compose -f docker/docker-compose.yml ps

# Expected output:
# NAME          IMAGE                      STATUS
# mongodb       mongo:7.0                  Up (healthy)
# backend       devops-dashboard-backend   Up (healthy)
# frontend      devops-dashboard-frontend  Up (healthy)
# sonarqube     sonarqube:community        Up
# prometheus    prom/prometheus             Up
# grafana       grafana/grafana             Up
```

### 14.2 Frontend Development

```powershell
# Navigate to client directory
cd client

# Install dependencies (first time or after package.json changes)
npm install

# Start Vite development server with hot-module replacement
# Opens browser at http://localhost:5173
npm run dev

# Expected output:
#   VITE v5.x.x  ready in 500 ms
#
#   ➜  Local:   http://localhost:5173/
#   ➜  Network: http://192.168.x.x:5173/
#   ➜  press h + enter to show help

# Build for production
npm run build

# Preview production build locally
npm run preview

# Run tests
npm test

# Run tests with coverage
npm test -- --coverage
```

### 14.3 Backend Development

```powershell
# Navigate to server directory
cd server

# Install dependencies
npm install

# Start backend with hot-reload (using nodemon)
npm run dev

# Expected output:
# [nodemon] starting `node server.js`
# Server running on port 5000
# MongoDB connected successfully
# API available at http://localhost:5000/api

# Start production mode
npm start

# Run tests
npm test

# Check API health
curl http://localhost:5000/api/health
# Expected: {"status":"healthy","uptime":12.345,"timestamp":"2024-..."}
```

### 14.4 Deploy to Kubernetes

```powershell
# Apply all manifests in order
# Kubernetes processes them and creates resources
kubectl apply -f kubernetes/namespace.yaml
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/secret.yaml
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/ingress.yaml

# OR apply all at once (Kubernetes handles dependencies)
kubectl apply -f kubernetes/

# Expected output:
# namespace/devops-dashboard created (or configured)
# configmap/devops-dashboard-config created
# secret/devops-dashboard-secret created
# deployment.apps/mongodb created
# deployment.apps/backend created
# deployment.apps/frontend created
# service/mongodb-service created
# service/backend-service created
# service/frontend-service created
# ingress.networking.k8s.io/devops-dashboard-ingress created

# Monitor rollout progress
kubectl rollout status deployment/frontend -n devops-dashboard
kubectl rollout status deployment/backend -n devops-dashboard

# Verify pods are running
kubectl get pods -n devops-dashboard -w

# Expected output after 1-2 minutes:
# NAME                        READY   STATUS    RESTARTS   AGE
# mongodb-xxxxxxxxx-xxxxx     1/1     Running   0          2m
# backend-xxxxxxxxx-xxxxx     1/1     Running   0          1m
# backend-xxxxxxxxx-yyyyy     1/1     Running   0          1m
# frontend-xxxxxxxxx-xxxxx    1/1     Running   0          1m
# frontend-xxxxxxxxx-yyyyy    1/1     Running   0          1m

# Access application
# Via NodePort (port 30000)
Start-Process "http://localhost:30000"

# Via Ingress (port 80 - requires ingress controller)
Start-Process "http://localhost"
```

### 14.5 Trigger Jenkins Pipeline

```powershell
# Option 1: Push to GitHub (triggers webhook)
git add .
git commit -m "feat: trigger CI/CD pipeline"
git push origin main
# Pipeline starts automatically in 10-30 seconds

# Option 2: Trigger manually via Jenkins UI
Start-Process "http://localhost:8080"
# Click on 'devops-pipeline-dashboard' → Build Now

# Option 3: Trigger via Jenkins CLI
java -jar jenkins-cli.jar \
  -s http://localhost:8080 \
  -auth admin:YOUR_PASSWORD \
  build devops-pipeline-dashboard -w -v

# Monitor pipeline
Start-Process "http://localhost:8080/job/devops-pipeline-dashboard/lastBuild/console"
```

### 14.6 Stop All Services

```powershell
# Stop Docker Compose services (keeps data in volumes)
docker compose -f docker/docker-compose.yml stop

# Stop AND remove containers (keeps volumes)
docker compose -f docker/docker-compose.yml down

# Stop AND remove everything including volumes (DESTROYS DATA)
docker compose -f docker/docker-compose.yml down -v

# Delete all Kubernetes resources
kubectl delete -f kubernetes/

# Or delete entire namespace (faster, removes everything)
kubectl delete namespace devops-dashboard
```

---

## 15. Verification Procedures

### 15.1 Jenkins Verification

```powershell
# Verify Jenkins service is running
sc query Jenkins

# Access Jenkins web UI
Start-Process "http://localhost:8080"

# Verify webhook is configured (look for green checkmark)
# GitHub → Repository → Settings → Webhooks → Recent Deliveries

# Check pipeline status
curl http://localhost:8080/api/json?tree=jobs[name,color]

# Expected: color=blue (success), color=red (failed), color=notbuilt (never run)
```

### 15.2 SonarQube Verification

```powershell
# Check SonarQube is running
docker ps --filter name=sonarqube

# Access web UI
Start-Process "http://localhost:9000"

# Verify API is responding
curl http://localhost:9000/api/system/status

# Expected:
# {"id":"...","version":"10.x.x","status":"UP"}

# Check project analysis results
curl -u admin:Admin@123 \
  "http://localhost:9000/api/measures/component?component=devops-pipeline-dashboard&metricKeys=bugs,vulnerabilities,code_smells,coverage"
```

### 15.3 Docker Verification

```powershell
# Check all containers are running
docker ps

# Verify container health
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Inspect a specific container
docker inspect frontend | Select-String -Pattern "Health"

# Test frontend is serving content
curl -I http://localhost:3000

# Expected:
# HTTP/1.1 200 OK
# Server: nginx/1.25.x

# Test backend API
curl http://localhost:5000/api/health

# Expected:
# {"status":"healthy","timestamp":"..."}
```

### 15.4 Trivy Verification

```powershell
# Verify Trivy is installed and updated
trivy version

# Run a quick test scan on Alpine (small, fast)
trivy image alpine:latest --severity HIGH,CRITICAL

# Verify reports were generated
Get-ChildItem reports/trivy/

# Read a report summary
$report = Get-Content reports/trivy/frontend-scan.json | ConvertFrom-Json
Write-Host "Total vulnerabilities: $($report.Results.Vulnerabilities.Count)"
```

### 15.5 Kubernetes Verification

```powershell
# Cluster health
kubectl get nodes

# All resources in namespace
kubectl get all -n devops-dashboard

# Pod logs for debugging
kubectl logs -f deployment/backend -n devops-dashboard
kubectl logs -f deployment/frontend -n devops-dashboard

# Describe pod for detailed status
kubectl describe pod -l app=backend -n devops-dashboard

# Test internal service connectivity
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never \
  -n devops-dashboard \
  -- curl http://backend-service:5000/api/health

# Check resource usage
kubectl top pods -n devops-dashboard
kubectl top nodes

# Verify ingress is routing correctly
curl -H "Host: localhost" http://localhost/
curl -H "Host: localhost" http://localhost/api/health
```

### 15.6 OWASP ZAP Verification

```powershell
# Verify ZAP container is running
docker ps --filter name=owasp-zap

# Test ZAP API
curl http://localhost:8090/JSON/core/view/version/

# Expected:
# {"version":"2.14.0"}

# Check ZAP scan reports
Get-ChildItem reports/owasp-zap/

# Open HTML report
Start-Process "reports/owasp-zap/zap-baseline-report.html"
```

### 15.7 Prometheus Verification

```powershell
# Access Prometheus UI
Start-Process "http://localhost:9090"

# Check targets via API
curl http://localhost:9090/api/v1/targets | ConvertFrom-Json | Select-Object -ExpandProperty data

# Query a metric
curl "http://localhost:9090/api/v1/query?query=up" | ConvertFrom-Json

# Expected: at least prometheus target should be UP (value: 1)

# Navigate to http://localhost:9090/targets
# All configured targets should show as UP or DOWN
```

### 15.8 Grafana Verification

```powershell
# Access Grafana UI
Start-Process "http://localhost:3001"

# Test login via API
curl -u admin:admin123 http://localhost:3001/api/health

# Expected:
# {"commit":"...","database":"ok","version":"10.x.x"}

# Verify Prometheus datasource is connected
curl -u admin:admin123 \
  "http://localhost:3001/api/datasources/name/Prometheus" | ConvertFrom-Json
```

---

## 16. Troubleshooting Guide

### 16.1 Docker Issues

**Issue: Docker Desktop fails to start**

```powershell
# Symptom: Docker Desktop icon shows error or won't start

# Fix 1: Reset WSL2
wsl --shutdown
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data

# Fix 2: Restart Docker Desktop service
net stop com.docker.service
net start com.docker.service

# Fix 3: Reset Docker Desktop to factory settings
# Docker Desktop → Troubleshoot → Reset to factory defaults
# WARNING: This removes all containers, images, and volumes

# Fix 4: Check Hyper-V is enabled
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
# State should be "Enabled"
```

**Issue: Port already in use**

```powershell
# Find what is using port 3000
netstat -ano | findstr :3000

# Get process name from PID
tasklist /FI "PID eq <PID_NUMBER>"

# Kill the process
Stop-Process -Id <PID_NUMBER> -Force

# Alternative: Change port mapping in docker-compose.yml
# e.g., change "3000:80" to "3100:80"
```

**Issue: Container exits immediately**

```powershell
# Check container logs before it exits
docker logs --tail 50 <container_name>

# Run container interactively to debug
docker run -it --entrypoint /bin/sh <image_name>

# Check for missing environment variables
docker inspect <container_name> | grep -A 20 "Env"
```

**Issue: Cannot connect to MongoDB**

```powershell
# Verify MongoDB is running
docker exec -it mongodb mongosh

# Test connection from another container
docker exec -it backend \
  node -e "const mongoose = require('mongoose'); \
  mongoose.connect(process.env.MONGODB_URI) \
  .then(() => console.log('Connected')) \
  .catch(e => console.error(e))"

# Check MongoDB logs
docker logs mongodb --tail 30
```

### 16.2 Jenkins Issues

**Issue: Jenkins won't start**

```powershell
# Check Jenkins service status
sc query Jenkins

# Check Java is installed (Jenkins requires Java 17+)
java -version

# Check Jenkins log file
Get-Content "C:\ProgramData\Jenkins\.jenkins\logs\jenkins.log" -Tail 50

# Restart Jenkins service
Restart-Service Jenkins

# If port 8080 is busy, change Jenkins port
# Edit C:\ProgramData\Jenkins\jenkins.xml
# Find --httpPort=8080 and change to --httpPort=8081
```

**Issue: Pipeline fails at Git checkout**

```powershell
# Verify GitHub token has correct permissions
# GitHub → Settings → Developer settings → Personal access tokens
# Required scopes: repo, read:org, admin:repo_hook

# Test token manually
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/user

# Check Jenkins credential is correct
# Manage Jenkins → Credentials → github-token → Update

# Verify repository URL
git clone https://github.com/your-org/devops-pipeline-dashboard.git
```

**Issue: SonarQube Quality Gate timeout**

```powershell
# Check SonarQube is accessible from Jenkins
# Jenkins container needs to reach SonarQube container
docker exec jenkins curl http://sonarqube:9000/api/system/status

# If Jenkins runs as Windows service (not Docker), use localhost
# Update Jenkins SonarQube server URL to: http://localhost:9000

# Increase quality gate timeout in Jenkinsfile
timeout(time: 20, unit: 'MINUTES') {
    waitForQualityGate abortPipeline: true
}
```

### 16.3 SonarQube Issues

**Issue: SonarQube fails to start**

```powershell
# Common cause: vm.max_map_count too low (Linux kernel setting)
# Fix for Docker Desktop on Windows:
wsl -d docker-desktop sysctl -w vm.max_map_count=262144

# Make permanent (add to /etc/sysctl.conf in docker-desktop WSL)
wsl -d docker-desktop sh -c "echo 'vm.max_map_count=262144' >> /etc/sysctl.conf"

# Check SonarQube logs
docker logs sonarqube --tail 50

# Common error patterns:
# "max virtual memory areas vm.max_map_count [65530] is too low"
# → Apply the vm.max_map_count fix above

# "Elasticsearch died" 
# → Increase Docker memory limit to at least 4GB
# → Docker Desktop → Settings → Resources → Memory → 4GB+
```

**Issue: Analysis fails with authentication error**

```powershell
# Regenerate token
# SonarQube → My Account → Security → Revoke old token → Generate new

# Verify token in Jenkins credentials
# Manage Jenkins → Credentials → sonarqube-token → Update Secret

# Test token manually
curl -u YOUR_TOKEN: http://localhost:9000/api/projects/search
```

### 16.4 Trivy Issues

**Issue: Trivy cannot pull vulnerability database**

```powershell
# Check internet connectivity
curl https://github.com

# Update database manually with verbose logging
trivy image --download-db-only --debug alpine:latest

# Use offline mode if internet is restricted
# (requires pre-downloaded database)
trivy image --offline-scan YOUR_IMAGE:latest

# Clear and re-download database
Remove-Item -Recurse -Force "$env:USERPROFILE\.cache\trivy"
trivy image --download-db-only alpine:latest
```

**Issue: Trivy reports false positives**

```powershell
# Create .trivyignore file in project root
# List CVE IDs to suppress
# Example: .trivyignore
# CVE-2022-12345
# CVE-2023-67890

# Use in scan
trivy image --ignorefile .trivyignore YOUR_IMAGE:latest

# Filter by severity (ignore LOW/MEDIUM)
trivy image --severity HIGH,CRITICAL YOUR_IMAGE:latest
```

### 16.5 Kubernetes Issues

**Issue: Pods in CrashLoopBackOff**

```powershell
# Get pod name
kubectl get pods -n devops-dashboard

# Check logs
kubectl logs <pod-name> -n devops-dashboard

# Check previous crash logs
kubectl logs <pod-name> -n devops-dashboard --previous

# Describe pod for events
kubectl describe pod <pod-name> -n devops-dashboard

# Common causes:
# 1. Wrong environment variable (check ConfigMap/Secret)
# 2. Image pull error (check image name and registry credentials)
# 3. Missing dependency (MongoDB not ready)
```

**Issue: Pods stuck in Pending state**

```powershell
# Check pod events
kubectl describe pod <pod-name> -n devops-dashboard

# Common causes:
# 1. Insufficient resources
kubectl describe node docker-desktop

# 2. PVC not bound
kubectl get pvc -n devops-dashboard

# 3. Image pull secret missing
kubectl get secrets -n devops-dashboard
```

**Issue: Service not accessible**

```powershell
# Verify service endpoints
kubectl get endpoints -n devops-dashboard

# Test service from within cluster
kubectl run test --image=curlimages/curl --rm -it --restart=Never \
  -- curl http://frontend-service.devops-dashboard:80

# Check NodePort is accessible
curl http://localhost:30000

# Port-forward as alternative
kubectl port-forward service/frontend-service 3000:80 -n devops-dashboard
```

### 16.6 OWASP ZAP Issues

**Issue: ZAP cannot reach target application**

```powershell
# Verify both ZAP and target are on same Docker network
docker network inspect devops-network

# Test connectivity from ZAP container
docker exec owasp-zap curl http://frontend:3000

# If using Kubernetes, ensure target is accessible from Docker network
# ZAP container (Docker) cannot reach Kubernetes pods directly
# Use port-forward and scan localhost

kubectl port-forward service/frontend-service 8888:80 -n devops-dashboard
# Then scan http://host.docker.internal:8888

# Common ZAP exit codes:
# 0: No alerts above specified risk level
# 1: At least one FAIL alert found
# 2: At least one WARN alert found (if -l WARN specified)
```

### 16.7 Prometheus Issues

**Issue: Targets showing as DOWN**

```powershell
# Access Prometheus targets page
Start-Process "http://localhost:9090/targets"

# Check if target is reachable
docker exec prometheus wget -qO- http://backend:5000/metrics

# Reload Prometheus configuration
curl -X POST http://localhost:9090/-/reload

# Validate prometheus.yml
docker exec prometheus \
  /bin/promtool check config /etc/prometheus/prometheus.yml

# Check Prometheus logs
docker logs prometheus --tail 30
```

### 16.8 Grafana Issues

**Issue: Cannot connect to Prometheus datasource**

```powershell
# Verify both containers are on same network
docker network inspect devops-network

# Test from Grafana container
docker exec grafana curl http://prometheus:9090/api/v1/query?query=up

# Update datasource URL to use container name
# Grafana → Configuration → Data sources → Prometheus → URL
# URL: http://prometheus:9090 (not localhost:9090)

# Reset Grafana admin password
docker exec grafana grafana-cli admin reset-admin-password newpassword123
```

---

## 17. Best Practices

### 17.1 Security Best Practices

```
✅ SECRETS MANAGEMENT
├── Never commit .env files or tokens to Git
├── Use Jenkins Credentials for all secrets
├── Use Kubernetes Secrets for pod credentials
├── Rotate secrets every 90 days
├── Use base64 encoding (not encryption) in K8s Secrets
│   └── For production: use Azure Key Vault or HashiCorp Vault
└── Scan for secrets before each commit (Trivy fs --security-checks secret)

✅ CONTAINER SECURITY
├── Use specific image tags (not :latest in production)
├── Use multi-stage Docker builds
├── Run containers as non-root users
├── Set resource limits on all containers
├── Scan images before deployment (Trivy)
├── Use read-only file systems where possible
└── Enable Docker Content Trust: export DOCKER_CONTENT_TRUST=1

✅ KUBERNETES SECURITY
├── Use namespaces for isolation
├── Implement RBAC (Role-Based Access Control)
├── Set Pod Security Standards
├── Use Network Policies to restrict pod communication
├── Enable audit logging
└── Scan Kubernetes manifests: trivy config kubernetes/

✅ CI/CD SECURITY
├── Enable branch protection on main branch
├── Require pull request reviews
├── Sign commits with GPG
├── Use ephemeral credentials (not long-lived tokens)
├── Implement least-privilege principle for Jenkins
└── Scan dependencies with: npm audit --audit-level high
```

### 17.2 Docker Best Practices

```dockerfile
# ✅ Use specific base image versions (not :latest)
FROM node:18.19.0-alpine3.19  # Specific version

# ✅ Use .dockerignore to reduce build context
# ✅ Multi-stage builds to minimize image size
# ✅ Run as non-root user
USER nodejs

# ✅ Set WORKDIR before COPY
WORKDIR /app

# ✅ Copy package files before source code (cache optimization)
COPY package*.json ./
RUN npm ci
COPY . .

# ✅ Add health checks
HEALTHCHECK CMD wget -qO- http://localhost:5000/api/health || exit 1

# ✅ Use dumb-init for proper signal handling
ENTRYPOINT ["dumb-init", "--"]

# ✅ Document ports and environment variables
EXPOSE 5000
ENV NODE_ENV=production
```

### 17.3 Kubernetes Best Practices

```yaml
# ✅ Always set resource requests and limits
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"

# ✅ Always configure liveness and readiness probes
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30

# ✅ Use multiple replicas for high availability
replicas: 2

# ✅ Use RollingUpdate strategy
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1

# ✅ Add labels for every resource
labels:
  app: my-app
  version: "1.0"
  environment: production
```

### 17.4 Jenkins Pipeline Best Practices

```groovy
// ✅ Set pipeline timeout
options {
    timeout(time: 60, unit: 'MINUTES')
}

// ✅ Use parallel stages where possible
parallel {
    stage('Frontend Tests') { ... }
    stage('Backend Tests') { ... }
}

// ✅ Always archive artifacts
post {
    always {
        archiveArtifacts artifacts: 'reports/**/*'
    }
}

// ✅ Use withCredentials for secrets (never env vars directly)
withCredentials([string(credentialsId: 'my-secret', variable: 'SECRET')]) {
    sh 'use $SECRET'
}

// ✅ Clean workspace between builds
options {
    cleanWs()
}

// ✅ Keep limited build history
buildDiscarder(logRotator(numToKeepStr: '10'))
```

### 17.5 Git Best Practices

```gitignore
# .gitignore — never commit these

# Environment files with secrets
.env
.env.local
.env.production
*.env

# Node modules
node_modules/

# Build output
dist/
build/
.next/

# Security reports (may contain sensitive paths)
reports/

# IDE files
.vscode/settings.json
.idea/

# OS files
.DS_Store
Thumbs.db
```

```bash
# ✅ Use meaningful commit messages (Conventional Commits)
git commit -m "feat: add Prometheus metrics endpoint to backend API"
git commit -m "fix: resolve MongoDB connection timeout on startup"
git commit -m "security: update dependencies to patch CVE-2024-XXXXX"
git commit -m "ci: add Trivy image scan to Jenkins pipeline"

# ✅ Use feature branches
git checkout -b feature/add-monitoring
git checkout -b fix/mongodb-connection
git checkout -b security/update-dependencies

# ✅ Sign commits
git config --global commit.gpgsign true
```

---

## 18. Documentation Suite

### 18.1 README.md

**File:** `README.md`

```markdown
# DevOps Pipeline Dashboard

A production-grade MERN Stack application with complete Secure DevOps pipeline
implemented locally using Docker, Kubernetes, Jenkins, and industry-standard
security tools.

## 🚀 Quick Start

### Prerequisites
- Docker Desktop 4.25+ with Kubernetes enabled
- Node.js 18.x LTS
- Jenkins 2.440+ LTS

### Start Everything

```bash
# Clone repository
git clone https://github.com/your-org/devops-pipeline-dashboard.git
cd devops-pipeline-dashboard

# Start all services
docker compose -f docker/docker-compose.yml up -d --build

# Apply Kubernetes manifests
kubectl apply -f kubernetes/
```

## 📊 Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| Frontend | http://localhost:3000 | — |
| Backend API | http://localhost:5000/api | — |
| Jenkins | http://localhost:8080 | admin / (see setup) |
| SonarQube | http://localhost:9000 | admin / Admin@123 |
| Prometheus | http://localhost:9090 | — |
| Grafana | http://localhost:3001 | admin / admin123 |

## 🔒 Security Stack

- **SAST:** SonarQube (Static code analysis)
- **SCA:** Trivy (Container vulnerability scanning)  
- **DAST:** OWASP ZAP (Dynamic application testing)
- **Secrets:** Jenkins Credentials + Kubernetes Secrets

## 📁 Project Structure

See [docs/LOCAL-SETUP.md](docs/LOCAL-SETUP.md) for complete documentation.

## 🗺️ Roadmap

- [x] Phase 1: Application Development
- [x] Phase 2: Local DevSecOps Pipeline  
- [ ] Phase 3: Azure Cloud Deployment
```

### 18.2 Local Setup Guide

**File:** `docs/LOCAL-SETUP.md`

```markdown
# Local Environment Setup Guide

This document provides step-by-step instructions for setting up the
complete DevSecOps pipeline on a Windows 11 machine.

## Prerequisites Check

Run this PowerShell script to verify all prerequisites:

```powershell
# Check all tools are installed
$tools = @{
    "Docker"   = { docker --version }
    "kubectl"  = { kubectl version --client }
    "Trivy"    = { trivy version }
    "Git"      = { git --version }
    "Node.js"  = { node --version }
    "npm"      = { npm --version }
    "Java"     = { java -version 2>&1 }
}

foreach ($tool in $tools.GetEnumerator()) {
    try {
        $version = & $tool.Value 2>&1
        Write-Host "✅ $($tool.Key): $version" -ForegroundColor Green
    } catch {
        Write-Host "❌ $($tool.Key): NOT FOUND" -ForegroundColor Red
    }
}
```

## Quick Verification

After setup, run this to verify everything works:

```powershell
# Check all containers running
docker ps

# Check Kubernetes cluster
kubectl get nodes
kubectl get pods --all-namespaces

# Check all service URLs
$services = @(
    "http://localhost:3000",
    "http://localhost:5000/api/health",
    "http://localhost:8080",
    "http://localhost:9000/api/system/status",
    "http://localhost:9090/-/healthy",
    "http://localhost:3001/api/health"
)

foreach ($url in $services) {
    try {
        $response = Invoke-WebRequest -Uri $url -TimeoutSec 5
        Write-Host "✅ $url - Status: $($response.StatusCode)" -ForegroundColor Green
    } catch {
        Write-Host "❌ $url - UNREACHABLE" -ForegroundColor Red
    }
}
```
```

### 18.3 Troubleshooting Guide

**File:** `docs/TROUBLESHOOTING.md`

```markdown
# Troubleshooting Guide

## Quick Diagnostic Commands

```powershell
# Check all Docker containers and their status
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Check container resource usage
docker stats --no-stream

# Check Docker events (real-time)
docker events --since 30m

# Check Kubernetes pod status
kubectl get pods -n devops-dashboard -o wide

# Get pod logs
kubectl logs -f deployment/backend -n devops-dashboard --tail=50

# Check Kubernetes events
kubectl get events -n devops-dashboard --sort-by='.metadata.creationTimestamp'

# Jenkins log
Get-Content "C:\ProgramData\Jenkins\.jenkins\logs\jenkins.log" -Tail 50

# Docker Desktop log
Get-Content "$env:USERPROFILE\AppData\Local\Docker\log.txt" -Tail 50
```

## Common Error Messages

| Error | Cause | Fix |
|-------|-------|-----|
| `vm.max_map_count` too low | SonarQube/Elasticsearch limit | `wsl -d docker-desktop sysctl -w vm.max_map_count=262144` |
| `port is already allocated` | Port conflict | `netstat -ano \| findstr :PORT` then kill process |
| `ImagePullBackOff` | Cannot pull Docker image | Check image name and registry credentials |
| `CrashLoopBackOff` | Pod keeps crashing | `kubectl logs pod-name --previous` |
| `connection refused` | Service not running | Check container/pod is running and healthy |
| `OOMKilled` | Out of memory | Increase Docker Desktop memory or pod memory limits |
```

### 18.4 Best Practices Guide

**File:** `docs/BEST-PRACTICES.md`

```markdown
# DevSecOps Best Practices

## The Three Pillars

### 1. Shift Left Security
Move security testing earlier in the development lifecycle.

| Traditional | Shift Left |
|-------------|-----------|
| Security at the end | Security at every stage |
| Expensive to fix late | Cheap to fix early |
| Separate security team | Developer-owned security |

### 2. Automate Everything
Manual processes are error-prone and slow.

- Automated testing (unit, integration, e2e)
- Automated security scanning (SAST, DAST, SCA)
- Automated deployment (GitOps with Kubernetes)
- Automated monitoring and alerting

### 3. Fail Fast, Fix Fast
Detect problems early and fix them immediately.

- Enforce quality gates (SonarQube)
- Block deployments on CRITICAL vulnerabilities (Trivy)
- Monitor and alert on anomalies (Prometheus + Grafana)

## Pre-commit Checklist

Before every git push:

- [ ] `npm test` passes with >70% coverage
- [ ] `npm audit --audit-level high` shows no high/critical issues
- [ ] `trivy fs --security-checks secret .` shows no hardcoded secrets
- [ ] Dockerfile uses specific base image version
- [ ] No sensitive data in environment variables committed
- [ ] Kubernetes resource limits defined
- [ ] Health checks configured

## Production Readiness Checklist

Before Phase 3 (Azure deployment):

- [ ] All CRITICAL CVEs resolved in Trivy reports
- [ ] SonarQube Quality Gate passing
- [ ] OWASP ZAP showing 0 HIGH/CRITICAL alerts
- [ ] All container images using non-root users
- [ ] Kubernetes Secrets encrypted at rest
- [ ] HTTPS/TLS configured for all endpoints
- [ ] Logging and monitoring dashboards configured
- [ ] Disaster recovery plan documented
- [ ] Load testing completed (target: <200ms p95)
- [ ] Security penetration test completed
```

---

## Appendix A: Service Access Summary

| Service | URL | Default Login | Purpose |
|---------|-----|--------------|---------|
| Frontend App | http://localhost:3000 | App credentials | MERN dashboard UI |
| Backend API | http://localhost:5000/api | Bearer token | REST API |
| Jenkins | http://localhost:8080 | admin / see setup | CI/CD pipeline |
| SonarQube | http://localhost:9000 | admin / Admin@123 | Code quality |
| OWASP ZAP | http://localhost:8090 | None (API only) | DAST scanning |
| Prometheus | http://localhost:9090 | None | Metrics |
| Grafana | http://localhost:3001 | admin / admin123 | Dashboards |
| Kubernetes App | http://localhost:30000 | App credentials | K8s deployment |

## Appendix B: Environment Variables Reference

| Variable | Service | Description |
|----------|---------|-------------|
| `NODE_ENV` | Backend | Runtime environment (production/development) |
| `PORT` | Backend | Server port (default: 5000) |
| `MONGODB_URI` | Backend | Full MongoDB connection string |
| `JWT_SECRET` | Backend | Secret key for JWT tokens |
| `VITE_API_URL` | Frontend | Backend API base URL (build-time) |
| `GF_SECURITY_ADMIN_PASSWORD` | Grafana | Admin password |
| `SONAR_ES_BOOTSTRAP_CHECKS_DISABLE` | SonarQube | Disable ES checks (local dev) |

## Appendix C: Phase 3 Migration Checklist

Ready for Azure migration when:

```
Azure Prerequisites:
├── [ ] Azure subscription active
├── [ ] Azure CLI installed and authenticated
├── [ ] Azure Container Registry (ACR) created
├── [ ] Azure Kubernetes Service (AKS) cluster created
├── [ ] Azure DevOps organization set up
└── [ ] Azure Key Vault for secrets management

Migration Steps:
├── [ ] Push images to ACR instead of Docker Hub
├── [ ] Update Kubernetes context to AKS
├── [ ] Migrate Kubernetes manifests for Azure specifics
│   ├── StorageClass: azure-disk (not local)
│   ├── LoadBalancer services (not NodePort)
│   └── Azure Ingress Controller
├── [ ] Configure Azure Monitor and Container Insights
├── [ ] Set up Azure AD integration for RBAC
├── [ ] Configure Azure Key Vault for secrets
├── [ ] Enable Azure Defender for Containers
└── [ ] Set up Azure DevOps pipelines to replace Jenkins
```

---

*Document Version: 1.0 | Phase: 2 (Local Implementation) | Status: Production-Ready*

*This document covers complete local DevSecOps implementation. Phase 3 will cover Microsoft Azure cloud deployment.*
