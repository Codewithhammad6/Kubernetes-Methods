# Step 3A - Create Pod (Learning Only)

> **Note:** This file is only for learning how a Pod works.
> In real projects, we usually **do not apply `pod.yml`** because a Deployment automatically creates and manages Pods.

Create **pod.yml**

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  namespace: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

**We will not apply this file.**

```bash
# Not used in this project
# kubectl apply -f pod.yml
```

---

# Step 3B - Create ReplicaSet (Learning Only)

> **Note:** ReplicaSet automatically creates and maintains the required number of Pods.
> In real projects, we usually **do not apply `replicaset.yml`** because a Deployment automatically creates and manages the ReplicaSet.

Create **replicaset.yml**

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-replicaset
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

**We will not apply this file.**

```bash
# Not used in this project
# kubectl apply -f replicaset.yml
```

---

# Step 3C - Create Deployment (Recommended)

> **Note:** This is the recommended and most commonly used approach in real projects.
> Deployment automatically creates a ReplicaSet, and the ReplicaSet automatically creates Pods.

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

Apply the Deployment

```bash
kubectl apply -f deployment.yml
```

---

# Real Kubernetes Hierarchy

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
Containers
```

---

# Resource Relationship

```
Pod
 │
 └── 1 or More Containers

ReplicaSet
 │
 └── Multiple Pods

Deployment
 │
 └── ReplicaSet
      │
      └── Multiple Pods
```

---

# Which Resource Should We Use?

| Resource | Learning | Production |
|----------|----------|------------|
| Pod | ✅ Yes | ❌ No |
| ReplicaSet | ✅ Yes | ❌ Rarely |
| Deployment | ✅ Yes | ✅ Yes |

**Conclusion:**

- **Pod** → Learn how a single Pod works.
- **ReplicaSet** → Learn how Kubernetes maintains the desired number of Pods.
- **Deployment** → Used in almost all real Kubernetes projects because it manages ReplicaSets, supports scaling, Rolling Updates, and Rollbacks automatically.