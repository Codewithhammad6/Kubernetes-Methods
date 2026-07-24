# Kubernetes (Kind) Setup on AWS EC2

## Objective
Learn how to create a Kubernetes cluster using Kind on an AWS EC2 instance.

---

# Step 1: Launch EC2 Instance

- Launch Ubuntu EC2 Instance
- Storage: **30 GB**
- Instance Type: Select a high-performance instance (as required)
- Download the `.pem` key
- Connect using SSH

```bash
ssh -i devops.pem ubuntu@<EC2-PUBLIC-IP>
```

---

# Step 2: Install Docker

Check Docker:

```bash
docker --version
```

If Docker is not installed:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

Add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify:

```bash
docker run hello-world
```

---

# Step 3: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

---

# Step 4: Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind
```

Verify:

```bash
kind version
```

---

# Kubernetes Architecture

```
Your Laptop
│
├── Operating System (Windows/Linux/macOS/Ubuntu)
│
├── Docker
│
└── Kubernetes Cluster (Kind)
    │
    ├── Docker Container
    │   └── Control Plane Node
    │       ├── API Server
    │       ├── etcd
    │       ├── Scheduler
    │       ├── Controller Manager
    │       └── kubelet
    │
    ├── Docker Container
    │   └── Worker Node
    │       ├── kubelet
    │       ├── kube-proxy
    │       ├── containerd
    │       ├── Pod
    │       │   └── Container
    │       └── ...
    │
    └── Docker Container
        └── Worker Node
            ├── kubelet
            ├── kube-proxy
            ├── containerd
            └── Pods
```

### Components

- **kubectl** → Command Line Tool (Manager)
- **API Server** → Receives all requests
- **etcd** → Stores cluster data
- **Scheduler** → Assigns Pods to Nodes
- **Controller Manager** → Maintains desired state
- **kubelet** → Runs Pods on each node
- **kube-proxy** → Handles networking
- **containerd** → Runs containers

---

# Step 5: Create Kind Configuration

Create a file named:

```
config.yml
```

Example:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

---

# Step 6: Create Cluster

```bash
kind create cluster --name mycluster --config config.yml
```

---

# Step 7: Verify Cluster

### List Clusters

```bash
kind get clusters
```

### Check Nodes

```bash
kubectl get nodes
```

### Check Current Context

```bash
kubectl config current-context
```

---

# Learning Summary

✅ Created AWS EC2 Instance

✅ Connected via SSH

✅ Installed Docker

✅ Installed kubectl

✅ Installed Kind

✅ Created config.yml

✅ Created Kubernetes Cluster

✅ Verified Cluster

---
