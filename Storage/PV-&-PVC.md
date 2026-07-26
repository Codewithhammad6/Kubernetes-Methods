# Kubernetes Persistent Volume (PV) & Persistent Volume Claim (PVC)



# BELOW SHORT NOTES


## Objective

Learn how Kubernetes stores data permanently using **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)**.

---

# Why Do We Need Persistent Storage?

Suppose you have a Pod.

```
Pod
 │
 ▼
Container
```

Inside the container, you create a file.

```
/data/user.txt
```

Now the Pod is deleted.

```
Pod ❌
```

A new Pod is created.

Will the file still exist?

**❌ No**

Because container storage is **ephemeral (temporary)**.

When the Pod is deleted, its local container data is also deleted.

---

# The Solution

Kubernetes says:

> **"Do not store important data inside the Pod."**

> **"Store the data outside the Pod."**

That external storage is called a **PersistentVolume (PV)**.

---

# Kubernetes Storage Architecture

```
                Kubernetes Cluster

      +-------------------------+
      |         Pod             |
      |                         |
      |   +-----------------+   |
      |   |   Container     |   |
      |   +-----------------+   |
      +-----------▲-------------+
                  │
                 PVC
                  │
                  ▼
      +-------------------------+
      | Persistent Volume (PV)  |
      +-----------▲-------------+
                  │
                  ▼
 AWS EBS / Azure Disk / GCP Disk /
      NFS / Local Disk / SSD
```

---

# What is a PersistentVolume (PV)?

A **PersistentVolume (PV)** is storage available inside the Kubernetes cluster.

It can be backed by:

- AWS EBS
- Azure Disk
- Google Persistent Disk
- NFS Server
- Local Disk
- SAN Storage

Example

```
100 GB Disk
```

Kubernetes knows that this storage is available for applications.

---

# PersistentVolume YAML

Create **pv.yml**

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: local-pv

  labels:
    app: local

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

---

# Important Fields

## capacity

```yaml
capacity:
  storage: 1Gi
```

Defines the total storage available.

---

## accessModes

```yaml
accessModes:
- ReadWriteOnce
```

Defines how the volume can be accessed.

Common values:

- ReadWriteOnce (RWO)
- ReadOnlyMany (ROX)
- ReadWriteMany (RWX)

---

## persistentVolumeReclaimPolicy

```yaml
persistentVolumeReclaimPolicy: Retain
```

Defines what happens when the PVC is deleted.

Common values:

- Retain
- Delete
- Recycle (deprecated)

---

## storageClassName

```yaml
storageClassName: local-storage
```

Used to match the PV with the correct PVC.

---

## hostPath

```yaml
hostPath:
  path: /mnt/data
```

The actual storage location on the Worker Node.

---

# What is a PersistentVolumeClaim (PVC)?

A **PersistentVolumeClaim (PVC)** is a request for storage.

The application does not use the PV directly.

Instead, it creates a PVC.

Example

```
I need 1Gi storage.
```

Kubernetes finds a matching PV and binds it automatically.

---

# PersistentVolumeClaim YAML

Create **pvc.yml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: local-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi

  storageClassName: local-storage
```

---

# PVC Flow

```
Developer

"I need 1Gi storage"

        │
        ▼

PersistentVolumeClaim

        │
        ▼

Kubernetes

        │
        ▼

PersistentVolume

        │
        ▼

Actual Storage
```

---

# Real-Life Example

Imagine a hotel.

```
Hotel
```

contains

```
100 Rooms
```

Rooms are the **PersistentVolume**.

A guest arrives.

The guest says:

```
I need a room.
```

This request is the **PVC**.

Reception assigns an available room.

The guest never chooses the room directly.

Exactly the same happens in Kubernetes.

---

# Complete Storage Flow

```
Developer

"I need storage"

        │
        ▼

PersistentVolumeClaim (PVC)

        │
        ▼

PersistentVolume (PV)

        │
        ▼

AWS EBS / Azure Disk /
NFS / Local Disk

        │
        ▼

Worker Node
```

---

# How Does a Pod Use Storage?

A Pod never connects directly to a PersistentVolume.

The flow is:

```
Pod

↓

PVC

↓

PV

↓

Storage
```

---

# Using PVC Inside a Pod

## Step 1 - Create a Volume

```yaml
volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: local-pvc
```

Meaning

Kubernetes creates a Pod volume named **my-storage** using the PVC **local-pvc**.

Flow

```
PV
 │
 ▼
PVC (local-pvc)
 │
 ▼
Pod Volume (my-storage)
```

---

## Step 2 - Mount the Volume

```yaml
volumeMounts:
- name: my-storage
  mountPath: /data
```

Meaning

Mount the Pod volume inside the container at `/data`.

Flow

```
PV
 │
 ▼
PVC (local-pvc)
 │
 ▼
Pod Volume (my-storage)
 │
 ▼
Container
   │
   ▼
 /data
```

Now anything written inside

```
/data
```

is stored in the PersistentVolume.

---

# Complete PV → PVC → Pod Flow

```
Developer
      │
      ▼
Creates PVC
      │
      ▼
PVC Bound to PV
      │
      ▼
Pod Created
      │
      ▼
Pod Volume Created
      │
      ▼
