# Dashboard work in HTTPS OR LOCAL

## In dashboard you have every thing that is in your cluster in which you apply dashboard it show pods deploymet replica job etc....

# Kubernetes Dashboard (EC2 / Kubeadm)

## Step 1: Check Cluster

First, verify that your Kubernetes cluster is running.

```bash
kubectl cluster-info
```

Check available nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME           STATUS   ROLES           AGE
master-node    Ready    control-plane   2d
worker-node    Ready    <none>          2d
```

---

# Step 2: Install Kubernetes Dashboard

Install the Dashboard:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Verify all resources:

```bash
kubectl get all -n kubernetes-dashboard
```

Expected resources:

- Pod
- Deployment
- ReplicaSet
- Service

---

# Step 3: Verify Dashboard Pods

```bash
kubectl get pods -n kubernetes-dashboard
```

Example:

```text
NAME                                         READY   STATUS
dashboard-metrics-scraper-xxxx               1/1     Running
kubernetes-dashboard-xxxx                    1/1     Running
```

---

# Step 4: Verify Dashboard Service

```bash
kubectl get svc -n kubernetes-dashboard
```

Example:

```text
NAME                        TYPE        PORT
kubernetes-dashboard        ClusterIP   443
dashboard-metrics-scraper   ClusterIP   8000
```

---

# Step 5: Create ServiceAccount

Create a file:

```text
admin-user.yml
```

```yaml
apiVersion: v1
kind: ServiceAccount

metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply:

```bash
kubectl apply -f admin-user.yml
```

---

# Step 6: Create ClusterRoleBinding

Create:

```text
cluster-role-binding.yml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: admin-user

subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f cluster-role-binding.yml
```

> **Note:** `cluster-admin` gives full access to the entire Kubernetes cluster. It is fine for learning, but in production always use the principle of least privilege (RBAC).

---

# Step 7: Generate Login Token

For Kubernetes v1.24+:

```bash
kubectl create token admin-user -n kubernetes-dashboard
```

Example:

```text
eyJhbGciOiJSUzI1NiIs...
```

Copy this token.

---

# Step 8: Start kubectl Proxy

### Allow access from all interfaces (EC2)

```bash
kubectl proxy --address=0.0.0.0 --port=8001 --accept-hosts='.*'
```

### Local machine only

```bash
kubectl proxy
```

Verify:

```text
Starting to serve on 0.0.0.0:8001
```

---

# Step 9: Test Proxy

```bash
curl http://localhost:8001
```

---

# Step 10: Open Dashboard

If using AWS EC2:

Allow **TCP Port 8001** in the EC2 Security Group.

Open:

```text
http://<EC2-PUBLIC-IP>:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Login using the generated token.

---

# Dashboard Architecture

```text
Browser
    │
    ▼
kubectl proxy
    │
    ▼
Kubernetes Dashboard
    │
    ▼
Kubernetes Cluster
```

---

# Useful Commands

```bash
kubectl get all -n kubernetes-dashboard

kubectl get pods -n kubernetes-dashboard

kubectl get svc -n kubernetes-dashboard

kubectl logs -n kubernetes-dashboard <pod-name>

kubectl describe pod <pod-name> -n kubernetes-dashboard
```

---

# Kubernetes Dashboard on Kind (Local Machine)

## Requirements

- Docker Desktop
- PowerShell (Run as Administrator)
- Chocolatey

---

# Step 1: Verify Chocolatey

```powershell
choco --version
```

---

# Step 2: Install Kind

```powershell
choco install kind -y
```

Verify installation:

```powershell
kind version
```

---

# Step 3: Create Kind Config

Create a file:

```text
kind-config.yml
```

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane

  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP

  - containerPort: 443
    hostPort: 443
    protocol: TCP

- role: worker

- role: worker
```

---

# Step 4: Create Cluster

```powershell
kind create cluster --name mycluster --config kind-config.yml
```

Verify:

```bash
kubectl get nodes
```

Example:

```text
NAME                          STATUS
mycluster-control-plane       Ready
mycluster-worker              Ready
mycluster-worker2             Ready
```

---

# Step 5: Install Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Verify:

```bash
kubectl get all -n kubernetes-dashboard
```

---

# Step 6: Create ServiceAccount

Create:

```text
admin-user.yml
```

```yaml
apiVersion: v1
kind: ServiceAccount

metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply:

```bash
kubectl apply -f admin-user.yml
```

---

# Step 7: Create ClusterRoleBinding

Create:

```text
cluster-role-binding.yml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: admin-user

subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f cluster-role-binding.yml
```

---

# Step 8: Generate Token

```bash
kubectl create token admin-user -n kubernetes-dashboard
```

Copy the generated token.

---

# Step 9: Start Proxy

```bash
kubectl proxy
```

---

# Step 10: Open Dashboard

Open in your browser:

```text
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Login using the generated token.

---

# Kind Architecture

```text
Docker Desktop
        │
        ▼
Kind Cluster
        │
 ┌──────┴──────┐
 │             │
 ▼             ▼
Control Plane  Worker 1
               Worker 2
        │
        ▼
Kubernetes Dashboard
        │
        ▼
Browser
```

> **Note:** Dashboard resources (Deployment, ReplicaSet, Pods, and Services) are created automatically when you install the Dashboard. You only need to create the **ServiceAccount** and **ClusterRoleBinding** to log in with admin permissions.