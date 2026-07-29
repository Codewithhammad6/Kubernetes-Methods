# Kubernetes StatefulSet

## What is a StatefulSet?

A **StatefulSet** is a Kubernetes resource used to deploy **stateful applications**.

Unlike a Deployment, a StatefulSet gives every Pod:

* A **fixed name**
* A **stable network identity**
* Its own **persistent storage**

It is mainly used for applications where **data must not be lost**.

---

# Why Do We Need StatefulSet?

Suppose you have a MERN application.

```text
React Frontend
        │
Node.js Backend
        │
MongoDB
```

For development, a single MongoDB instance is enough.

```text
React
   │
Node.js
   │
MongoDB
```

A **Deployment + PVC** can work for this.

---

# Production Example

In production, companies want the application to keep running even if one database server fails.

Instead of one MongoDB Pod, they create multiple database Pods.

```text
             Node.js
                │
      ┌─────────┼─────────┐
      │         │         │
   mongo-0   mongo-1   mongo-2
```

Each MongoDB Pod stores different data.

This is why **StatefulSet** is used.

---

# Why Not Deployment?

A Deployment creates Pods like:

```text
mongo-abcd

mongo-xzy1

mongo-1234
```

If a Pod is deleted:

```text
mongo-abcd ❌
```

Kubernetes creates:

```text
mongo-xyz99
```

The Pod name changes.

For databases, changing Pod names and identities can cause problems.

---

# StatefulSet Solution

StatefulSet creates Pods with fixed names.

Example:

```text
mongo-0

mongo-1

mongo-2
```

If:

```text
mongo-1
```

is deleted,

Kubernetes recreates:

```text
mongo-1
```

The Pod keeps the same identity.

---

# Headless Service

Before creating a StatefulSet, we create a **Headless Service**.

Create **service.yml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: mysql

spec:
  clusterIP: None

  selector:
    app: mysql

  ports:
  - name: mysql
    protocol: TCP
    port: 3306
    targetPort: 3306
```

Notice:

```yaml
clusterIP: None
```

This makes it a **Headless Service**.

A Headless Service gives every StatefulSet Pod its own DNS name.

---

# Converting Deployment into StatefulSet

Normally you may have:

```text
deployment.yml
```

When using StatefulSet, you can rename it to:

```text
statefulset.yml
```

The filename is optional and only helps with project organization.

---

## Step 1

Change

```yaml
kind: Deployment
```

to

```yaml
kind: StatefulSet
```

---

## Step 2

Add:

```yaml
spec:
  serviceName: mysql-service
```

The **serviceName** must match the Headless Service.

Example:

```yaml
spec:
  serviceName: mysql-service
```

---

## Step 3

The Headless Service should contain:

```yaml
clusterIP: None
```

---

## Step 4

If every Pod needs its own storage, add:

```yaml
volumeClaimTemplates:
```

Example:

```yaml
volumeClaimTemplates:

- metadata:
    name: mysql-storage

  spec:
    accessModes:
      - ReadWriteOnce

    resources:
      requests:
        storage: 5Gi
```

StatefulSet automatically creates a separate PVC for every Pod.

Example:

```text
mysql-0  → PVC-0

mysql-1  → PVC-1

mysql-2  → PVC-2
```

Every Pod gets its own storage.

---

# Complete Flow

```text
Node.js Backend
        │
        ▼
Headless Service
(clusterIP: None)
        │
        ▼
StatefulSet
        │
        ├──────────────┬──────────────┐
        ▼              ▼              ▼

     mysql-0       mysql-1       mysql-2
        │              │              │
        ▼              ▼              ▼

      PVC-0          PVC-1          PVC-2
        │              │              │
        ▼              ▼              ▼

 Persistent Storage  Persistent Storage  Persistent Storage
```

---

# If a Pod is Deleted

Suppose:

```bash
kubectl delete pod mysql-1
```

Kubernetes automatically recreates:

```text
mysql-1
```

and attaches the same PVC.

```
mysql-1

↓

PVC-1

↓

Old Data Restored
```

The application continues to work.

---

# Real Company Examples

StatefulSet is commonly used for:

* MongoDB
* MySQL
* PostgreSQL
* MariaDB
* Redis (Persistent Mode)
* Kafka
* ZooKeeper
* Elasticsearch
* Cassandra

These applications need stable identities and persistent data.

---

# Deployment vs StatefulSet

| Deployment                         | StatefulSet                                    |
| ---------------------------------- | ---------------------------------------------- |
| Used for stateless applications    | Used for stateful applications                 |
| Pod names can change               | Pod names remain fixed                         |
| Shared or simple storage           | Separate storage for every Pod                 |
| Suitable for React, Node.js, Nginx | Suitable for MongoDB, MySQL, PostgreSQL, Kafka |

---

# MERN Project Architecture

```text
React Frontend
        │
        ▼
Node.js Backend
        │
        ▼
Headless Service
        │
        ▼
MongoDB StatefulSet
        │
        ├──────────────┬──────────────┐
        ▼              ▼              ▼

    mongo-0       mongo-1       mongo-2
        │              │              │
        ▼              ▼              ▼

      PVC-0          PVC-1          PVC-2
        │              │              │
        ▼              ▼              ▼

 Persistent Storage  Persistent Storage  Persistent Storage
```

---

# Deployment → StatefulSet Checklist

When converting a Deployment to a StatefulSet:

* ✅ Change `kind: Deployment` → `kind: StatefulSet`
* ✅ Add `spec.serviceName`
* ✅ Create a Headless Service (`clusterIP: None`)
* ✅ Use `volumeClaimTemplates` for per-Pod storage
* ✅ Keep the existing `selector` and `template`

---

# One-Line Revision

* **Deployment** → React, Node.js, Nginx (Stateless Applications)
* **StatefulSet** → MongoDB, MySQL, PostgreSQL, Kafka (Stateful Applications)
* **Headless Service** → Gives every Pod a stable DNS name.
* **volumeClaimTemplates** → Creates one PVC for each Pod automatically.
* **StatefulSet** → Fixed Pod names + Stable Network Identity + Persistent Storage.
