# 🚀 GitOps with ArgoCD Setup

This guide will help you set up GitOps with ArgoCD step by step.

---

## 1. System Update & Prerequisites

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker

# Download and install KIND
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind --version

# Create Kubernetes cluster
kind create cluster

# Download and install kubectl
curl -LO https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

# Verify cluster
kubectl get nodes

# Download and install kubectl
curl -LO https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

# Verify cluster
kubectl get nodes

# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl get pods -n argocd --watch

# Get initial admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo

# Port forward to access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Configure git
git config --global user.email "your-email@example.com"
git config --global user.name "Your Name"

# Clone your repository
git clone https://github.com/your-username/gitops-demo.git
cd gitops-demo

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP

git add nginx-app/
git commit -m "Add nginx deployment and service manifests"
git push origin main

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/gitops-demo.git
    targetRevision: HEAD
    path: nginx-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

kubectl apply -f argo-app.yaml -n argocd

# Check application status in ArgoCD
kubectl get applications -n argocd

# Check resources in default namespace
kubectl get all -n default

# Access the application
kubectl port-forward svc/nginx-service 8081:80 -n default

