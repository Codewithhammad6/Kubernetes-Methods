# Kubernetes RBAC Project

This project demonstrates how to implement **Role-Based Access Control (RBAC)** in Kubernetes using the following resources:

- Namespace
- ServiceAccount
- Role
- RoleBinding
- Deployment
- Service

---

# Project Structure

```text
k8s/
├── namespace.yml
├── service-account.yml
├── role.yml
├── rolebinding.yml
├── deployment.yml
└── service.yml
```

---

# Prerequisites

Verify your Kubernetes cluster before deploying.

```bash
kubectl version
kubectl cluster-info
kubectl get nodes
```

---

# Deployment Flow

```text
Namespace
    │
    ▼
ServiceAccount
    │
    ▼
Role
    │
    ▼
RoleBinding
    │
    ▼
Deployment
    │
    ▼
Service
```

---

# Step 1 — Create Namespace

Apply

```bash
kubectl apply -f namespace.yml
```

Verify

```bash
kubectl get ns
```

Describe

```bash
kubectl describe namespace apache
```

---

# Step 2 — Create ServiceAccount

Apply

```bash
kubectl apply -f service-account.yml
```

Verify

```bash
kubectl get sa -n apache
```

Describe

```bash
kubectl describe sa apache-user -n apache
```

---

# Step 3 — Create Role

Apply

```bash
kubectl apply -f role.yml
```

Verify

```bash
kubectl get roles -n apache
```

Describe

```bash
kubectl describe role apache-manager -n apache
```

---

# Step 4 — Create RoleBinding

Apply

```bash
kubectl apply -f rolebinding.yml
```

Verify

```bash
kubectl get rolebindings -n apache
```

Describe

```bash
kubectl describe rolebinding apache-manager-binding -n apache
```

---

# Step 5 — Create Deployment

Apply

```bash
kubectl apply -f deployment.yml
```

Verify Deployment

```bash
kubectl get deployments -n apache
```

Verify Pods

```bash
kubectl get pods -n apache
```

Detailed View

```bash
kubectl get pods -o wide -n apache
```

Describe Deployment

```bash
kubectl describe deployment apache-deployment -n apache
```

Describe Pod

```bash
kubectl describe pod <pod-name> -n apache
```

View Logs

```bash
kubectl logs <pod-name> -n apache
```

Execute Inside Pod

```bash
kubectl exec -it <pod-name> -n apache -- sh
```

---

# Step 6 — Create Service

Apply

```bash
kubectl apply -f service.yml
```

Verify

```bash
kubectl get svc -n apache
```

Describe

```bash
kubectl describe svc apache-service -n apache
```

---

# Verify All Resources

```bash
kubectl get all -n apache
```

```bash
kubectl get namespaces
```

```bash
kubectl get sa -n apache
```

```bash
kubectl get roles -n apache
```

```bash
kubectl get rolebindings -n apache
```

```bash
kubectl get deployments -n apache
```

```bash
kubectl get pods -n apache
```

```bash
kubectl get svc -n apache
```

---

# RBAC Testing

Check all permissions for the current user.

```bash
kubectl auth can-i --list
```

---

## Test ServiceAccount Permissions

Get Pods

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

Create Pods

```bash
kubectl auth can-i create pods \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

Delete Pods

```bash
kubectl auth can-i delete pods \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

Get Services

```bash
kubectl auth can-i get services \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

Delete Deployments

```bash
kubectl auth can-i delete deployments \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

List All Allowed Permissions

```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:apache:apache-user \
  -n apache
```

---

# Cleanup

Delete Individual Resources

```bash
kubectl delete -f service.yml
kubectl delete -f deployment.yml
kubectl delete -f rolebinding.yml
kubectl delete -f role.yml
kubectl delete -f service-account.yml
kubectl delete -f namespace.yml
```

Delete Entire Namespace

```bash
kubectl delete namespace apache
```

---

# Quick Commands

```bash
kubectl get all -n apache

kubectl get pods -o wide -n apache

kubectl describe pod <pod-name> -n apache

kubectl logs <pod-name> -n apache

kubectl exec -it <pod-name> -n apache -- sh

kubectl auth can-i --list

kubectl auth can-i get pods \
--as=system:serviceaccount:apache:apache-user \
-n apache
```