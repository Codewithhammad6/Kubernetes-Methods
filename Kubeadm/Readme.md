# Kubernetes Cluster Setup using kubeadm (AWS EC2)

## Objective

Learn how to create a Production-Style Kubernetes Cluster using **kubeadm** on AWS EC2.

Unlike Kind, kubeadm creates a **real Kubernetes cluster** where the Control Plane and Worker Nodes run on separate machines.

---

# What is kubeadm?

**kubeadm** is an official Kubernetes tool used to bootstrap a Kubernetes cluster.

It is commonly used for:

- Self-managed Production Clusters
- On-Premise Data Centers
- Virtual Machines (VMs)
- Learning real Kubernetes architecture

---

# Cluster Architecture

```
               Kubernetes Cluster

              Control Plane Node
               (kubeadm init)
                     │
        ┌────────────┴────────────┐
        │                         │
   Worker Node 1             Worker Node 2
   (kubeadm join)            (kubeadm join)
```

---

# AWS EC2 Setup

Create **2 EC2 instances together** so both have the same Security Group.

## Instance Configuration

| Setting | Value |
|---------|------|
| OS | Ubuntu 22.04 |
| Storage | 20 GB |
| Instance Type | High Performance (2 vCPU recommended) |
| Security Group | Same for both instances |
| HTTP | Allow |
| SSH (22) | Allow |
| Kubernetes API (6443) | Allow |

---

# Node Configuration

| Node | Role |
|------|------|
| Master | Control Plane |
| Worker | Worker Node |

---

# Requirements

Each node should have:

- Ubuntu 22.04
- Minimum 2 CPU
- Minimum 2 GB RAM
- 20 GB Storage
- Private IP
- Internet Access

---

# Step 1 - Connect to EC2

```bash
ssh -i devops.pem ubuntu@<PUBLIC-IP>
```

Do this for both instances.

---

# Step 2 - Update System

Run on **all nodes**

```bash
sudo apt update
sudo apt upgrade -y
```

---

# Step 3 - Disable Swap

Kubernetes does not support swap.

Temporary

```bash
sudo swapoff -a
```

Permanent

```bash
sudo nano /etc/fstab
```

Comment or remove the swap entry.

---

# Step 4 - Install Container Runtime (containerd)

```bash
sudo apt install containerd -y
```

Create default configuration

```bash
sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml
```

Edit configuration

```bash
sudo nano /etc/containerd/config.toml
```

Find

```
SystemdCgroup = false
```

Change to

```
SystemdCgroup = true
```

Restart

```bash
sudo systemctl restart containerd

sudo systemctl enable containerd
```

---

# Step 5 - Install Kubernetes Packages

Run on all nodes.

Install dependencies

```bash
sudo apt update

sudo apt install -y apt-transport-https ca-certificates curl
```

Download Kubernetes GPG Key

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes.gpg
```

Add Repository

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install Packages

```bash
sudo apt update

sudo apt install -y kubelet kubeadm kubectl
```

Prevent Automatic Updates

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

---

# Control Plane Setup

Run only on the **Master Node**

## Step 6 - Initialize Cluster

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

At the end you will get a command like

```bash
kubeadm join 10.0.1.5:6443 \
--token abc123 \
--discovery-token-ca-cert-hash sha256:xxxxxxxx
```

Save this command.

It will be used on Worker Nodes.

---

## Step 7 - Configure kubectl

```bash
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify

```bash
kubectl get nodes
```

Initially

```
NAME      STATUS      ROLE
master    NotReady    control-plane
```

---

# Step 8 - Install CNI Plugin

Install Calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Verify

```bash
kubectl get pods -n kube-system
```

---

# Worker Node Setup

Run on every Worker Node.

```bash
sudo kubeadm join <MASTER_PRIVATE_IP>:6443 \
--token <TOKEN> \
--discovery-token-ca-cert-hash sha256:<HASH>
```

Example

```bash
sudo kubeadm join 10.0.1.5:6443 \
--token abcdef.123456 \
--discovery-token-ca-cert-hash sha256:abcd1234
```

