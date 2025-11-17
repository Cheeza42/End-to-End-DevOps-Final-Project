# 🚀 End-to-End-DevOps-Final-Project
Hi there! This is my ENd-to-End final project in DevOps studies. This project will present you a full DevOps work-flow.  

---

# 🌐 AWS Flask Viewer
A simple Flask web application packaged in a Docker container.  
It uses **boto3** to connect to AWS and displays the following resources in an HTML page:
- 🖥️ EC2 instances  
- 🌉 VPCs  
- ⚖️ Load Balancers  
- 📸 AMIs  

The application runs on **port 5001**.

---

## ✅ Prerequisites
- 🐳 Docker installed  
- 🔑 AWS IAM user with:  
  - Access Key ID  
  - Secret Access Key  
- 👮 Permissions to list EC2, VPC, ELB, and AMI resources (for testing, `AdministratorAccess` is sufficient)  

---

## 🛠️ Instructions

### 🛠️ Build Locally
```bash
docker build -t project-flask .
```
## 📦 Pull from Docker Hub
```bash
docker pull cheeza42/dockerizing-project:v1
```
**View on Docker Hub:**
 https://hub.docker.com/r/cheeza42/dockerizing-project


## ▶️ Run the Container on AWS
 ```bash
docker run -d --name project-flask \
  -p 5001:5001 \
  -e AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY \
  -e AWS_DEFAULT_REGION=eu-west-1 \
  cheeza42/dockerizing-project:v1
 ```
## 🌍 Access the Application
Open your browser at:
👉 http://localhost:5001/


## 🛑 Stop and Remove the Container
```bash
docker stop project-flask
docker rm project-flask
```

## ⎈ Deploy on Kubernetes with Helm

This repo also includes a Helm chart (chart/helm_chart_projct) to deploy the app to Kubernetes.

## 📋 Prerequisites

Kubernetes cluster with kubectl configured

Helm v3+

NGINX Ingress Controller (required because ingress.enabled: true in values.yaml)

## 🚀 Install
```bash
helm install aws-viewer ./helm_chart_projct
kubectl get pods,svc,ingress
```

## 🌐 Access the app
Ingress (enabled)

According to values.yaml:

- ingress.enabled: true

- ingress.ingressClassName: nginx

- ingress.host: awsviewer.com

So the app will be available at:

http://awsviewer.com/


## ⚠️ DNS for awsviewer.com must resolve to your Ingress controller’s IP.
For local testing (e.g., minikube), add an entry in /etc/hosts:
<INGRESS-IP> awsviewer.com

## Fallback: ClusterIP (port-forward)

If Ingress is not working, the Service is ClusterIP on port 80 → targetPort 5001.
You can forward locally:
```bash
kubectl port-forward svc/aws-viewer 8080:80
```
 # Then open http://localhost:8080/

## 🔄 Upgrade / Uninstall
```bash
helm upgrade aws-viewer ./helm_chart_projct -f helm_chart_projct/values.yaml
helm uninstall aws-viewer
```
## ⚙️ CI/CD Automation with Jenkins & Kaniko

This project integrates **Jenkins** and **Kaniko** to automate the entire build and push process of the Docker image to Docker Hub — without requiring Docker to run inside the Jenkins agent.  

### 🧩 Pipeline Overview
The Jenkins pipeline performs the following stages:

1. **Clone Repository** – Pulls the latest code from GitHub.  
2. **Compute Tag** – Generates a unique image tag based on date and commit SHA.  
3. **Parallel Checks** –  
   - *Linting:* Runs `flake8`, `hadolint`, and `shellcheck`.  
   - *Security Scanning:* Runs `bandit` and `trivy`.  
4. **Build & Push with Kaniko** –  
   Uses the Kaniko executor container to build the Docker image from the `Dockerfile` and push it directly to Docker Hub.

### ☸️ Environment
The Jenkins pipeline runs inside a **Kubernetes Pod**, where:
- The **Kaniko container** handles image building and pushing to Docker Hub.  
- The **Jenkins agent container** manages pipeline orchestration and log output.

### 🧱 Jenkinsfile Reference
The complete pipeline implementation can be found in the project root under:  
📄 **[`jenkinsfile`](./jenkinsfile)**  

It defines all stages, environment variables, and credentials used for the automated CI/CD workflow.

### 📦 Result
Once the pipeline completes, the image is automatically pushed to Docker Hub under:  
👉 **[`cheeza42/dockerizing-project`](https://hub.docker.com/r/cheeza42/dockerizing-project)**

## Prometheus & Grafana Monitoring

The application exposes metrics using `prometheus-flask-exporter`.

A `PodMonitor` object is included to allow Prometheus to scrape metrics directly
from the application pods instead of relying only on the Service layer.

Prometheus collects:
- total request count
- request latency
- per-pod health and performance details
- all metrics exposed from the `/metrics` endpoint on port `5001`

Grafana visualizes these metrics in real time through the Prometheus data source.
After accessing Grafana (for example using `kubectl port-forward`), you can view
live dashboards that reflect the application's traffic, behavior, and performance.


## 📑 Chart details

Replicas: 2

Image: cheeza42/dockerizing-project:v1

Service: ClusterIP (port 80 → container port 5001)

Ingress: NGINX, host awsviewer.com

AWS credentials: passed as environment variables

Probes: readiness & liveness on /ready and /health  

Resources: requests (100m CPU, 128Mi RAM), limits (500m CPU, 256Mi RAM) 

Service labels: app: <Release.Name>  

✅ Recommendation: In production, store AWS credentials in a Kubernetes Secret and reference them from the Deployment.

