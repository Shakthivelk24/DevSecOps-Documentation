# 🔐 DevSecOps Documentation on Microsoft Azure

<div align="center">

![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![DevSecOps](https://img.shields.io/badge/DevSecOps-Security-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

A comprehensive documentation repository for implementing a **Secure DevSecOps CI/CD Pipeline on Microsoft Azure**, covering secure application deployment, automated code quality analysis, container security, Kubernetes, monitoring, and cloud deployment.

### 🔗 Project Repository

1. **https://github.com/Shakthivelk24/Request-System-DevSecOps**
2. **https://github.com/Shakthivelk24/virtual-assistant.git**

</div>

---

# 📖 About

This repository documents the implementation of a secure DevSecOps pipeline using **Microsoft Azure** and modern DevOps tools.

The documentation covers the complete workflow from source code management to deployment, container orchestration, security scanning, and monitoring.

It includes:

- Secure CI/CD Pipeline using Jenkins
- Microsoft Azure Virtual Machine Deployment
- Docker Containerization
- Kubernetes Deployment (Local Cluster)
- SonarQube Code Quality Analysis
- Trivy Container Image Vulnerability Scanning
- Docker Hub Image Publishing
- Docker Compose Deployment
- Monitoring with Prometheus
- Visualization with Grafana

---

# 📸 Screenshots

## 🔑 Jenkins Dashboard

![Jenkins](./Output/jenkins/SecureCICD.png)

---

## 🛡️ SonarQube Dashboard

![SonarQube](./Output/security/SonarQube.png)

---


## 🔍 Trivy Report

![Trivy](./Output/security/TrivyFrontendreport.png)

---

## 📈 Grafana Dashboard

![Grafana](./Output/monitoring/Node.jsSystemMetrics.png)

---

## 💻 Promethus Dashboard

![Promethus](./Output/monitoring/Prometheus.png)

---



# 🛠️ Technology Stack

| Category | Technology |
|------------|------------|
| Cloud Platform | Microsoft Azure Virtual Machine |
| Version Control | Git, GitHub |
| CI/CD | Jenkins |
| Build Tool | NPM |
| Code Quality | SonarQube |
| Vulnerability Scanner | Trivy |
| Containerization | Docker |
| Container Orchestration | Kubernetes (Local Cluster) |
| Container Registry | Docker Hub |
| Deployment | Docker Compose |
| Monitoring | Prometheus |
| Visualization | Grafana |

---

# 🚀 Topics Covered

- DevSecOps Fundamentals
- Git & GitHub Integration
- Jenkins CI/CD Pipeline
- NPM Build Automation
- SonarQube Static Code Analysis
- Trivy Container Image Vulnerability Scanning
- Docker Image Creation
- Docker Hub Integration
- Docker Compose Deployment
- Kubernetes Deployment (Local Cluster)
- Prometheus Monitoring
- Grafana Dashboards
- Secure DevSecOps Best Practices

---

# 🔄 DevSecOps Workflow

```text
                      Developer
                          │
                          ▼
                  GitHub Repository
                          │
                          ▼
                  Jenkins Pipeline
                          │
        ┌─────────────────┼──────────────────┐
        │                 │                  │
        ▼                 ▼                  ▼
 Install Dependencies  Unit Tests   SonarQube Analysis
        │
        ▼
   Trivy Image Scan
        │
        ▼
 Docker Image Build
        │
        ▼
 Push Image to Docker Hub
        │
        ├──────────────────────────────┐
        ▼                              ▼
 Docker Compose                 Kubernetes
 (Azure Virtual Machine)       (Local Cluster)
        │                              │
        └──────────────┬───────────────┘
                       ▼
             Running Application
                       │
                       ▼
            Prometheus Monitoring
                       │
                       ▼
              Grafana Dashboard
```


---

# 🎯 Objectives

- Learn DevSecOps concepts
- To design and implement a secure CI/CD pipeline 
- To identify and mitigate vulnerabilities at early stages of development 
- To reduce security risks and remediation costs 
- To ensure faster and secure software delivery 
- Build a secure CI/CD pipeline
- Deploy applications on Microsoft Azure
- Deploy applications on Kubernetes
- Automate code quality analysis
- Perform container vulnerability scanning
- Build and publish Docker images
- Monitor deployed applications
- Understand production-ready DevSecOps workflows

---


# 📚 Documentation Includes


- Jenkins Installation & Configuration
- GitHub Integration
- Docker Installation
- Docker Compose Deployment
- Kubernetes Deployment
- SonarQube Installation & Configuration
- Trivy Installation & Image Scanning
- Docker Hub Integration
- Prometheus Installation
- Grafana Dashboard Configuration
- Complete CI/CD Pipeline Setup
- Best Practices and Troubleshooting

---

# 🎓 Learning Outcomes

After following this documentation, you will understand how to:

- Configure a secure DevSecOps environment
- Build automated CI/CD pipelines
- Deploy applications using Docker Compose
- Deploy containerized applications on Kubernetes
- Perform static code analysis with SonarQube
- Scan Docker images using Trivy
- Publish Docker images to Docker Hub
- Deploy applications on Microsoft Azure Virtual Machines
- Monitor applications using Prometheus
- Visualize metrics using Grafana

---

# 🤝 Contributing

Contributions are welcome!

Feel free to:

- Improve documentation
- Add new tutorials
- Fix issues
- Share DevSecOps best practices
- Enhance Azure deployment guides

---

# 📄 License

This project is licensed under the **MIT License**.

---

<div align="center">

⭐ If you find this repository useful, consider giving it a star!

### 🚀 Project Repository

1. **https://github.com/Shakthivelk24/Request-System-DevSecOps**
2. **https://github.com/Shakthivelk24/virtual-assistant.git**

Made with ❤️ by **Shakthi Vel K , Sanket , Revanth Kumar T S, Vignesh P**

</div>