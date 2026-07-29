# Kubernetes Scheduling (Labels, Node Selector, Taints & Tolerations)

Kubernetes Scheduler ka kaam hota hai:

> **"Kis Node par kaunsa Pod chalana hai?"**

Iske liye Kubernetes mein ye concepts use hote hain:

- Labels
- Node Selector
- Taints
- Tolerations
- Node Affinity (Advanced)

Learning ke liye pehle **Labels + Node Selector**, phir **Taints & Tolerations**, aur baad mein **Node Affinity** seekhna best approach hai.

---

# Project Architecture

Suppose tumhare cluster mein 2 Worker Nodes hain.

```text
mycluster-worker
↓
Frontend + MySQL

mycluster-worker2
↓
Backend
```

Aur tum chahte ho:

- Frontend aur MySQL sirf Worker-1 par chale.
- Backend sirf Worker-2 par chale.
- Koi aur Pod Worker-2 par schedule na ho.

---

# Step 1 - Add Labels to Nodes

Frontend aur MySQL wale node par label lagao.

```bash
kubectl label node mycluster-worker role=frontend-db
```

Backend wale node par label lagao.

```bash
kubectl label node mycluster-worker2 role=backend
```

Check labels:

```bash
kubectl get nodes --show-labels
```

Example Output:

```text
mycluster-worker
role=frontend-db

mycluster-worker2
role=backend
```

---

# Step 2 - Add Taint on Backend Node

Backend node ko protect karo.

```bash
kubectl taint nodes mycluster-worker2 prod=true:NoSchedule
```

Check:

```bash
kubectl describe node mycluster-worker2
```

Output:

```text
Taints:

prod=true:NoSchedule
```

Ab koi normal Pod Worker-2 par schedule nahi ho sakta.

---

# Step 3 - Frontend Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: frontend-db

      containers:
      - name: frontend
        image: hammadch123/frontendnotes:v1
```

Meaning:

> Frontend Pod sirf us Node par chalega jahan label `role=frontend-db` hoga.

---

# Step 4 - MySQL Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: frontend-db

      containers:
      - name: mysql
        image: mysql:8.0
```

Meaning:

> MySQL Pod bhi sirf Worker-1 par chalega.

---

# Step 5 - Backend Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: backend

      tolerations:
      - key: "prod"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

      containers:
      - name: backend
        image: hammadch123/backendnotes:v1
```

Meaning:

- `nodeSelector` → Backend Pod ko backend node par bhejo.
- `tolerations` → Backend Pod ko tainted node par chalne ki permission do.

---

# Final Architecture

```text
                Kubernetes Cluster

        ┌─────────────────────────────────┐
        │                                 │
        │        Control Plane            │
        │                                 │
        └──────────────┬──────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
┌──────────────────────┐     ┌─────────────────────────┐
│ mycluster-worker      │     │ mycluster-worker2       │
│                       │     │                         │
│ Label                 │     │ Label                  │
│ role=frontend-db      │     │ role=backend           │
│                       │     │                        │
│                       │     │ Taint                  │
│                       │     │ prod=true:NoSchedule   │
│                       │     │                        │
│ Frontend Pod          │     │ Backend Pod            │
│ Frontend Pod          │     │ Backend Pod            │
│ MySQL Pod             │     │ Backend Pod            │
└──────────────────────┘     └─────────────────────────┘
```

---

# What Happens if Another Deployment Comes?

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

spec:
  replicas: 2

  template:
    spec:
      containers:
      - name: nginx
        image: nginx
```

Is Deployment mein:

- ❌ nodeSelector nahi hai.
- ❌ toleration nahi hai.

Result:

```text
Worker-1 ✅ Allowed

Worker-2 ❌ Not Allowed
(Because Worker-2 par taint laga hua hai.)
```

---

# What Happens if Backend Toleration is Removed?

Backend Deployment:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: backend
```

Result:

```text
Backend Pod

↓

Worker-2 par jana chahta hai

↓

Worker-2 tainted hai

↓

Permission nahi mili

↓

Pod Pending ❌
```

---

# Taint Effects

## 1. NoSchedule

```text
Naye Pods schedule nahi honge.

Existing Pods chalti rahengi.
```

---

## 2. PreferNoSchedule

```text
Kubernetes koshish karega ke Pod is Node par na bheje.

Agar zarurat ho to schedule kar sakta hai.
```

---

## 3. NoExecute

```text
Naye Pods schedule nahi honge.

Purane Pods bhi remove kar diye jayenge.
```

---

# Labels vs Node Selector vs Taints vs Tolerations

## Label

Node ko identity deta hai.

Example:

```text
role=frontend-db

role=backend
```

---

## Node Selector

Pod ko specific label wale Node par schedule karta hai.

Example:

```yaml
nodeSelector:
  role: backend
```

---

## Taint

Node ko normal Pods se protect karta hai.

Example:

```bash
kubectl taint nodes mycluster-worker2 prod=true:NoSchedule
```

---

## Toleration

Sirf selected Pods ko tainted Node par chalne ki permission deta hai.

Example:

```yaml
tolerations:
- key: "prod"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```

---

# Production Use Cases

Ye scheduling concepts production mein bahut use hote hain.

Examples:

- Backend Nodes
- Database Nodes
- GPU Nodes
- Monitoring Nodes
- Logging Nodes

Har workload ko dedicated node par run karna ho to Labels, Node Selector, Taints aur Tolerations use kiye jate hain.

---

# Formula Yaad Rakho

```text
Label
↓
Node ki identity

nodeSelector
↓
Pod ko kis Node par bhejna hai

Taint
↓
Node ko protect karna hai

Toleration
↓
Kis Pod ko us protected Node par allow karna hai
```

---

# Interview Summary

- **Label** → Node ko identity deta hai.
- **nodeSelector** → Pod ko specific label wale Node par schedule karta hai.
- **Taint** → Node ko normal Pods se protect karta hai.
- **Toleration** → Sirf selected Pods ko tainted Node par chalne ki permission deta hai.
- **Node Affinity** → Node Selector ka advanced aur flexible version.

---

# One-Line Summary

```text
Label → Node ki pehchan

nodeSelector → Pod ko specific Node par bhejta hai

Taint → Node ko protect karta hai

Toleration → Protected Node par specific Pod ko allow karti hai

Node Affinity → Advanced scheduling rules ke liye use hoti hai.
```