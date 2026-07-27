# Kubernetes DaemonSet

## Objective

Learn how **DaemonSet** works, when to use it, and how it differs from a Deployment.

---

# What is a DaemonSet?

A **DaemonSet** is a Kubernetes resource that ensures **one Pod runs on every Worker Node** in the cluster.

Unlike a Deployment, **DaemonSet does not use `replicas`**.

Its rule is:

> **"Run one Pod on every Worker Node."**

---

# DaemonSet Architecture

```
              DaemonSet
                   │
        ┌──────────┴──────────┐
        │                     │
    Worker-1             Worker-2
        │                     │
      1 Pod                 1 Pod
```

---

# Example 1

Cluster:

```
1 Master
1 Worker
```

DaemonSet

```
DaemonSet
     │
     ▼
Worker-1
└── Pod
```

Result

```
1 Worker = 1 Pod
```

---

# Example 2

Cluster:

```
1 Master
3 Workers
```

DaemonSet

```
DaemonSet
     │
     ├── Worker-1
     │      └── Pod
     │
     ├── Worker-2
     │      └── Pod
     │
     └── Worker-3
            └── Pod
```

Result

```
3 Workers = 3 Pods
```

---

# Example 3 - New Worker Added

Before

```
Worker-1 → Pod

Worker-2 → Pod

Worker-3 → Pod
```

A new Worker joins the cluster

```
Worker-4
```

DaemonSet automatically creates a Pod

```
Worker-1 → Pod

Worker-2 → Pod

Worker-3 → Pod

Worker-4 → Pod ✅
```

No manual action is required.

---

# Example 4 - Worker Removed

Before

```
Worker-1 → Pod

Worker-2 → Pod

Worker-3 → Pod
```

Worker-3 is deleted

```
Worker-1 → Pod

Worker-2 → Pod
```

DaemonSet automatically removes the Pod from the deleted node.

---

# DaemonSet YAML

Create **daemonset.yml**

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx-daemonset
  namespace: nginx

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      name: nginx-dmn-pod
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80
```

Apply

```bash
kubectl apply -f daemonset.yml
```

---

# Verify DaemonSet

Check DaemonSets

```bash
kubectl get daemonsets -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Describe DaemonSet

```bash
kubectl describe daemonset nginx-daemonset -n nginx
```

Delete DaemonSet

```bash
kubectl delete daemonset nginx-daemonset -n nginx
```

---

# Real Use Cases

DaemonSet is **not used for normal applications** like Frontend or Backend.

It is used for software that must run on **every Worker Node**.

## 1. Monitoring

Collect CPU, RAM and Disk metrics.

Example

- Prometheus Node Exporter
- Datadog Agent
- New Relic Agent

```
Worker-1 → Node Exporter

Worker-2 → Node Exporter

Worker-3 → Node Exporter
```

---

## 2. Logging

Collect logs from every Worker Node.

Examples

- Fluent Bit
- Fluentd
- Filebeat

```
Worker-1 → Fluent Bit

Worker-2 → Fluent Bit

Worker-3 → Fluent Bit
```

---

## 3. Security

Run security agents on every Worker.

Examples

- Falco
- CrowdStrike Agent
- Sysdig Agent

```
Worker-1 → Security Agent

Worker-2 → Security Agent

Worker-3 → Security Agent
```

---

## 4. Networking

Many Kubernetes networking plugins run as DaemonSets.

Examples

- Calico
- Cilium
- Weave Net

---

# Deployment vs DaemonSet

| Feature | Deployment | DaemonSet |
|----------|------------|-----------|
| Pods | Fixed number of replicas | One Pod per Worker Node |
| Uses `replicas` | ✅ Yes | ❌ No |
| Auto-create Pod on new Worker | ❌ No | ✅ Yes |
| Best For | Frontend, Backend, APIs | Monitoring, Logging, Security, Networking |

---

# Example

Cluster

```
1 Master

2 Workers
```

Deployment

```
replicas: 4
```

Result

```
Worker-1
├── Nginx Pod
└── Chat Pod

Worker-2
├── Auth Pod
└── Redis Pod
```

DaemonSet (Fluent Bit)

```
Worker-1
├── Nginx Pod
├── Chat Pod
└── Fluent Bit Pod

Worker-2
├── Auth Pod
├── Redis Pod
└── Fluent Bit Pod
```

Notice:

- Nginx, Chat, Auth and Redis Pods are created by **Deployment**.
- Fluent Bit Pods are created by **DaemonSet**.

---

# Real Kubernetes Hierarchy

```
DaemonSet
     │
     └── One Pod Per Worker Node
              │
              ▼
          Container
```

---

# Quick Comparison

| Resource | Rule |
|----------|------|
| Pod | Create one Pod |
| ReplicaSet | Maintain a fixed number of Pods |
| Deployment | Manage ReplicaSets and Pods |
| DaemonSet | Run one Pod on every Worker Node |

---

# Useful Commands

Check DaemonSets

```bash
kubectl get daemonsets -A
```

Check Pods

```bash
kubectl get pods -o wide
```

Describe DaemonSet

```bash
kubectl describe daemonset nginx-daemonset -n nginx
```

Delete DaemonSet

```bash
kubectl delete daemonset nginx-daemonset -n nginx
```

---

# Learning Summary

✅ Learned DaemonSet

✅ One Pod runs on every Worker Node

✅ New Worker automatically gets a Pod

✅ Deleted Worker automatically removes its Pod

✅ Learned Monitoring use case

✅ Learned Logging use case

✅ Learned Security use case

✅ Learned Networking use case

---

# One-Line Revision

- **Deployment:** "I need **5 total Pods**."
- **DaemonSet:** "I need **1 Pod on every Worker Node**."