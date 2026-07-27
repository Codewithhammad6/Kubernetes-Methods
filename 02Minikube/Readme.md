# Kubernetes Cluster Setup using Minikube

## Objective

Learn how to create a single-node Kubernetes cluster using **Minikube**.

Minikube is designed for local development and Kubernetes practice. It creates a lightweight Kubernetes cluster on a single machine.

---

# What is Minikube?

Minikube is an official Kubernetes tool that runs a local Kubernetes cluster.

It is commonly used for:

- Learning Kubernetes
- Local Development
- Testing Applications
- Practicing kubectl commands

---

# Architecture

```
Your Laptop / EC2
│
├── Ubuntu
│
├── Docker
│
└── Minikube Cluster
    │
    └── Single Node
        ├── API Server
        ├── etcd
        ├── Scheduler
        ├── Controller Manager
        ├── kubelet
        ├── kube-proxy
        ├── containerd
        │
        ├── Pod 1
        │    └── Container A
        │
        ├── Pod 2
        │    └── Container B
        │
        └── Pod 3
             └── Container C
```

---

# AWS EC2 Setup

## Instance Configuration

| Setting | Value |
|---------|------|
| OS | Ubuntu 22.04 |
| Storage | 20 GB |
| Instance Type | 2 vCPU / 2GB+ RAM |
| Security Group | SSH (22), HTTP (80) |

---

# Step 1 - Connect to EC2

```bash
ssh -i devops.pem ubuntu@<PUBLIC-IP>
```

---

# Step 2 - Install Docker

```bash
sudo apt update
sudo apt install docker.io -y

sudo systemctl enable docker
sudo systemctl start docker

sudo usermod -aG docker $USER
newgrp docker
```

Verify

```bash
docker run hello-world
```

---

# Step 3 - Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

Verify

```bash
kubectl version --client
```

---

# Step 4 - Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify

```bash
minikube version
```

---

# Step 5 - Start Cluster

```bash
minikube start --driver=docker
```

---

# Step 6 - Verify Cluster

Check status

```bash
minikube status
```

Check nodes

```bash
kubectl get nodes
```

Expected Output

```
NAME        STATUS   ROLES
minikube    Ready    control-plane
```

---

# Useful Commands

Start Cluster

```bash
minikube start
```

Stop Cluster

```bash
minikube stop
```

Delete Cluster

```bash
minikube delete
```

Dashboard

```bash
minikube dashboard
```

Cluster Info

```bash
kubectl cluster-info
```

Get Nodes

```bash
kubectl get nodes
```

Get Pods

```bash
kubectl get pods -A
```

---

# Learning Summary

✅ Installed Docker

✅ Installed kubectl

✅ Installed Minikube

✅ Created Local Kubernetes Cluster

✅ Verified Cluster

---
