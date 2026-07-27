````md id="service-readme"
# Kubernetes Service

## Objective

Learn how a **Service** works in Kubernetes and why it is required.

---

# What is a Service?

A **Service** is a Kubernetes resource that provides **network access** to Pods and gives them a **stable identity (fixed IP and DNS name)**.

In simple words:

> **Pods are temporary, but a Service provides a permanent address.**

---

# Why Do We Need a Service?

Suppose you have a Deployment.

```yaml
replicas: 2
```

It creates two Pods.

```
Deployment
      │
      ▼
 ReplicaSet
      │
      ▼
 ┌───────────────┐
 │               │
 ▼               ▼
Pod-1         Pod-2
nginx         nginx
```

Now suppose Pod-1 crashes.

```
Pod-1 ❌
```

Kubernetes automatically creates a new Pod.

```
Pod-3 ✅
```

The problem is that the Pod gets a new IP address.

Example

```
Pod-1

IP: 10.244.1.5

Deleted

↓

Pod-3

IP: 10.244.2.8
```

If users connect directly to the Pod IP, the application will stop working because the Pod IP has changed.

---

# Solution

A **Service** provides a fixed IP and DNS name.

```
User
 │
 ▼
Service
(Fixed IP)
 │
 ├──────────────┐
 ▼              ▼
Pod-1        Pod-2
```

Even if Pods are deleted and recreated, the Service IP remains the same.

---

# Service YAML

Create **service.yml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: nginx

spec:
  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: ClusterIP

```

Apply

```bash
kubectl apply -f service.yml
```

---

# Understanding the YAML

## selector

```yaml
selector:
  app: nginx
```

This tells Kubernetes:

> Send traffic to every Pod whose label is:

```yaml
app: nginx
```

Example

Deployment

```yaml
labels:
  app: nginx
```

Service automatically finds those Pods.

```
Service
     │
     ▼

app: nginx

     │
     ▼

Pod-1

Pod-2
```

---

## ports

```yaml
ports:
- port: 80
  targetPort: 80
```

### port

The port exposed by the Service.

```
Service

Port 80
```

---

### targetPort

The port on which the container is listening.

```
Container

Port 80
```

Traffic Flow

```
User

↓

Service :80

↓

Container :80
```

---

# Complete Network Flow

```
Browser
      │
      ▼
nginx-service
      │
      ▼
Service
10.96.0.20
      │
      ├─────────────┐
      ▼             ▼
Pod-1           Pod-2
```

---

# Load Balancing

Suppose you have:

```
Deployment

Replicas = 3
```

```
Service
      │
      ├──────────────┐
      │              │
      ▼              ▼
   Pod-1          Pod-2
         │
         ▼
      Pod-3
```

When multiple requests arrive:

```
Request 1 → Pod-1

Request 2 → Pod-2

Request 3 → Pod-3

Request 4 → Pod-1

Request 5 → Pod-2
```

The Service distributes traffic across available Pods.

---

# Service Types

## 1. ClusterIP (Default)

Accessible only inside the Kubernetes cluster.

```
Pod

↓

Service

↓

Pod
```

Use Cases

- Backend APIs
- Databases
- Internal Microservices

---

## 2. NodePort

Opens a port on every Worker Node.

```
User

↓

EC2-IP:30080

↓

NodePort Service

↓

Pod
```

Use Cases

- Testing
- Development
- Accessing applications from outside the cluster

---

## 3. LoadBalancer

Creates a cloud load balancer.

```
Internet
      │
      ▼
AWS / Azure / GCP
Load Balancer
      │
      ▼
Service
      │
      ▼
Pods
```

Use Cases

- Production applications
- Public websites
- APIs

---

# Deployment vs Service

```
Deployment

↓

Creates Pods
```

```
Service

↓

Provides Network Access
```

Deployment manages Pods.

Service provides access to those Pods.

---

# Complete Architecture

```
Browser
      │
      ▼
Service
      │
      ▼
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pods
      │
      ▼
Containers
```

---

# Useful Commands

Apply

```bash
kubectl apply -f service.yml
```

Check Services

```bash
kubectl get services -n nginx
```

Describe Service

```bash
kubectl describe service nginx-service -n nginx
```

Delete Service

```bash
kubectl delete service nginx-service -n nginx
```

---

# Interview Question

## Why do we need a Service in Kubernetes?

**Answer**

Pods are temporary and their IP addresses can change whenever they are recreated.

A **Service** provides a stable IP address and DNS name, allowing users and applications to access Pods without worrying about changing Pod IPs.

---

# Resource Flow

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
Service
      │
      ▼
Users / Other Applications
```

---

# One-Line Revision

- **Pod** → Temporary workload.
- **Deployment** → Creates and manages Pods.
- **Service** → Provides a stable IP and DNS name for Pods.
- **selector** → Selects Pods using labels.
- **port** → Service port.
- **targetPort** → Container port.
- **ClusterIP** → Internal access only.
- **NodePort** → External access through a Node port.
- **LoadBalancer** → External access using a cloud load balancer.
````
