# Kubernetes Custom Resources (CRD)

## What are Custom Resources?

By default, Kubernetes already knows how to manage built-in resources such as:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- PersistentVolumes
- StatefulSets
- DaemonSets

But sometimes applications need their own Kubernetes resource.

Examples:

- MySQL
- PostgreSQL
- Redis
- Kafka
- Elasticsearch
- Prometheus
- Certificate
- Backup

These resources do **not** exist in Kubernetes by default.

To support them, Kubernetes provides:

- **Custom Resource Definition (CRD)**
- **Custom Resource (CR)**

---

# What is a CRD?

**CRD (Custom Resource Definition)** is a way to extend the Kubernetes API by creating your own resource type.

Think of it as teaching Kubernetes a new object.

Example:

```
Before CRD

Kubernetes knows:

Pod
Deployment
Service
ConfigMap
Secret
```

After installing a CRD:

```
Kubernetes also knows:

MySQL
Redis
Kafka
Certificate
Backup
```

---

# What is a Custom Resource (CR)?

A **Custom Resource (CR)** is an actual object created from a CRD.

Think of it like this:

```
CRD
↓

Defines

↓

MySQL Resource
```

Then you create:

```
kind: MySQL

↓

Custom Resource
```

---

# Why Do We Need Custom Resources?

Without CRDs, deploying MySQL manually requires creating many Kubernetes objects.

Example:

```
Deployment

+

Service

+

PersistentVolumeClaim

+

Secret

+

ConfigMap
```

That's a lot of YAML.

Instead, with a CRD you can simply create:

```yaml
kind: MySQL
```

The rest can be created automatically.

---

# Real-Life Example

Without CRD

```
Developer

↓

Deployment

↓

Service

↓

PVC

↓

Secret

↓

ConfigMap
```

Many YAML files are required.

---

With CRD

```
Developer

↓

MySQL

↓

Operator

↓

Deployment
Service
PVC
Secret
ConfigMap
```

Only one YAML file is needed.

---

# How CRDs Work

```
Developer

↓

Install CRD

↓

Kubernetes API learns new resource

↓

Create Custom Resource

↓

Operator watches it

↓

Creates Kubernetes objects
```

---

# CRD Example

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: mysqls.database.example.com

spec:
  group: database.example.com

  names:
    plural: mysqls
    singular: mysql
    kind: MySQL

  scope: Namespaced

  versions:
    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object
```

---

# Apply the CRD

```bash
kubectl apply -f crd.yml
```

Now Kubernetes understands:

```
kind: MySQL
```

Verify:

```bash
kubectl api-resources
```

Output:

```
mysqls
```

---

# Create a Custom Resource

```yaml
apiVersion: database.example.com/v1
kind: MySQL

metadata:
  name: my-db

spec:
  version: "8.0"
  storage: 10Gi
```

Apply:

```bash
kubectl apply -f mysql.yml
```

Check:

```bash
kubectl get mysqls
```

Output:

```
NAME

my-db
```

---

# Does Creating a Custom Resource Automatically Create Pods?

**No.**

A CRD only tells Kubernetes that a new resource type exists.

It does **not** create Deployments, Pods, or Services by itself.

Example:

```
Developer

↓

MySQL Resource

↓

Stored in etcd

↓

Nothing else happens
```

To automate resource creation, Kubernetes needs an **Operator**.

---

# What is an Operator?

An **Operator** is a Kubernetes controller that watches Custom Resources and performs automation.

Example:

```
Developer

↓

MySQL Resource

↓

MySQL Operator

↓

Deployment

↓

Service

↓

PVC

↓

Secret

↓

ConfigMap
```

The Operator continuously watches the cluster.

Whenever a new MySQL resource is created, it automatically creates and manages everything required.

---

# CRD + Operator Flow

```
Install CRD

↓

Install Operator

↓

Create MySQL Resource

↓

Operator Detects Resource

↓

Creates Deployment

↓

Creates Service

↓

Creates PVC

↓

Creates Secret

↓

