# Kubernetes Resources (CPU, Memory & Storage)

## What are Resources?

In Kubernetes, **Resources** define how much **CPU** and **Memory (RAM)** a container needs and how much it is allowed to use.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Here, `resources` means:

* CPU
* Memory (RAM)

These resources come directly from the **Node (Host Machine)**.

---

# Resource Types

There are three commonly used resources in Kubernetes.

```yaml
resources:
  requests:
    storage: 5Gi
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

However, there is one important difference.

* **CPU** → Comes from the Node (Host Machine)
* **Memory (RAM)** → Comes from the Node (Host Machine)
* **Storage** → Comes through **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)**

---

# CPU

Example:

```yaml
cpu: "250m"
```

CPU is provided by the **Node (Host Machine)**.

Example:

Host Machine

```text
CPU = 4 Cores
```

Pod Request

```text
250m = 0.25 Core
```

Diagram

```text
Host Machine

CPU (4 Cores)
      │
      ▼
Pod

CPU Request = 250m
```

---

# Memory (RAM)

Example:

```yaml
memory: "256Mi"
```

Memory also comes from the **Node (Host Machine)**.

Example:

Host Machine

```text
RAM = 8 GB
```

Pod Request

```text
256 MiB
```

Diagram

```text
Host Machine

RAM (8 GB)
      │
      ▼
Pod

Memory Request = 256Mi
```

---

# Storage

Example:

```yaml
storage: 5Gi
```

Storage does **not** come directly from the Node like CPU and RAM.

A Pod receives storage through a **PVC**, which is connected to a **PV**.

Flow

```text
Persistent Volume (PV)

        │

        ▼

Persistent Volume Claim (PVC)

        │

        ▼

Pod
```

---

# Local Kubernetes

If you are using **Kind**, **Minikube**, or **kubeadm** with local storage:

```text
Host Disk
      │
      ▼
Persistent Volume (PV)
      │
      ▼
Persistent Volume Claim (PVC)
      │
      ▼
Pod
```

Here, the storage may come from the host machine's disk.

---

# Cloud Kubernetes

On cloud platforms, storage usually comes from cloud storage services.

Example:

```text
AWS EBS
      │
      ▼
Persistent Volume (PV)
      │
      ▼
Persistent Volume Claim (PVC)
      │
      ▼
Pod
```

Similarly,

* Azure → Azure Disk
* Google Cloud → Persistent Disk

---

# Complete Resource Flow

```text
                 Node (Host Machine)

      CPU ----------------------► Pod

      RAM ----------------------► Pod

      Disk
        │
        ▼
 Persistent Volume (PV)
        │
        ▼
 Persistent Volume Claim (PVC)
        │
        ▼
        Pod
```

---

# requests

`requests` tells Kubernetes the **minimum resources** required by the Pod.

Example:

```yaml
requests:
  cpu: "250m"
  memory: "256Mi"
```

Meaning:

* Reserve **0.25 CPU Core**
* Reserve **256Mi RAM**

If the Node does not have these resources available, the Pod will remain **Pending**.

---

# limits

`limits` defines the **maximum resources** a Pod is allowed to use.

Example:

```yaml
limits:
  cpu: "500m"
  memory: "512Mi"
```

Meaning:

* Maximum CPU = **0.5 Core**
* Maximum Memory = **512Mi**

The container cannot use more than these limits.

---

# requests vs limits

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Result:

```text
Minimum CPU    = 250m

Maximum CPU    = 500m

Minimum Memory = 256Mi

Maximum Memory = 512Mi
```

---

# Example

Suppose your Host Machine has:

```text
CPU = 4 Cores

RAM = 8 GB
```

Pod Configuration

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Kubernetes reserves:

* 0.25 CPU Core
* 256Mi RAM

The Pod can use up to:

* 0.5 CPU Core
* 512Mi RAM

---

# If Resources Are Not Available

Suppose your Node has:

```text
RAM = 2 GB
```

But your Pod requests:

```yaml
requests:
  memory: 4Gi
