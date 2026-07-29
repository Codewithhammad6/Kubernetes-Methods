# Kubernetes Scheduling (Node Selector, Taints & Tolerations)

Kubernetes mein **Scheduling** ka matlab hai:

> **"Kis Node par kaunsa Pod chalega?"**

Iske liye Kubernetes mein 3 common concepts use hote hain:

- Node Selector
- Taints & Tolerations
- Node Affinity

Learning ke liye pehle **Node Selector**, phir **Taints & Tolerations**, aur baad mein **Node Affinity** seekhna best approach hai.

---

# 1. Node Selector

Node Selector ka use tab hota hai jab aap chahte ho ke koi Pod sirf ek specific Node par hi chale.

## Step 1 - Node Labels

Frontend aur MySQL wale node (`mycluster-worker`):

```bash
kubectl label node mycluster-worker role=frontend-db
```

Backend wale node (`mycluster-worker2`):

```bash
kubectl label node mycluster-worker2 role=backend
```

Check labels:

```bash
kubectl get nodes --show-labels
```

---

## Frontend Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: frontend-db
```

Iska matlab:

> Frontend Pod sirf us Node par chalega jahan label `role=frontend-db` hoga.

---

## MySQL Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: frontend-db
```

Iska matlab:

> MySQL Pod bhi usi Node par chalega jahan label `role=frontend-db` hoga.

---

## Backend Deployment

```yaml
spec:
  template:
    spec:
      nodeSelector:
        role: backend
```

Iska matlab:

> Backend Pod sirf us Node par chalega jahan label `role=backend` hoga.

---

## Result

```text
Worker-1
├── Frontend
├── Frontend
└── MySQL

Worker-2
├── Backend
├── Backend
└── Backend
```

---

## Verify

```bash
kubectl get pods -o wide -n notes-app
```

Example:

```text
NAME                    NODE

frontend-xxxxx          mycluster-worker
frontend-yyyyy          mycluster-worker

mysql-xxxxx             mycluster-worker

backend-xxxxx           mycluster-worker2
backend-yyyyy           mycluster-worker2
backend-zzzzz           mycluster-worker2
```

---

# 2. Taints & Tolerations

## Taint kya hota hai?

**Taint Node par lagta hai.**

Ye Kubernetes ko bolta hai:

> **"Is Node par har Pod mat bhejo."**

Example:

```bash
kubectl taint nodes mycluster-worker app=database:NoSchedule
```

Matlab:

> Sirf database wale Pods is Node par chal sakte hain.

---

## Toleration kya hoti hai?

**Toleration Pod par lagti hai.**

Ye Kubernetes ko bolti hai:

> **"Ye Pod is Taint ko tolerate karta hai."**

Example:

```yaml
tolerations:
- key: "app"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"
```

Ab Database Pod us Node par schedule ho sakta hai.

---

## Taint Effects

### NoSchedule

```text
Naye Pods schedule nahi honge.
```

Existing Pods chalti rahengi.

---

### PreferNoSchedule

```text
Kubernetes koshish karega ke Pod is Node par na bheje.

Lekin zarurat ho to schedule kar sakta hai.
```

---

### NoExecute

```text
Naye Pods schedule nahi honge.

Purane Pods bhi remove kar diye jayenge.
```

---

# Database ko Specific Node par Chalana

Agar MySQL, MongoDB ya koi bhi database **ek specific Node par chalana ho**, to sirf PVC par depend nahi karna chahiye.

Production mein aksar:

- Pehle specific Node par PV banaya jata hai.
- Phir PVC us PV ko claim karti hai.
- Phir Database Pod us PVC ko mount karta hai.

Flow:

```text
Specific Worker Node
        │
        ▼
Persistent Volume (PV)
        │
        ▼
PersistentVolumeClaim (PVC)
        │
        ▼
MySQL / MongoDB Pod
```

---

## Example PV

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mysql-pv

spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/mysql

  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - mycluster-worker
```

### Meaning

```text
PV sirf mycluster-worker Node par available hai.

Jab PVC is PV ko claim karegi,
to Database Pod bhi usi Node par schedule hoga.
```

---

# Interview Notes

**Node Selector**

> Pod ko specific Node par chalana.

**Taint**

> Node par restriction lagana.

**Toleration**

> Pod ko restricted Node par chalne ki permission dena.

**Node Affinity**

> Node Selector ka advanced aur flexible version.

---

# Final Summary

```text
Node Selector → Pod ko specific Node par schedule karta hai.

Taint → Node par restriction lagata hai.

Toleration → Specific Pod ko us restricted Node par chalne deta hai.

Node Affinity → Advanced scheduling ke liye use hoti hai.

Database (MongoDB, MySQL) → Aksar specific Node + PV + PVC ke saath deploy ki jati hai taake data aur storage stable rahe.
```