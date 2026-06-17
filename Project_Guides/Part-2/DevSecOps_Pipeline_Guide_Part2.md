# Secure CI/CD Pipeline — Part 2
## Monitoring, Jenkins Pipeline, Security, Troubleshooting, Q&A

---

# 21. PROMETHEUS AND GRAFANA MONITORING

## What is Prometheus?

Prometheus is an open-source **metrics collection and alerting system**. It periodically scrapes (pulls) metrics from your applications and Kubernetes nodes, stores them as time-series data, and lets you query them.

**Analogy:** Prometheus is like a nurse who checks your application's vital signs (CPU, memory, request count, error rate) every 15 seconds and records everything.

## What is Grafana?

Grafana is a **visualization and dashboard tool**. It connects to Prometheus and displays the metrics as beautiful graphs and charts.

**Analogy:** Grafana is the monitor in an ICU — it shows the nurse's recorded data visually so doctors can understand it quickly.

## Install Prometheus and Grafana Using Helm

```bash
# Add Prometheus community Helm chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
# This single Helm chart installs BOTH Prometheus AND Grafana,
# plus pre-configured dashboards for Kubernetes monitoring
helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=7d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.storageClassName=default \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=10Gi \
  --set grafana.adminPassword=Admin@Grafana123 \
  --set grafana.service.type=ClusterIP \
  --set prometheus.service.type=ClusterIP \
  --set alertmanager.enabled=false
```

### Flags Explained

| Flag | Meaning |
|---|---|
| `--namespace monitoring` | Install in 'monitoring' namespace |
| `prometheus.prometheusSpec.retention=7d` | Keep 7 days of metrics (saves disk) |
| `grafana.adminPassword=Admin@Grafana123` | Set Grafana admin password |
| `grafana.service.type=ClusterIP` | Don't expose Grafana publicly (saves cost) |
| `alertmanager.enabled=false` | Disable alertmanager to save resources |

### Check Installation

```bash
kubectl get pods -n monitoring
```

Expected:
```
NAME                                                  READY   STATUS    AGE
prometheus-stack-grafana-xxxx                         3/3     Running   2m
prometheus-stack-kube-prometheus-operator-xxxx        1/1     Running   2m
prometheus-stack-prometheus-0                         2/2     Running   2m
prometheus-stack-kube-state-metrics-xxxx              1/1     Running   2m
prometheus-stack-prometheus-node-exporter-xxxx        1/1     Running   2m
```

## Access Grafana Dashboard

Since Grafana is ClusterIP (not publicly accessible), use port-forwarding:

```bash
# Forward local port 3000 to Grafana pod port 80
kubectl port-forward \
  service/prometheus-stack-grafana \
  3000:80 \
  --namespace monitoring
```

Now open your browser at: `http://localhost:3000`

Login:
- Username: `admin`
- Password: `Admin@Grafana123`

## Pre-Built Dashboards

The kube-prometheus-stack automatically includes these dashboards:

```
Dashboard Name                    | ID     | What It Shows
----------------------------------|--------|----------------------------------
Kubernetes Cluster Overview       | 7249   | Node CPU, memory, pod count
Kubernetes Pod Overview           | 6781   | Per-pod CPU, memory, restarts
Kubernetes Namespace Overview     | 8588   | Namespace resource usage
Node Exporter Full                | 1860   | Host-level CPU, disk, network
```

### Import Additional Dashboard for Node.js

1. In Grafana, click **"+"** → **Import**
2. Enter dashboard ID: `11159` (Node.js Application Dashboard)
3. Click **"Load"**
4. Select "Prometheus" as the data source
5. Click **"Import"**

## prometheus.yml (Custom Configuration)

While Helm installs Prometheus with a good default configuration, here is the prometheus.yml for reference and custom scrape targets:

```yaml
# prometheus.yml
# This is the Prometheus configuration file.
# When using Helm (kube-prometheus-stack), this is configured via Helm values.
# This file is provided for educational understanding.

# ============================================================
# GLOBAL CONFIGURATION
# Settings that apply to all scrape jobs unless overridden
# ============================================================
global:
  scrape_interval: 15s      # How often to pull metrics from targets
  evaluation_interval: 15s  # How often to evaluate alerting rules
  scrape_timeout: 10s       # Timeout for a single scrape request

# ============================================================
# SCRAPE CONFIGURATIONS
# Defines which endpoints to scrape metrics from
# ============================================================
scrape_configs:

  # ---- Prometheus itself ----
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']  # Prometheus scrapes its own metrics

  # ---- Kubernetes Nodes ----
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:   # sd = Service Discovery — automatically finds nodes
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  # ---- Kubernetes Pods ----
  # Scrapes metrics from pods that have the annotation:
  # prometheus.io/scrape: "true"
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with this annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Use the port specified in the annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__

  # ---- Our MERN Backend ----
  - job_name: 'mern-backend'
    static_configs:
      # Scrape metrics from the backend service
      # Backend must expose a /metrics endpoint (use prom-client npm package)
      - targets: ['backend-service.mern-app.svc.cluster.local:5000']
    metrics_path: /metrics   # Path where metrics are exposed

  # ---- Jenkins ----
  - job_name: 'jenkins'
    static_configs:
      - targets: ['<JENKINS-VM-IP>:8080']
    metrics_path: /prometheus
    # Jenkins requires the Prometheus Plugin to be installed
```

## Add Metrics to Your Node.js Backend

To expose metrics from your Node.js backend, add this code to your `server.js`:

```javascript
// Install: npm install prom-client
const client = require('prom-client');

// Collect default Node.js metrics (memory, CPU, event loop, etc.)
const collectDefaultMetrics = client.collectDefaultMetrics;
collectDefaultMetrics({ timeout: 5000 });

// Custom counter: tracks total HTTP requests
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Expose metrics at /metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

---

# 22. COMPLETE JENKINS PIPELINE

## Jenkinsfile

This is the complete Jenkinsfile. Place it in the root of your GitHub repository.

```groovy
// ============================================================
// JENKINSFILE — Complete DevSecOps CI/CD Pipeline
// MERN Stack Application
// Tools: SonarQube, Trivy, OWASP ZAP, Docker, AKS
// ============================================================