volumeMounts
      │
      ▼
Container
      │
      ▼
/data
      │
      ▼
Persistent Storage
```

---

# hostPath vs PV/PVC

## hostPath

```
Container
      │
      ▼
Worker Node
/demo-data
```

Problem

If the Pod moves to another Worker Node,

```
Worker-1

/demo-data

↓

Pod Rescheduled

↓

Worker-2

/demo-data
```

The data may not exist on the new Worker.

---

## PV/PVC

```
Worker-1
     │
     ▼
Pod
     │
     ▼
PVC
     │
     ▼
AWS EBS
```

If the Pod moves,

```
Worker-2
     │
     ▼
Pod
     │
     ▼
PVC
     │
     ▼
Same AWS EBS
```

The same storage is attached again.

Data remains available.

---

# Common Production Use Cases

| Application | Uses PVC? |
|-------------|-----------|
| MongoDB | ✅ Yes |
| MySQL | ✅ Yes |
| PostgreSQL | ✅ Yes |
| Kafka | ✅ Yes |
| Elasticsearch | ✅ Yes |
| Redis (Persistent Mode) | ✅ Yes |

---

# Useful Commands

Create PV

```bash
kubectl apply -f pv.yml
```

Create PVC

```bash
kubectl apply -f pvc.yml
```

Check PV

```bash
kubectl get pv
```

Check PVC

```bash
kubectl get pvc
```

Describe PV

```bash
kubectl describe pv local-pv
```

Describe PVC

```bash
kubectl describe pvc local-pvc
```

Delete PVC

```bash
kubectl delete pvc local-pvc
```

Delete PV

```bash
kubectl delete pv local-pv
```

---

# Interview Question

## Why doesn't a Pod use a PersistentVolume directly?

**Answer**

Kubernetes uses an abstraction layer.

- **PersistentVolume (PV)** provides the actual storage.
- **PersistentVolumeClaim (PVC)** requests storage.
- **Pod** mounts the PVC.

This allows applications to remain independent of the underlying storage provider (AWS EBS, Azure Disk, NFS, Local Disk, etc.).

---

# Quick Revision

```
PersistentVolume (PV)
        │
Provides Storage
        │
        ▼
PersistentVolumeClaim (PVC)
        │
Requests Storage
        │
        ▼
Pod Volume
        │
        ▼
volumeMounts
        │
        ▼
Container
        │
        ▼
/data
```

---

# One-Line Revision

- ✅ **PV** → Storage Provider (Actual storage such as hostPath, AWS EBS, Azure Disk, NFS, etc.)
- ✅ **PVC** → Storage Request
- ✅ **Pod** → Uses storage through the PVC
- ✅ **volumes** → Creates a Pod volume from the PVC
- ✅ **volumeMounts** → Mounts the Pod volume inside the container




# ============================




# Kubernetes Storage (PV & PVC) Short Notes

Normally agar MongoDB container ke andar data save karta hai to wo **container ki storage** me hota hai. Agar Pod delete ho jaye to naya Pod ban sakta hai aur container ka data lose ho sakta hai.

Is problem ko solve karne ke liye hum **PersistentVolume (PV)** aur **PersistentVolumeClaim (PVC)** use karte hain.

### PV (PersistentVolume)

* PV actual storage hoti hai jo kisi storage source se aati hai.
* Example: Worker Node ki disk ka folder:

```
/mnt/data
```

* PV Kubernetes ko batati hai:
  "Mere paas itni storage available hai."

Example:

```
Worker Node
     |
     ▼
/mnt/data
     |
     ▼
PV (1Gi Storage)
```

---

### PVC (PersistentVolumeClaim)

* PVC storage ki request hoti hai.
* Pod direct PV use nahi karta.
* Pod pehle PVC ko use karta hai.
* PVC suitable PV ko claim karti hai.

Flow:

```
PV
 |
 |  (storage provide)
 ▼
PVC
 |
 |  (storage request)
 ▼
Pod
 |
 ▼
Container
```

---

### Pod me PVC use karna

Pehle Pod me volume define karte hain:

```yaml
volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: local-pvc
```

Matlab:

> Pod ke andar `my-storage` naam ka volume banao aur usko `local-pvc` se connect karo.

Phir container ke andar mount karte hain:

```yaml
volumeMounts:
- name: my-storage
  mountPath: /data
```

Matlab:

> Ye storage container ke andar `/data` folder me available kar do.

---

Final Flow:

```
Worker Node
 |
 | /mnt/data
 |
 ▼
PersistentVolume (PV)
 |
 ▼
PersistentVolumeClaim (PVC)
 |
 ▼
Pod Volume (my-storage)
 |
 ▼
Container
 |
 ▼
/data
```

Ab agar application:

```
/data/file.txt
```

me data save karegi to wo asal me:

```
Worker Node:
/mnt/data/file.txt
```

me save hoga.

Agar Pod delete ho jaye:

```
Pod ❌ delete

New Pod ✅ create

Data ✅ safe (PV ki wajah se)
```

---

## Yaad rakhne ki line

**PV = Storage provide karta hai**

**PVC = Storage claim karta hai**

**Volume = PVC ko Pod se connect karta hai**

**VolumeMount = Volume ko Container ke folder me mount karta hai** ✅