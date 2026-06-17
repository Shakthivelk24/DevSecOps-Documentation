## 📋 Prerequisites

Before running the Docker commands, ensure the following software is installed on your system:

| Software | Purpose |
|----------|---------|
| **Docker Desktop** | Required to create and manage Docker containers. |
| **Git** | Used to clone repositories from GitHub. |
| **GitHub Account** | Required for source code management and version control. |
| **Internet Connection** | Needed to pull Docker images from Docker Hub. |

> **Note:** Kubernetes (Kind/Minikube) can be installed later if container orchestration is required. Docker Desktop must be running before executing any of the commands below.


## 🐳 Docker Commands

| Tool | Command | Purpose |
|--------|---------|----------|
| **Jenkins** | `docker pull jenkins/jenkins:lts`<br>`docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts` | Downloads and starts the Jenkins container. |
| **SonarQube** | `docker pull sonarqube:community`<br>`docker run -d --name sonarqube -p 9000:9000 sonarqube:community` | Downloads and starts the SonarQube container for static code analysis. |
| **Trivy** | `docker pull aquasec/trivy`<br>`docker run --rm aquasec/trivy --version` | Downloads Trivy and verifies its installation. |
| **OWASP ZAP** | `docker pull ghcr.io/zaproxy/zaproxy:stable`<br>`docker run -d --name owasp-zap -u zap -p 8081:8080 ghcr.io/zaproxy/zaproxy:stable zap-webswing.sh` | Downloads and starts the OWASP ZAP web interface. |
| **Prometheus** | `docker pull prom/prometheus`<br>`docker run -d --name prometheus -p 9090:9090 prom/prometheus` | Downloads and starts the Prometheus monitoring server. |
| **Grafana** | `docker pull grafana/grafana`<br>`docker run -d --name grafana -p 3000:3000 grafana/grafana` | Downloads and starts the Grafana dashboard server. |
| **Docker Management** | `docker ps`<br>`docker ps -a`<br>`docker stop <container_name>`<br>`docker start <container_name>`<br>`docker restart <container_name>`<br>`docker rm -f <container_name>` | Lists, stops, starts, restarts, and removes Docker containers. |
