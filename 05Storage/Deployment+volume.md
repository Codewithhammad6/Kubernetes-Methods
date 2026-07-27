# Kubernetes Deployment with PersistentVolume (PV) & PersistentVolumeClaim (PVC)

## Objective

Learn how a Deployment uses a **PersistentVolumeClaim (PVC)** to store data permanently.

---

# Files Used

```
pv.yml
pvc.yml
deployment.yml
```

Deployment does **not** use the PersistentVolume directly.

Instead, it uses the PersistentVolumeClaim.

---

# Architecture

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pods
      │
      ▼
Container
      │
      ▼
volumeMounts
      │
      ▼
Pod Volume
      │
      ▼
PersistentVolumeClaim (PVC)
      │
      ▼
PersistentVolume (PV)
      │
      ▼
Worker Node Storage
      │
      ▼
/mnt/data
```

---

# Step 1 - Create PersistentVolume

Create **pv.yml**

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: local-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: local-storage

  hostPath:
    path: /mnt/data
```

Apply

```bash
kubectl apply -f pv.yml
```

---

# Step 2 - Create PersistentVolumeClaim

Create **pvc.yml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: local-pvc
  namespace: nginx

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi

  storageClassName: local-storage
```

Apply

```bash
kubectl apply -f pvc.yml
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

        volumeMounts:
        - name: my-storage
          mountPath: /data

      volumes:
      - name: my-storage
        persistentVolumeClaim:
          claimName: local-pvc
```

Apply

```bash
kubectl apply -f deployment.yml
```

---

# How It Works

## Step 1

Deployment creates Pods.

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pods
```

---

## Step 2

Pod reads

```yaml
volumes:
```

```yaml
volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: local-pvc
```

Meaning

```
Pod

↓

Create Volume

↓

my-storage

↓

Use local-pvc
```

---

## Step 3

PVC is already connected with the PV.

```
PersistentVolume

↓

PersistentVolumeClaim

↓

Pod Volume
```

---

## Step 4

Container reads

```yaml
volumeMounts:
```

```yaml
volumeMounts:
- name: my-storage
  mountPath: /data
```

Meaning

```
Pod Volume

↓

Container

↓

/data
```

---

# Complete Flow

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
Container
      │
      ▼
volumeMounts
      │
      ▼
Pod Volume
      │
      ▼
PersistentVolumeClaim
      │
      ▼
PersistentVolume
      │
      ▼
Worker Node

/mnt/data
```

---

# Data Flow

Application writes

```
/data/file.txt
```

Actually stored in

```
Worker Node

/mnt/data/file.txt
```

---

# If Pod Gets Deleted

```
Pod ❌

↓

New Pod ✅

↓

Uses same PVC

↓

Uses same PV

↓

Same Data ✅
```

The data is **not lost** because it is stored in the PersistentVolume.

---

# Difference Between volumes and volumeMounts

## volumes

Creates a Pod volume from the PVC.

```yaml
volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: local-pvc
```

Flow

```
PV

↓

PVC

↓

Pod Volume
```

---

## volumeMounts

Mounts the Pod volume inside the container.

```yaml
volumeMounts:
- name: my-storage
  mountPath: /data
```

Flow

```
Pod Volume

↓

Container

↓

/data
```

---

# Useful Commands

Apply PV

```bash
kubectl apply -f pv.yml
```

Apply PVC

```bash
kubectl apply -f pvc.yml
```

Apply Deployment

```bash
kubectl apply -f deployment.yml
```

Check PV

```bash
kubectl get pv
```

Check PVC

```bash
kubectl get pvc -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Describe PVC

```bash
kubectl describe pvc local-pvc -n nginx
```

---

# Final Revision

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
Container
      │
      ▼
volumeMounts
      │
      ▼
Pod Volume
      │
      ▼
PersistentVolumeClaim (PVC)
      │
      ▼
PersistentVolume (PV)
      │
      ▼
Worker Node Storage
```

---

# One-Line Revision

- **PV** → Provides actual storage.
- **PVC** → Requests storage from the PV.
- **volumes** → Creates a Pod volume using the PVC.
- **volumeMounts** → Mounts that Pod volume inside the container.
- **Deployment** → Creates Pods that use the PVC for persistent storage.