pipeline {
    
    // ---- AGENT ----
    // Specifies where Jenkins runs this pipeline.
    // 'any' means: use any available Jenkins agent (the Jenkins server itself).
    agent any
    
    // ---- ENVIRONMENT VARIABLES ----
    // Variables available to all stages.
    // Credentials are securely stored in Jenkins and referenced by ID.
    environment {
        // Azure Container Registry details
        ACR_NAME        = 'merncicdacr'
        ACR_URL         = 'merncicdacr.azurecr.io'
        
        // Docker image names (without tag)
        FRONTEND_IMAGE  = "${ACR_URL}/mern-frontend"
        BACKEND_IMAGE   = "${ACR_URL}/mern-backend"
        
        // Image tag: Jenkins build number ensures unique tags
        IMAGE_TAG       = "${BUILD_NUMBER}"
        
        // Kubernetes configuration
        K8S_NAMESPACE   = 'mern-app'
        
        // SonarQube project key (must match what you created in SonarQube UI)
        SONAR_PROJECT   = 'mern-cicd-project'
        
        // ACR credentials stored in Jenkins Credential Store
        ACR_CREDENTIALS = credentials('acr-credentials')
        // 'credentials()' automatically provides:
        //   ACR_CREDENTIALS_USR = username
        //   ACR_CREDENTIALS_PSW = password
        
        // Azure credentials for AKS deployment
        AZURE_SP_SECRET = credentials('azure-sp-secret')
        AZURE_SP_APPID  = credentials('azure-sp-appid')
        AZURE_TENANT    = credentials('azure-tenant-id')
        
        // SonarQube token
        SONAR_TOKEN     = credentials('sonarqube-token')
        
        // ZAP target URL — the deployed application URL
        ZAP_TARGET_URL  = "http://<INGRESS-EXTERNAL-IP>"
    }
    
    // ---- OPTIONS ----
    // Pipeline-level options
    options {
        // Automatically clean the workspace before each build
        // Prevents old files from affecting the new build
        cleanWs()
        
        // Fail the pipeline if it runs more than 60 minutes
        // Prevents runaway builds consuming resources
        timeout(time: 60, unit: 'MINUTES')
        
        // Keep only the last 10 build logs (saves disk space)
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // Do not run concurrent builds of the same pipeline
        // Prevents race conditions in AKS deployment
        disableConcurrentBuilds()
    }
    
    // ========================================================
    // STAGES: Each stage is a step in the pipeline
    // ========================================================
    stages {
        
        // ====================================================
        // STAGE 1: GIT CHECKOUT
        // Pull the latest code from GitHub
        // ====================================================
        stage('Git Checkout') {
            steps {
                echo '========== STAGE 1: Git Checkout =========='
                
                // Checkout code from GitHub
                // Jenkins automatically uses the branch that triggered the build
                checkout scm
                // 'scm' = Source Control Management configured in the Jenkins job
                
                // Print commit info for traceability
                script {
                    env.GIT_COMMIT_HASH = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    echo "Building commit: ${env.GIT_COMMIT_HASH}"
                    
                    env.GIT_AUTHOR = sh(
                        script: 'git log -1 --pretty=format:"%an"',
                        returnStdout: true
                    ).trim()
                    echo "Commit by: ${env.GIT_AUTHOR}"
                }
            }
        }
        
        // ====================================================
        // STAGE 2: INSTALL FRONTEND DEPENDENCIES
        // Run npm install in the frontend directory
        // ====================================================
        stage('Install Frontend Dependencies') {
            steps {
                echo '========== STAGE 2: Frontend NPM Install =========='
                
                dir('frontend') {
                    // 'dir()' changes the working directory to frontend/
                    
                    // Install dependencies from package-lock.json
                    // --ci is like npm install but:
                    //   - Uses exact versions from lock file
                    //   - Fails if lock file is out of date
                    //   - Faster in CI environments
                    sh 'npm ci --legacy-peer-deps'
                    
                    echo 'Frontend dependencies installed successfully'
                }
            }
        }
        
        // ====================================================
        // STAGE 3: INSTALL BACKEND DEPENDENCIES
        // ====================================================
        stage('Install Backend Dependencies') {
            steps {
                echo '========== STAGE 3: Backend NPM Install =========='
                
                dir('backend') {
                    sh 'npm ci --legacy-peer-deps'
                    echo 'Backend dependencies installed successfully'
                }
            }
        }
        
        // ====================================================
        // STAGE 4: BUILD REACT FRONTEND
        // Creates optimized production build
        // ====================================================
        stage('Build Frontend') {
            steps {
                echo '========== STAGE 4: React Production Build =========='
                
                dir('frontend') {
                    // Create optimized production build
                    // Outputs to frontend/build directory
                    // This build is then copied into the Docker image
                    sh 'npm run build'
                    
                    echo 'React frontend built successfully'
                    
                    // Archive the build directory size for reference
                    sh 'du -sh build/'
                }
            }
        }
        
        // ====================================================
        // STAGE 5: BACKEND SECURITY AUDIT
        // Check for known vulnerabilities in npm packages
        // ====================================================
        stage('Backend NPM Audit') {
            steps {
                echo '========== STAGE 5: NPM Security Audit =========='
                
                dir('backend') {
                    // npm audit checks your dependencies against the NPM security database
                    // --audit-level=high: fail only for high/critical severity issues
                    // || true: don't fail the build for medium/low issues (for demo)
                    // Remove '|| true' in real projects for stricter security
                    sh 'npm audit --audit-level=high || true'
                    
                    // Generate audit report in JSON for archiving
                    sh 'npm audit --json > npm-audit-report.json || true'
                }
                
                dir('frontend') {
                    sh 'npm audit --audit-level=high || true'
                    sh 'npm audit --json > npm-audit-report.json || true'
                }
            }
        }
        
        // ====================================================
        // STAGE 6: SONARQUBE STATIC ANALYSIS (SAST)
        // Scan source code for security vulnerabilities and bugs
        // ====================================================
        stage('SonarQube Analysis') {
            steps {
                echo '========== STAGE 6: SonarQube SAST Scan =========='
                
                // withSonarQubeEnv wraps the block with SonarQube environment variables
                // 'SonarQube' must match the server name configured in Jenkins settings
                withSonarQubeEnv('SonarQube') {
                    
                    // Run SonarQube Scanner
                    // -Dsonar.* are SonarQube analysis parameters
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT} \
                        -Dsonar.projectName='MERN CICD Project' \
                        -Dsonar.projectVersion=${BUILD_NUMBER} \
                        -Dsonar.sources=frontend/src,backend/src \
                        -Dsonar.exclusions=**/node_modules/**,**/build/**,**/*.test.js \
                        -Dsonar.javascript.lcov.reportPaths=frontend/coverage/lcov.info \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                    // -Dsonar.sources: directories to scan
                    // -Dsonar.exclusions: files/dirs to skip
                    // -Dsonar.login: authentication token
                }
            }
        }
        
        // ====================================================
        // STAGE 7: SONARQUBE QUALITY GATE
        // Check if code meets the quality standards we defined
        // Pipeline FAILS if quality gate fails
        // ====================================================
        stage('SonarQube Quality Gate') {
            steps {
                echo '========== STAGE 7: SonarQube Quality Gate =========='
                
                script {
                    // Wait for SonarQube to finish analysis and return result
                    // timeout: wait maximum 5 minutes for result
                    timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        // waitForQualityGate() polls SonarQube until analysis completes
                        
                        if (qualityGate.status != 'OK') {
                            // Quality gate FAILED — stop the pipeline
                            error "❌ SonarQube Quality Gate FAILED: ${qualityGate.status}"
                            // The pipeline stops here — no Docker build, no deployment
                        } else {
                            echo "✅ SonarQube Quality Gate PASSED: ${qualityGate.status}"
                        }
                    }
                }
            }
        }
        
        // ====================================================
        // STAGE 8: BUILD DOCKER IMAGES
        // Create Docker images for frontend and backend
        // ====================================================
        stage('Build Docker Images') {
            steps {
                echo '========== STAGE 8: Docker Image Build =========='
                
                script {
                    // Login to Azure Container Registry
                    sh """
                        echo ${ACR_CREDENTIALS_PSW} | docker login \
                            ${ACR_URL} \
                            --username ${ACR_CREDENTIALS_USR} \
                            --password-stdin
                    """
                    // '--password-stdin' reads password from stdin (more secure than -p flag)
                    
                    // Build frontend image with TWO tags:
                    // 1. :latest - always points to newest build
                    // 2. :BUILD_NUMBER - immutable tag for this specific build
                    sh """
                        docker build \
                            -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                            -t ${FRONTEND_IMAGE}:latest \
                            --build-arg BUILD_DATE=\$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                            --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                            --build-arg GIT_COMMIT=${GIT_COMMIT_HASH} \
                            ./frontend
                    """
                    // --build-arg: Pass build-time arguments into the Dockerfile
                    // These can be used for labels/metadata in the image
                    
                    // Build backend image
                    sh """
                        docker build \
                            -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                            -t ${BACKEND_IMAGE}:latest \
                            --build-arg BUILD_DATE=\$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                            --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                            --build-arg GIT_COMMIT=${GIT_COMMIT_HASH} \
                            ./backend
                    """
                    
                    echo "Frontend image: ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    echo "Backend image: ${BACKEND_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
        
        // ====================================================
        // STAGE 9: TRIVY FILESYSTEM SCAN
        // Scan the source code and dependencies for vulnerabilities
        // This runs BEFORE pushing to ACR — catches issues early
        // ====================================================
        stage('Trivy Filesystem Scan') {
            steps {
                echo '========== STAGE 9: Trivy Filesystem Vulnerability Scan =========='
                
                script {
                    // Scan the entire project directory
                    // --format table: human-readable output
                    // --severity HIGH,CRITICAL: only show important vulnerabilities
                    // --exit-code 0: don't fail build even if vulnerabilities found (for demo)
                    //   Change to --exit-code 1 for strict security enforcement
                    sh """
                        trivy fs \
                            --format table \
                            --severity HIGH,CRITICAL \
                            --exit-code 0 \
                            --output trivy-fs-report.txt \
                            .
                    """
                    
                    // Also generate JSON report for archiving
                    sh """
                        trivy fs \
                            --format json \
                            --severity HIGH,CRITICAL \
                            --output trivy-fs-report.json \
                            .
                    """
                    
                    echo 'Trivy filesystem scan complete. Check trivy-fs-report.txt'
                }
            }
        }
        
        // ====================================================
        // STAGE 10: TRIVY IMAGE SCAN
        // Scan Docker images for OS-level and library vulnerabilities
        // ====================================================
        stage('Trivy Image Scan') {
            steps {
                echo '========== STAGE 10: Trivy Docker Image Scan =========='
                
                script {
                    // Scan frontend image
                    sh """
                        trivy image \
                            --format table \
                            --severity HIGH,CRITICAL \
                            --exit-code 0 \
                            --output trivy-frontend-image-report.txt \
                            ${FRONTEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    // Scan backend image
                    sh """
                        trivy image \
                            --format table \
                            --severity HIGH,CRITICAL \
                            --exit-code 0 \
                            --output trivy-backend-image-report.txt \
                            ${BACKEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    // Generate JSON reports
                    sh """
                        trivy image \
                            --format json \
                            --severity HIGH,CRITICAL \
                            --output trivy-frontend-image-report.json \
                            ${FRONTEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    sh """
                        trivy image \
                            --format json \
                            --severity HIGH,CRITICAL \
                            --output trivy-backend-image-report.json \
                            ${BACKEND_IMAGE}:${IMAGE_TAG}
                    """
                    
                    echo 'Trivy image scans complete'
                }
            }
        }
        
        // ====================================================
        // STAGE 11: PUSH IMAGES TO ACR
        // Upload built and scanned images to Azure Container Registry
        // ====================================================
        stage('Push Images to ACR') {
            steps {
                echo '========== STAGE 11: Push Docker Images to ACR =========='
                
                script {
                    // Push frontend image with both tags
                    sh "docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${FRONTEND_IMAGE}:latest"
                    
                    // Push backend image with both tags
                    sh "docker push ${BACKEND_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${BACKEND_IMAGE}:latest"
                    
                    echo "Images pushed to ACR successfully"
                    echo "Frontend: ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                    echo "Backend:  ${BACKEND_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
        
        // ====================================================
        // STAGE 12: DEPLOY TO AKS
        // Apply Kubernetes manifests to deploy updated images
        // ====================================================
        stage('Deploy to AKS') {
            steps {
                echo '========== STAGE 12: Kubernetes Deployment =========='
                
                script {
                    // Step 1: Login to Azure using Service Principal
                    // A Service Principal is like a service account for Azure CLI
                    sh """
                        az login \
                            --service-principal \
                            --username ${AZURE_SP_APPID} \
                            --password ${AZURE_SP_SECRET} \
                            --tenant ${AZURE_TENANT}
                    """
                    
                    // Step 2: Get AKS credentials (download kubeconfig)
                    sh """
                        az aks get-credentials \
                            --resource-group mern-cicd-rg \
                            --name mern-cicd-aks \
                            --overwrite-existing
                    """
                    
                    // Step 3: Apply ConfigMap and Secret (in case they changed)
                    sh "kubectl apply -f k8s/configmap.yaml"
                    sh "kubectl apply -f k8s/secret.yaml"
                    
                    // Step 4: Update the image tag in deployments
                    // kubectl set image: updates the container image in a deployment
                    // This triggers a rolling update automatically
                    sh """
                        kubectl set image deployment/frontend-deployment \
                            frontend-container=${FRONTEND_IMAGE}:${IMAGE_TAG} \
                            --namespace=${K8S_NAMESPACE}
                    """
                    
                    sh """
                        kubectl set image deployment/backend-deployment \
                            backend-container=${BACKEND_IMAGE}:${IMAGE_TAG} \
                            --namespace=${K8S_NAMESPACE}
                    """
                    
                    // Step 5: Wait for deployments to complete
                    // --timeout=5m: give it up to 5 minutes to rollout
                    sh """
                        kubectl rollout status deployment/frontend-deployment \
                            --namespace=${K8S_NAMESPACE} \
                            --timeout=5m
                    """
                    
                    sh """
                        kubectl rollout status deployment/backend-deployment \
                            --namespace=${K8S_NAMESPACE} \
                            --timeout=5m
                    """
                    
                    echo "Deployment to AKS completed successfully"
                }
            }
        }
        
        // ====================================================
        // STAGE 13: OWASP ZAP DAST SCAN
        // Dynamically test the deployed application for vulnerabilities
        // This runs AGAINST the live deployed application
        // ====================================================
        stage('OWASP ZAP Security Scan') {
            steps {
                echo '========== STAGE 13: OWASP ZAP DAST Scan =========='
                
                script {
                    // Run ZAP baseline scan against the deployed application
                    // Baseline scan: passive scan (no active attacks)
                    // Full scan: active attack simulation (more thorough, more time)
                    
                    // Create a directory for ZAP reports
                    sh 'mkdir -p zap-reports'
                    
                    // Run ZAP baseline scan in a Docker container
                    sh """
                        docker run --rm \
                            --network host \
                            -v \$(pwd)/zap-reports:/zap/wrk/:rw \
                            ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
                            -t ${ZAP_TARGET_URL} \
                            -r zap-report.html \
                            -J zap-report.json \
                            -x zap-report.xml \
                            -I
                    """
                    // -t: target URL to scan
                    // -r: HTML report filename
                    // -J: JSON report filename
                    // -x: XML report filename
                    // -I: do not fail on warnings (use -I for demo; remove for production)
                    
                    echo 'ZAP scan complete. Reports saved in zap-reports/'
                }
            }
        }
        
        // ====================================================
        // STAGE 14: DEPLOYMENT VERIFICATION
        // Verify the deployed application is healthy
        // ====================================================
        stage('Deployment Verification') {
            steps {
                echo '========== STAGE 14: Deployment Verification =========='
                
                script {
                    // Check pod status
                    sh """
                        kubectl get pods \
                            --namespace=${K8S_NAMESPACE} \
                            -o wide
                    """
                    
                    // Verify all pods are running (not CrashLooping)
                    sh """
                        kubectl get deployments \
                            --namespace=${K8S_NAMESPACE}
                    """
                    
                    // Check services
                    sh """
                        kubectl get services \
                            --namespace=${K8S_NAMESPACE}
                    """
                    
                    // Check ingress
                    sh """
                        kubectl get ingress \
                            --namespace=${K8S_NAMESPACE}
                    """
                    
                    // Test application health endpoint
                    // Give the app 30 seconds to stabilize after deployment
                    sh 'sleep 30'
                    
                    // Check if the frontend is responding
                    sh """
                        curl -f ${ZAP_TARGET_URL} \
                            --max-time 30 \
                            --silent \
                            --output /dev/null \
                            -w "HTTP Status: %{http_code}" \
                        || echo 'Warning: Frontend health check failed'
                    """
                    
                    // Check backend health endpoint
                    sh """
                        curl -f ${ZAP_TARGET_URL}/api/health \
                            --max-time 30 \
                            --silent \
                            -w "Backend HTTP Status: %{http_code}" \
                        || echo 'Warning: Backend health check failed'
                    """
                    
                    echo "✅ Deployment verification complete"
                }
            }
        }
        
        // ====================================================
        // STAGE 15: ARCHIVE SECURITY REPORTS
        // Save all security reports as Jenkins build artifacts
        // Can be downloaded from the Jenkins UI
        // ====================================================
        stage('Archive Security Reports') {
            steps {
                echo '========== STAGE 15: Archiving Security Reports =========='
                
                // archiveArtifacts: saves files as downloadable artifacts in Jenkins
                archiveArtifacts artifacts: 'trivy-*.txt', allowEmptyArchive: true
                archiveArtifacts artifacts: 'trivy-*.json', allowEmptyArchive: true
                archiveArtifacts artifacts: 'zap-reports/**', allowEmptyArchive: true
                archiveArtifacts artifacts: 'backend/npm-audit-report.json', allowEmptyArchive: true
                archiveArtifacts artifacts: 'frontend/npm-audit-report.json', allowEmptyArchive: true
                
                // Publish ZAP HTML report as a Jenkins build page
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'zap-reports',
                    reportFiles: 'zap-report.html',
                    reportName: 'OWASP ZAP Security Report'
                ])
                
                echo "All security reports archived"
            }
        }
    }
    
    // ========================================================
    // POST: Actions to run after all stages complete
    // These run regardless of whether the pipeline succeeded or failed
    // ========================================================
    post {
        
        // ---- SUCCESS ----
        success {
            echo """
            ╔═══════════════════════════════════════╗
            ║  ✅ PIPELINE SUCCEEDED                 ║
            ║  Build: #${BUILD_NUMBER}               ║
            ║  Commit: ${GIT_COMMIT_HASH}            ║
            ║  Images pushed to ACR                  ║
            ║  Application deployed to AKS           ║
            ╚═══════════════════════════════════════╝
            """
        }
        
        // ---- FAILURE ----
        failure {
            echo """
            ╔═══════════════════════════════════════╗
            ║  ❌ PIPELINE FAILED                   ║
            ║  Build: #${BUILD_NUMBER}               ║
            ║  Check the logs above for errors       ║
            ╚═══════════════════════════════════════╝
            """
        }
        
        // ---- ALWAYS ----
        always {
            echo 'Pipeline execution complete'
            
            // Clean up dangling Docker images to free disk space
            sh 'docker image prune -f || true'
            
            // Remove the sensitive kubeconfig
            sh 'rm -f ~/.kube/config || true'
        }
    }
}
```

---

# 23. SECURITY REPORTS AND VERIFICATION

## SonarQube Report Summary

After each pipeline run, view the SonarQube report at:
```
http://<JENKINS-VM-IP>:9000/dashboard?id=mern-cicd-project
```

The report shows:

```
Security Hotspots:   Issues that need manual review
Vulnerabilities:     Confirmed security flaws (A-E rating)
Bugs:                Code that will cause runtime errors
Code Smells:         Maintainability issues
Coverage:            What % of code has unit tests
Duplications:        Repeated code blocks
```

### What Good Results Look Like

```
Security Rating:     A (zero vulnerabilities)
Reliability Rating:  A or B (few or no bugs)
Maintainability:     A (low technical debt)
Coverage:            > 70%
Quality Gate:        Passed ✅
```

## Trivy Vulnerability Report

Trivy output format:
```
2024-01-01 00:00:00 INFO Vulnerability scanning is enabled
2024-01-01 00:00:00 INFO Secret scanning is enabled

merncicdacr.azurecr.io/mern-frontend:42 (alpine 3.19.1)
================================================
Total: 0 (HIGH: 0, CRITICAL: 0)

Node.js packages (package-lock.json)
======================================
┌─────────────────┬────────────────┬──────────┬───────────────────┬─────────────────────────────┐
│    Library      │ Vulnerability  │ Severity │  Installed Version│    Fixed Version             │
├─────────────────┼────────────────┼──────────┼───────────────────┼─────────────────────────────┤
│ lodash          │ CVE-2021-23337 │ HIGH     │      4.17.20      │     4.17.21                 │
└─────────────────┴────────────────┴──────────┴───────────────────┴─────────────────────────────┘
```

**Remediation:** Update lodash: `npm install lodash@4.17.21`

## OWASP ZAP Report

ZAP generates a detailed HTML report with:

```
Risk Level     | Alert Name                  | Confidence | Description
---------------|-----------------------------|-----------|-----------------
High           | SQL Injection               | Medium     | Possible SQL...
Medium         | Missing CSP Header          | High       | No Content-...
Low            | X-Frame-Options Not Set     | High       | Clickjacking...
Informational  | Session Management          | Medium     | Session token...
```

**Common Fixes:**

| Alert | Fix |
|---|---|
| Missing CSP Header | Add `Content-Security-Policy` header in Nginx config |
| X-Frame-Options | Already added in nginx.conf (`X-Frame-Options SAMEORIGIN`) |
| SQL Injection (Node.js) | Use parameterized queries / Mongoose ODM (auto-sanitized) |
| Missing HTTPS | Add TLS/SSL certificate (Let's Encrypt) |

---

# 24. TROUBLESHOOTING GUIDE

## Problem: `az: command not found`

**Cause:** Azure CLI not in PATH or not installed.

**Fix:**
```bash
# Check if installed
which az

# If not found, reinstall
sudo apt-get update && sudo apt-get install -y azure-cli

# Add to PATH
export PATH=$PATH:/usr/local/bin
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## Problem: `UNAUTHORIZED: authentication required` when pushing to ACR

**Cause:** Not logged in to ACR, or credentials expired.

**Fix:**
```bash
# Login to ACR
az acr login --name merncicdacr

# If that fails, use credentials directly
docker login merncicdacr.azurecr.io \
  --username merncicdacr \
  --password <ACR-PASSWORD>
```

---

## Problem: AKS Pods stuck in `Pending` state

**Cause:** Usually insufficient resources on the node.

**Diagnosis:**
```bash
# Describe the pending pod to see the reason
kubectl describe pod <POD-NAME> -n mern-app

# Look for "Events" section at the bottom
# Common message: "0/1 nodes are available: 1 Insufficient memory"
```

**Fix:**
```bash
# Option 1: Scale down other pods to free resources
kubectl scale deployment backend-deployment --replicas=1 -n mern-app

# Option 2: Check node resources
kubectl describe node <NODE-NAME>

# Option 3: Reduce resource requests in deployment.yaml
# Change: memory: "128Mi" to memory: "64Mi"
```

---

## Problem: Pods in `CrashLoopBackOff`

**Cause:** Container starts then crashes repeatedly. Usually a code error.

**Diagnosis:**
```bash
# View container logs (the crash reason)
kubectl logs <POD-NAME> -n mern-app

# View previous container logs (from before the crash)
kubectl logs <POD-NAME> -n mern-app --previous

# Describe the pod
kubectl describe pod <POD-NAME> -n mern-app
```

**Common causes:**
- Wrong MongoDB URI (cannot connect to database)
- Missing environment variable
- Port conflict
- Missing `npm start` script

---

## Problem: Jenkins pipeline fails at SonarQube Quality Gate

**Cause:** Code quality metrics do not meet the gate thresholds.

**Fix:**
```bash
# Option 1: Lower the quality gate thresholds in SonarQube
# Go to: SonarQube → Quality Gates → Edit your gate

# Option 2: Temporarily use the built-in "Sonar way" gate (more lenient)

# Option 3: Add sonar.qualitygate.wait=false to skip the gate
# (NOT recommended for production — defeats the purpose)
```

---

## Problem: `ImagePullBackOff` on Kubernetes pods

**Cause:** Kubernetes cannot pull the Docker image from ACR.

**Diagnosis:**
```bash
kubectl describe pod <POD-NAME> -n mern-app
# Look for: "Failed to pull image ... unauthorized"
```

**Fix:**
```bash
# Recreate the ACR pull secret
kubectl delete secret acr-secret -n mern-app
kubectl create secret docker-registry acr-secret \
  --docker-server=merncicdacr.azurecr.io \
  --docker-username=merncicdacr \
  --docker-password=<ACR-PASSWORD> \
  --namespace=mern-app
```

---

## Problem: Ingress shows `<pending>` for EXTERNAL-IP

**Cause:** Azure Load Balancer is being provisioned (takes 3-5 minutes) OR quota exceeded.

**Fix:**
```bash
# Wait and check again after 5 minutes
kubectl get service ingress-nginx-controller -n ingress-nginx --watch

# If still pending after 10 minutes, check Azure subscription quota
az network lb list --resource-group MC_mern-cicd-rg_mern-cicd-aks_eastus
```

---

## Problem: Docker build fails — `npm ERR! Cannot find module`

**Cause:** `node_modules` is not being copied correctly OR dependencies mismatch.

**Fix:**
```bash
# Clean Docker build cache and rebuild
docker build --no-cache -t mern-frontend ./frontend
```

---

## Problem: ZAP scan fails — `Connection refused`

**Cause:** Application is not accessible at the ZAP_TARGET_URL.

**Fix:**
```bash
# Verify the Ingress IP
kubectl get ingress -n mern-app

# Test the URL manually
curl -v http://<INGRESS-IP>

# Update ZAP_TARGET_URL in Jenkinsfile with the correct IP
```

---

# 25. AZURE COST OPTIMIZATION

## Techniques to Stay Within $100

### 1. Stop AKS When Not Using It

```bash
# Stop all node VMs (saves ~$1.15/day per node)
az aks stop \
  --name mern-cicd-aks \
  --resource-group mern-cicd-rg

# Start when needed
az aks start \
  --name mern-cicd-aks \
  --resource-group mern-cicd-rg
```

> NOTE: When AKS is stopped, you still pay for the managed disk (~$0.05/day) and public IP (~$0.004/hour). But VM compute costs stop.

### 2. Stop Jenkins VM Overnight

```bash
# Stop Jenkins VM (saves $0.048/hour when idle)
az vm deallocate \
  --resource-group mern-cicd-rg \
  --name jenkins-vm

# Start when needed
az vm start \
  --resource-group mern-cicd-rg \
  --name jenkins-vm
```

> IMPORTANT: Use `deallocate`, not `stop`. `stop` still charges for the VM. `deallocate` stops billing.

### 3. Use Azure Budget Alerts

```bash
# Set a spending alert at $80 (warns you before you exceed budget)
az consumption budget create \
  --budget-name "StudentBudget" \
  --amount 80 \
  --category Cost \
  --time-grain Monthly \
  --start-date 2024-01-01 \
  --end-date 2024-12-31 \
  --resource-group mern-cicd-rg
```

### 4. Delete Everything After Demonstration

```bash
# THE NUCLEAR OPTION — deletes EVERYTHING in one command
# Do this after your project viva
az group delete \
  --name mern-cicd-rg \
  --yes \
  --no-wait

# Verify deletion
az group exists --name mern-cicd-rg
# Should return: false
```

### 5. Use Basic SKUs Everywhere

| Service | Basic SKU | Standard SKU | Savings |
|---|---|---|---|
| ACR | $5/month | $20/month | $15/month |
| Load Balancer | $3.65/month | $18/month | $14.35/month |
| Public IP | Static Basic free | Static Standard $3/month | $3/month |

### 6. Reduce Replica Count for Demo

```bash
# For demo: reduce to 1 replica (saves memory, allows cheaper node)
kubectl scale deployment frontend-deployment --replicas=1 -n mern-app
kubectl scale deployment backend-deployment --replicas=1 -n mern-app
```

### 7. Cost Tracking During Development

```bash
# Check current spending
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --output table

# Or use the Azure Portal: Cost Management + Billing
```

## Final Cost Estimate (7-Day Demo Scenario)

```
Resource                    Duration    Cost
-----------------------------------------
AKS (1x B2s)               7 days      $8.05
Jenkins VM (active 8h/day)  7 days      $2.69
ACR                        7 days      $1.17
Load Balancer              7 days      $0.85
Public IPs                 7 days      $0.56
-----------------------------------------
TOTAL (7-day demo)                     $13.32
REMAINING CREDITS:                     $86.68
```

---

# 26. VIVA QUESTIONS AND ANSWERS

## Section A: CI/CD Fundamentals

**Q1: What is CI/CD and why is it used?**

**A:** CI (Continuous Integration) is the practice of automatically building and testing code every time a developer commits changes. CD (Continuous Delivery/Deployment) extends this by automatically deploying the tested code to production. It is used to:
- Detect bugs early (in minutes, not days)
- Reduce manual deployment errors
- Enable faster, more frequent releases
- Improve team collaboration
- Provide audit trail of all deployments

**Q2: What is the difference between Continuous Delivery and Continuous Deployment?**

**A:** Continuous Delivery means the pipeline automatically deploys to a staging environment, but a human approves the final production deployment. Continuous Deployment means everything is fully automated — code goes to production automatically after all tests pass. Our project implements Continuous Deployment since the pipeline deploys to AKS automatically.

**Q3: What is Jenkins and what role does it play?**

**A:** Jenkins is an open-source automation server. In our pipeline, Jenkins acts as the orchestrator — it listens for GitHub webhooks, triggers the pipeline on every code push, executes each stage (build, test, scan, deploy), and reports the result. Without Jenkins, all these steps would need to be done manually.

---

## Section B: Docker and Containerization

**Q4: What is Docker and what problem does it solve?**

**A:** Docker is a containerization platform that packages an application with all its dependencies into a container. The problem it solves is "it works on my machine" — before Docker, code that worked on a developer's laptop might fail in production due to different OS versions, library versions, or configurations. Docker containers run identically everywhere.

**Q5: What is the difference between a Docker image and a Docker container?**

**A:** A Docker image is a read-only blueprint (like a class in OOP). A Docker container is a running instance of that image (like an object created from a class). Multiple containers can be created from the same image, and each runs independently.

**Q6: What is a multi-stage Docker build and why did you use it?**

**A:** A multi-stage build uses multiple `FROM` statements in one Dockerfile. The first stage (builder) compiles the code. The second stage (production) copies only the compiled output, not the build tools or source code. We used it because:
- The React build requires Node.js, webpack, etc. (hundreds of MB of tools)
- But the final output is just HTML/CSS/JS files, which only needs Nginx
- Multi-stage build: final image is ~23MB instead of ~900MB

**Q7: What is the purpose of .dockerignore?**

**A:** `.dockerignore` tells Docker which files to exclude from the build context (the files sent to the Docker daemon). We exclude `node_modules/` because it's platform-specific and reinstalled inside the container. We exclude `.env` files because they contain sensitive data that should not be baked into Docker images.

---

## Section C: Kubernetes

**Q8: What is Kubernetes and why is it needed?**

**A:** Kubernetes (K8s) is a container orchestration platform. When you have Docker containers, Kubernetes:
- Ensures your desired number of replicas are always running
- Automatically restarts crashed containers
- Distributes traffic across multiple pods (load balancing)
- Rolls out updates without downtime
- Scales automatically based on load
- Manages secrets and configuration

**Q9: What is the difference between a Pod, Deployment, and Service?**

**A:** 
- **Pod**: The smallest unit in Kubernetes; runs one or more containers. Ephemeral — can be killed/replaced.
- **Deployment**: Manages a set of identical pods. Ensures desired count is running. Handles rolling updates.
- **Service**: Provides a stable network endpoint (DNS name + IP) to access pods. Since pod IPs change, the Service gives a consistent address.

**Q10: What are liveness and readiness probes?**

**A:** 
- **Liveness probe**: Checks if the container is alive (not deadlocked). If it fails, Kubernetes KILLS and RESTARTS the container.
- **Readiness probe**: Checks if the container is ready to receive traffic. If it fails, the pod is removed from the Service's endpoint list — no traffic is sent to it, but it is NOT restarted.
- In our deployment, both probes hit the `/health` endpoint on port 5000.

**Q11: What is a Rolling Update strategy?**

**A:** Rolling Update deploys new pods gradually while keeping old pods running, ensuring zero downtime. With our settings:
- `maxSurge: 1`: Creates 1 extra pod before removing old ones
- `maxUnavailable: 0`: Never has fewer pods than desired during update
- Result: Deployment goes from 2 old → 3 (2 old + 1 new) → 2 (1 old + 1 new) → 2 (2 new)

**Q12: What is the difference between ConfigMap and Secret?**

**A:** Both store configuration data for pods. ConfigMaps store non-sensitive data (URLs, port numbers, feature flags). Secrets store sensitive data (passwords, tokens, certificates) in base64 encoding. In production, Secrets should be backed by a proper secrets manager like Azure Key Vault (not just base64 which is not encryption).

---

## Section D: Security (DevSecOps)

**Q13: What is DevSecOps?**

**A:** DevSecOps is the practice of integrating security throughout the CI/CD pipeline, not just at the end. "Shift left security" — security checks happen early (at code commit), not after deployment. Our pipeline integrates:
- SAST (SonarQube) — at code level
- Dependency scanning (npm audit) — at build level
- Image scanning (Trivy) — at container level
- DAST (OWASP ZAP) — at runtime level

**Q14: What is SAST vs DAST?**

**A:**
- **SAST (Static Application Security Testing)**: Analyzes source code without running it. Like proofreading a document. Tool: SonarQube. Finds: SQL injection patterns, hardcoded passwords, insecure functions.
- **DAST (Dynamic Application Security Testing)**: Tests the running application by sending real HTTP requests. Like actually trying to break into a building. Tool: OWASP ZAP. Finds: runtime XSS, authentication bypasses, actual SQL injection.

**Q15: What is Trivy and what does it scan?**

**A:** Trivy is an open-source vulnerability scanner that scans:
1. Docker images for known CVEs in OS packages (Alpine, Ubuntu packages)
2. Application dependencies for known CVEs (npm packages, pip packages)
3. Kubernetes YAML files for security misconfigurations
4. Filesystems for exposed secrets (API keys, private keys)

**Q16: What is a CVE?**

**A:** CVE (Common Vulnerabilities and Exposures) is a standardized list of publicly known software vulnerabilities. Each CVE has an ID (e.g., CVE-2021-44228 = Log4Shell), a description, and a severity score (0-10). Trivy checks your dependencies against the CVE database to identify if you're using software with known security flaws.

**Q17: What is the SonarQube Quality Gate?**

**A:** A Quality Gate is a set of conditions that code must meet to proceed through the pipeline. If code has more vulnerabilities than allowed, or less than the minimum test coverage, the Quality Gate fails and the pipeline stops. This enforces security standards on every commit and prevents vulnerable code from being deployed.

---

## Section E: Azure and Cloud

**Q18: What is ACR and why did you use it instead of Docker Hub?**

**A:** Azure Container Registry (ACR) is a private Docker registry hosted on Azure. We used ACR because:
- Images are private (unlike public Docker Hub repositories)
- Integrated with AKS — no separate authentication needed
- Same Azure region = faster pulls (lower latency)
- Azure RBAC controls who can push/pull
- Built-in vulnerability scanning available

**Q19: What is AKS?**

**A:** Azure Kubernetes Service (AKS) is a fully managed Kubernetes cluster. "Managed" means Microsoft handles the Kubernetes control plane (API server, etcd, scheduler) for free. We only pay for the worker node VMs where our containers actually run. We chose Standard_B2s VMs because they are the cheapest burstable option that can run our MERN application.

**Q20: What is a Resource Group in Azure?**

**A:** A Resource Group is a logical container for Azure resources. All resources in our project (ACR, AKS, VMs, Networks) are inside the `mern-cicd-rg` resource group. Benefits:
- Delete all resources at once by deleting the group
- Apply access policies to all resources at once
- View all costs for the project in one place

---

# 27. INTERVIEW QUESTIONS AND ANSWERS

## DevOps Core Concepts

**Q: "Walk me through your CI/CD pipeline."**

**A:** "Our pipeline begins when a developer pushes code to GitHub. This triggers a webhook that notifies Jenkins. Jenkins then:

1. **Checks out** the latest code
2. Runs **npm install** for both frontend and backend
3. **Builds** the React production bundle using `npm run build`
4. Runs **SonarQube** for static analysis — if the Quality Gate fails, the pipeline stops here
5. **Builds Docker images** for both services with the build number as tag
6. Runs **Trivy** to scan the images for CVE vulnerabilities
7. **Pushes** the images to Azure Container Registry
8. **Deploys** to AKS using `kubectl set image` for zero-downtime rolling update
9. Runs **OWASP ZAP** against the live deployed application
10. Archives all security reports as Jenkins artifacts

The entire pipeline takes about 15-20 minutes. If any stage fails, the pipeline stops and notifies the team."

---

**Q: "What security tools did you use and why?"**

**A:** "We implemented a three-layer security approach:

**Layer 1 — Code (SAST):** SonarQube scans source code for security anti-patterns like potential SQL injection, hardcoded credentials, and insecure function calls. It enforces a Quality Gate that prevents vulnerable code from proceeding.

**Layer 2 — Container (Image Scanning):** Trivy scans our Docker images for known CVEs in both the OS packages and npm dependencies. This catches vulnerabilities in third-party libraries we're using.

**Layer 3 — Runtime (DAST):** OWASP ZAP runs against our deployed application, simulating real HTTP-level attacks. It finds runtime vulnerabilities that only manifest when the application is actually running.

Each layer catches different types of vulnerabilities, so together they provide comprehensive security coverage."

---

**Q: "How does Kubernetes achieve zero-downtime deployment?"**

**A:** "Kubernetes uses a Rolling Update strategy. When we update the image tag in our deployment, Kubernetes doesn't kill all old pods at once. Instead, it creates one new pod, waits for its readiness probe to pass (confirming it's healthy), then removes one old pod. It repeats this process until all pods are updated. Our configuration has `maxUnavailable: 0`, which means there's never a moment with fewer pods than the desired count. Traffic is always being served."

---

**Q: "What is the difference between Docker Compose and Kubernetes?"**

**A:** "Docker Compose is for running multi-container applications on a single machine. It's great for local development — you run `docker-compose up` and all services start together. Kubernetes is for production: it runs across multiple machines (nodes), handles failover between nodes, auto-scales based on load, manages secrets centrally, and provides enterprise-grade features. In our project, we use Docker Compose for local testing and AKS (Kubernetes) for production deployment."

---

**Q: "What would you do if a Kubernetes deployment fails after reaching production?"**

**A:** "We use Kubernetes rollback:

```bash
# See rollout history
kubectl rollout history deployment/backend-deployment -n mern-app

# Rollback to previous version
kubectl rollout undo deployment/backend-deployment -n mern-app

# Verify rollback
kubectl rollout status deployment/backend-deployment -n mern-app
```

This takes about 30 seconds. The beauty of our setup is that old images are still tagged with their build numbers in ACR, so we can also roll back to any specific previous version by updating the image tag in the deployment."

---

**Q: "How do you store secrets securely in your pipeline?"**

**A:** "We have a layered approach:

1. **Jenkins credentials store**: Passwords and tokens are stored in Jenkins (not in Jenkinsfile). The Jenkinsfile references them by ID using `credentials('id')`. Jenkins masks the values in logs.

2. **Kubernetes Secrets**: MongoDB URI and JWT secret are stored as Kubernetes Secrets (base64-encoded). They are injected into pods as environment variables at runtime.

3. **Never in Git**: `.env` files are in `.gitignore`. ACR passwords and Azure credentials are never committed to code.

4. **For production hardening**: We would replace Kubernetes Secrets with Azure Key Vault and use the External Secrets Operator to sync secrets dynamically."

---

**Q: "How does Prometheus know what to monitor?"**

**A:** "Prometheus uses a 'pull' model — it scrapes metrics from targets at regular intervals. There are two ways it discovers targets:

1. **Static config**: We explicitly list endpoints in `prometheus.yml` (e.g., our Jenkins VM's IP)

2. **Kubernetes Service Discovery**: Prometheus automatically discovers pods and services in Kubernetes. It scrapes any pod that has the annotation `prometheus.io/scrape: 'true'` and the metric port specified.

Our Node.js backend exposes metrics at the `/metrics` endpoint using the `prom-client` library. Prometheus scrapes this every 15 seconds and stores the time-series data."

---

**Q: "What is the purpose of a Namespace in Kubernetes?"**

**A:** "A namespace provides logical isolation within a Kubernetes cluster. Different teams or applications can share the same cluster but operate in separate namespaces. Resources in one namespace don't conflict with resources in another (you can have two services named 'backend-service' in different namespaces). Namespaces also enable access control — you can give a team permissions only to their namespace, not the entire cluster. Our MERN application lives in the `mern-app` namespace, while monitoring lives in `monitoring`."

---

## Summary: What Each Tool Does

```
Tool             | Category    | What It Secures / Does
-----------------|-------------|----------------------------------------
Jenkins          | CI/CD       | Orchestrates the entire pipeline
SonarQube        | SAST        | Scans source code for vulnerabilities
Trivy            | Container   | Scans images for CVEs
OWASP ZAP        | DAST        | Tests running app like a hacker
Docker           | Container   | Packages app + dependencies
ACR              | Registry    | Stores Docker images privately
AKS              | Kubernetes  | Runs and manages containers
Prometheus       | Monitoring  | Collects metrics from apps/cluster
Grafana          | Monitoring  | Visualizes metrics as dashboards
Azure CLI        | Cloud       | Manages all Azure resources
kubectl          | Kubernetes  | Interacts with Kubernetes cluster
```

---

# APPENDIX: QUICK REFERENCE COMMANDS

## Most Used Commands

```bash
# Get all resources in namespace
kubectl get all -n mern-app

# Tail logs from a pod
kubectl logs -f <pod-name> -n mern-app

# Execute a command inside a running pod
kubectl exec -it <pod-name> -n mern-app -- sh

# Rollback a deployment
kubectl rollout undo deployment/<name> -n mern-app

# Scale a deployment
kubectl scale deployment/<name> --replicas=2 -n mern-app

# View pod details and events
kubectl describe pod <pod-name> -n mern-app

# Check AKS credentials
az aks get-credentials --resource-group mern-cicd-rg --name mern-cicd-aks

# List ACR images
az acr repository list --name merncicdacr --output table

# Check Azure credit balance
az consumption budget list

# Stop AKS (save money)
az aks stop --name mern-cicd-aks --resource-group mern-cicd-rg

# Delete all resources
az group delete --name mern-cicd-rg --yes
```

---

*Guide Complete — Implementation of Secure CI/CD Pipeline Using DevOps Tools*  
*Azure for Students | MERN Stack | Jenkins | SonarQube | Trivy | OWASP ZAP | AKS | Prometheus | Grafana*