```

Result:

```text
Pod Status

Pending
```

Reason:

The Node does not have enough RAM to satisfy the requested resources.

---

# Real Company Usage

Almost every production Deployment includes resource requests and limits.

Example:

* Nginx
* React
* Node.js
* Django
* Spring Boot
* Python APIs
* Go Services

This prevents one application from consuming all CPU or Memory on a Node.

---

# Quick Revision

```
CPU
        │
        ▼
Host Machine

RAM
        │
        ▼
Host Machine

Storage
        │
        ▼
PV → PVC → Pod
```

---

# One-Line Revision

* **CPU** → Comes from the Node (Host Machine).
* **Memory (RAM)** → Comes from the Node (Host Machine).
* **Storage** → Comes through PV and PVC.
* **requests** → Minimum resources required by the Pod.
* **limits** → Maximum resources the Pod can use.
* **If requests cannot be satisfied** → Pod remains **Pending**.











---

# Resources with Multiple Replicas

Suppose your StatefulSet has:

```yaml
replicas: 3
```

Storage for each Pod:

```yaml
volumeClaimTemplates:
  resources:
    requests:
      storage: 5Gi
```

Container Resources:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Three Pods will be created:

```text
mongo-0
mongo-1
mongo-2
```

Each Pod gets:

| Pod     | Storage | CPU Request | Memory Request |
| ------- | ------- | ----------- | -------------- |
| mongo-0 | 5Gi     | 250m        | 256Mi          |
| mongo-1 | 5Gi     | 250m        | 256Mi          |
| mongo-2 | 5Gi     | 250m        | 256Mi          |

---

# Total Minimum Resources

Storage

```text
5Gi + 5Gi + 5Gi = 15Gi
```

CPU Requests

```text
250m + 250m + 250m = 750m
(0.75 CPU Core)
```

Memory Requests

```text
256Mi + 256Mi + 256Mi = 768Mi
```

---

# Maximum Resources (Limits)

If all Pods use their maximum limits:

CPU

```text
500m × 3 = 1500m
(1.5 CPU Cores)
```

Memory

```text
512Mi × 3 = 1536Mi
≈ 1.5Gi RAM
```

---

# Example

Node Resources:

```text
CPU  = 2 Cores
RAM  = 4Gi
Disk = 100Gi
```

Kubernetes checks:

* Is **15Gi Storage** available?
* Is **750m CPU** available?
* Is **768Mi RAM** available?

If resources are available, all Pods will be **Running**.

---

# Important Note

`5Gi` means the **maximum storage available for each Pod**, not that the Pod immediately uses 5Gi.

Example:

```text
mongo-0 → 500MB

mongo-1 → 1GB

mongo-2 → 200MB
```

Each Pod can still use **up to 5Gi** whenever needed.





# Kubernetes Resource Requests & Limits

## 1. Experience (Sabse common)

Shuru mein estimate lagaya jata hai.

### Example

### Frontend (Nginx + React)

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

### Kyun?

- Static files hain.
- CPU bahut kam use hoti hai.
- RAM bhi kam lagti hai.

---

### Backend (Node.js/Django)

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"

  limits:
    cpu: "1000m"
    memory: "1Gi"
```

Backend API requests process karta hai, isliye frontend se zyada resources chahiye.

---

### MySQL

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"

  limits:
    cpu: "2"
    memory: "2Gi"
```

Database caching karta hai, isliye RAM zyada chahiye hoti hai.

---

# 2. Monitoring (Production)

Application deploy karte hain.

Phir dekhte hain:

```bash
kubectl top pods
```

### Example

```text
frontend
CPU 30m
RAM 90Mi

backend
CPU 450m
RAM 420Mi

mysql
CPU 120m
RAM 700Mi
```

Ab samajh aa gaya:

- Frontend sirf **90Mi** use kar raha hai.

To request:

```yaml
requests:
  memory: 128Mi
```

Kaafi hai.