Application Running
```

---

# Real Production Examples

## 1. Prometheus Operator

Custom Resource:

```yaml
kind: Prometheus
```

Automatically creates:

- Prometheus Pods
- Services
- ConfigMaps
- ServiceMonitors

---

## 2. Cert Manager

Custom Resource:

```yaml
kind: Certificate
```

Automatically:

- Requests SSL certificates
- Renews certificates
- Creates Kubernetes Secrets

---

## 3. Argo CD

Custom Resource:

```yaml
kind: Application
```

Automatically:

- Pulls code from Git
- Applies Kubernetes manifests
- Keeps the cluster synchronized

---

## 4. Kafka Operator

Custom Resource:

```yaml
kind: Kafka
```

Automatically creates:

- Kafka Brokers
- Services
- Storage
- Networking

---

## 5. Redis Operator

Custom Resource:

```yaml
kind: Redis
```

Automatically creates:

- Redis Pods
- Services
- Storage
- Replication

---

# CRD vs Custom Resource

| CRD | Custom Resource |
|------|-----------------|
| Defines a new resource type | Actual object created from that type |
| Installed once | Created many times |
| Extends Kubernetes API | Uses the new API |
| Example: MySQL Definition | Example: my-db |

---

# CRD vs Operator

| CRD | Operator |
|------|----------|
| Defines a new API | Watches the API |
| Adds resource to Kubernetes | Performs automation |
| Stores objects in etcd | Creates Deployments, Services, PVCs |
| No business logic | Contains business logic |

Think of it like this:

```
CRD

↓

Creates a New Form


Operator

↓

Processes that Form
```

---

# Complete Flow

```
Developer

      │
      ▼

Install CRD

      │
      ▼

Kubernetes API

Learns New Resource

      │
      ▼

Create Custom Resource

      │
      ▼

Operator Detects It

      │
      ▼

Creates Deployment

      │
      ▼

Creates Service

      │
      ▼

Creates PVC

      │
      ▼

Application Starts
```

---

# Useful Commands

## Install CRD

```bash
kubectl apply -f crd.yml
```

---

## Create Custom Resource

```bash
kubectl apply -f mysql.yml
```

---

## List Custom Resources

```bash
kubectl get mysqls
```

---

## Describe Custom Resource

```bash
kubectl describe mysql my-db
```

---

## List Installed CRDs

```bash
kubectl get crd
```

---

## Describe a CRD

```bash
kubectl describe crd mysqls.database.example.com
```

---

## Show All Kubernetes Resources

```bash
kubectl api-resources
```

---

## Delete Custom Resource

```bash
kubectl delete mysql my-db
```

---

## Delete CRD

```bash
kubectl delete crd mysqls.database.example.com
```

---

# Common Mistakes

## 1. Creating a Custom Resource Before Installing the CRD

```bash
kubectl apply -f mysql.yml
```

Error:

```
no matches for kind "MySQL"
```

Reason:

Kubernetes does not know what **MySQL** is yet.

---

## 2. Installing Only the CRD

A CRD only creates a new API.

It does **not** create:

- Pods
- Deployments
- Services
- PVCs

You also need an **Operator**.

---

## 3. Forgetting to Install the Operator

```
CRD

+

Operator

=

Automation
```

Without the Operator, your Custom Resource is simply stored in etcd.

---

# Easy Memory Trick

## CRD

```
Teach Kubernetes a new resource.
```

---

## Custom Resource

```
Create an object from that resource.
```

---

## Operator

```
Watch the object and automate everything.
```

---

# Final Summary

- **CRD (CustomResourceDefinition)** extends the Kubernetes API by adding new resource types.
- **Custom Resource (CR)** is an instance created from a CRD.
- **Operator** watches Custom Resources and automates application management.
- CRDs make Kubernetes extensible, allowing platforms like MySQL, Kafka, Redis, Prometheus, and Cert-Manager to behave like native Kubernetes resources.
- In production, **CRDs and Operators are commonly used together** to automate deployment, scaling, upgrades, backups, and recovery.

---

# One-Line Formula

```
CRD
=
Teach Kubernetes a New Resource

↓

Custom Resource
=
Create an Object

↓

Operator
=
Automate Everything
```