---

# Verify Cluster

Run on Master

```bash
kubectl get nodes
```

Expected Output

```
NAME       STATUS   ROLE
master     Ready    control-plane
worker1    Ready    <none>
worker2    Ready    <none>
```

---

# Useful Commands

Check Nodes

```bash
kubectl get nodes
```

Check Pods

```bash
kubectl get pods -A
```

Cluster Information

```bash
kubectl cluster-info
```

Component Status

```bash
kubectl get componentstatus
```

Current Context

```bash
kubectl config current-context
```

---

# Production Comparison

| Tool | Purpose |
|------|---------|
| Kind | Local Learning |
| Minikube | Local Practice |
| kubeadm | Self-Managed Production Cluster |
| Amazon EKS | Managed Kubernetes on AWS |
| Azure AKS | Managed Kubernetes on Azure |
| Google GKE | Managed Kubernetes on GCP |

---

# Learning Summary

✅ Created AWS EC2 Instances

✅ Configured Security Group

✅ Installed containerd

✅ Installed kubeadm

✅ Installed kubelet

✅ Installed kubectl

✅ Initialized Control Plane

✅ Installed Calico Network

✅ Joined Worker Nodes

✅ Verified Kubernetes Cluster

---
================================================================
# Deploying an Nginx Application in Kubernetes

## Objective

Learn how to deploy an Nginx application using Kubernetes YAML files.

---

# Kubernetes Deployment Workflow

```
Create Folder
      │
      ▼
Create Namespace
      │
      ▼
Apply Namespace
      │
      ▼
Create Deployment
      │
      ▼
Apply Deployment
      │
      ▼
Deployment creates ReplicaSet
      │
      ▼
ReplicaSet creates Pods
      │
      ▼
Verify Resources
```

---

# Step 1 - Create Project Folder (Optional)

```bash
mkdir -p ~/kubernetes/nginx

cd ~/kubernetes/nginx
```

Folder Structure

```
nginx/
├── namespace.yml
├── deployment.yml
└── service.yml    (later)
```

---

# Step 2 - Create Namespace

**namespace.yml**

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: nginx
```

Apply

```bash
kubectl apply -f namespace.yml
```

Verify

```bash
kubectl get namespaces
```

---

# Step 3 - Create Deployment

Create **deployment.yml**

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: nginx

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
        image: nginx:1.14.2

        ports:
        - containerPort: 80
```

Apply

```bash
kubectl apply -f deployment.yml
```

---


# Step 4 - Verify Deployment

Check Deployments

```bash
kubectl get deployments -n nginx
```

Check ReplicaSets

```bash
kubectl get replicasets -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Expected Output

```
NAME                                READY   STATUS
nginx-deployment-xxxxxxxxxx-abcde   1/1     Running
nginx-deployment-xxxxxxxxxx-fghij   1/1     Running
```

---

# Deployment Flow

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pod
      │
      ▼
Container (Nginx)
```

---

# Do We Need pod.yml?

### For Learning

You can create a Pod directly using **pod.yml** to understand how a Pod works.

### For Real Projects

Normally **No**.

A Deployment automatically creates and manages Pods.

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pods
```

So, in production projects, **deployment.yml replaces pod.yml**.

---

# Useful Commands

Check Deployments

```bash
kubectl get deployments -n nginx
```

Check ReplicaSets

```bash
kubectl get replicasets -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Describe Deployment

```bash
kubectl describe deployment nginx-deployment -n nginx
```

Delete Deployment

```bash
kubectl delete deployment nginx-deployment -n nginx
```

Delete Namespace

```bash
kubectl delete namespace nginx
```

Increase replicas

```bash
kubectl scale deployment nginx-deployment --replicas=5 -n nginx
```

Decrease replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=2 -n nginx
```
---

# Learning Summary

✅ Created Namespace

✅ Created Deployment

✅ Deployment created ReplicaSet

✅ ReplicaSet created Pods

✅ Verified running Pods

✅ Learned why Deployment is preferred over Pod