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