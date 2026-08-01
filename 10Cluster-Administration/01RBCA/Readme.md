# RBAC (Role-Based Access Control) in Kubernetes

## What is RBAC?

**RBAC (Role-Based Access Control)** is a Kubernetes authorization mechanism used to control **who can perform which actions on which resources**.

It allows administrators to grant or restrict permissions for users, service accounts, or groups.

---

# Why RBAC?

Without RBAC, anyone with cluster access could:

- Delete Pods
- Delete Deployments
- Read Secrets
- Modify Services
- Access ConfigMaps

RBAC prevents unauthorized access by assigning only the required permissions.

---

# RBAC Testing Commands

After creating Roles, RoleBindings, ClusterRoles, or ClusterRoleBindings, you can verify permissions using the `kubectl auth can-i` command.

## Check if you can access Namespaces

```bash
kubectl auth can-i get namespaces
```

Example Output:

```text
Warning: resource 'namespaces' is not namespace scoped

yes
```

> **Note:** Namespaces are **cluster-scoped resources**, so Kubernetes shows this warning. It is normal and does not indicate an error.

---

## Check if you can read Pods

```bash
kubectl auth can-i get pods
```

Example Output:

```text
yes
```

---

## Check if you can read Pods in a specific Namespace

```bash
kubectl auth can-i get pods -n apache
```

Example Output:

```text
yes
```

---

## Check if you can read Deployments

```bash
kubectl auth can-i get deployments -n apache
```

Example Output:

```text
yes
```

---

## Check if you can delete Deployments

```bash
kubectl auth can-i delete deployments -n apache
```

Example Output:

```text
yes
```

---

## Check all your permissions

```bash
kubectl auth can-i --list
```

This command displays all the actions your current user or ServiceAccount is allowed to perform.

---

# Understanding the Output

| Output | Meaning |
|---------|----------|
| `yes` | You have permission to perform the requested action. |
| `no` | You do not have permission to perform the requested action. |

---

## Real Example

```bash
kubectl auth can-i get namespaces
# yes

kubectl auth can-i get pods
# yes

kubectl auth can-i get pods -n apache
# yes

kubectl auth can-i get deployments -n apache
# yes

kubectl auth can-i delete deployments -n apache
# yes
```

If the output is **yes**, RBAC allows the operation.

If the output is **no**, RBAC blocks the operation because the required permission is not granted.


---

# RBAC Components

```
User / ServiceAccount
         │
         ▼
   Role / ClusterRole
         │
         ▼
 RoleBinding / ClusterRoleBinding
         │
         ▼
   Kubernetes API Server
```

---

# 1. Role

A **Role** defines permissions **within a single namespace**.

Example:

- Read Pods
- Create Services
- Update Deployments

A Role **cannot** access resources outside its namespace.

---

## Role Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: pod-reader
  namespace: apache

rules:
- apiGroups: [""]
  resources:
  - pods

  verbs:
  - get
  - list
  - watch
```

### Explanation

- Namespace → apache
- Resource → Pods
- Permissions:
  - get
  - list
  - watch

This Role only allows reading Pods in the **apache** namespace.

---

# 2. RoleBinding

A **RoleBinding** assigns a **Role** to a User, Group, or ServiceAccount.

RoleBinding itself does not contain permissions.

It only connects:

```
User
   │
   ▼
RoleBinding
   │
   ▼
Role
```

---

## RoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: read-pods
  namespace: apache

subjects:
- kind: ServiceAccount
  name: frontend-sa
  namespace: apache

roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Now **frontend-sa** can read Pods inside the **apache** namespace.

---

# Role Flow

```
ServiceAccount
      │
      ▼
RoleBinding
      │
      ▼
Role
      │
      ▼
Can Read Pods
```

---

# 3. ClusterRole

A **ClusterRole** defines permissions **for the entire Kubernetes cluster**.

Unlike Role, it is **not limited to one namespace**.

It can:

- Read Nodes
- Read PersistentVolumes
- Read Namespaces
- Access all Pods in every namespace

---

## ClusterRole Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole

metadata:
  name: pod-reader

rules:
- apiGroups: [""]
  resources:
  - pods

  verbs:
  - get
  - list
  - watch
```

This ClusterRole can read Pods from **all namespaces**.

---

# 4. ClusterRoleBinding

A **ClusterRoleBinding** assigns a ClusterRole to a User, Group, or ServiceAccount.

```
ServiceAccount
       │
       ▼
ClusterRoleBinding
       │
       ▼
ClusterRole
       │
       ▼
Entire Cluster Access
```

---

## ClusterRoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: read-all-pods

subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: monitoring

roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Now **monitoring-sa** can read Pods from **every namespace**.

---

# Role vs ClusterRole

| Feature | Role | ClusterRole |
|---------|------|-------------|
| Scope | One Namespace | Entire Cluster |
| Access Pods | ✅ | ✅ |
| Access Nodes | ❌ | ✅ |
| Access PV | ❌ | ✅ |
| Access Multiple Namespaces | ❌ | ✅ |

---

# RoleBinding vs ClusterRoleBinding

| Feature | RoleBinding | ClusterRoleBinding |
|---------|-------------|--------------------|
| Binds | Role | ClusterRole |
| Scope | One Namespace | Entire Cluster |
| Used For | Namespace Access | Cluster-wide Access |

---

# Real-World Example

Suppose your cluster has three namespaces:

```
frontend

backend

database
```

### Frontend Developer

Needs access only to the frontend namespace.

Use:

```
Role
+
RoleBinding
```

---

### Backend Developer

Needs access only to backend.

Use:

```
Role
+
RoleBinding
```

---

### DevOps Engineer

Needs access to the whole cluster.

Use:

```
ClusterRole
+
ClusterRoleBinding
```

---

# Complete RBAC Flow

```
Developer
      │
      ▼
ServiceAccount
      │
      ▼
RoleBinding
      │
      ▼
Role
      │
      ▼
Permissions
      │
      ▼
Kubernetes API Server
```

Cluster-wide:

```
DevOps Engineer
        │
        ▼
ServiceAccount
        │
        ▼
ClusterRoleBinding
        │
        ▼
ClusterRole
        │
        ▼
Entire Kubernetes Cluster
```

---

# Useful Commands

Create RBAC Resources

```bash
kubectl apply -f role.yml
kubectl apply -f rolebinding.yml
kubectl apply -f clusterrole.yml
kubectl apply -f clusterrolebinding.yml
```

List Roles

```bash
kubectl get roles --all-namespaces
```

List RoleBindings

```bash
kubectl get rolebindings --all-namespaces
```

List ClusterRoles

```bash
kubectl get clusterroles
```

List ClusterRoleBindings

```bash
kubectl get clusterrolebindings
```

Describe a Role

```bash
kubectl describe role pod-reader -n apache
```

Describe a ClusterRole

```bash
kubectl describe clusterrole pod-reader
```

---

# Summary

- **RBAC** controls access to Kubernetes resources.
- **Role** defines permissions within a single namespace.
- **RoleBinding** assigns a Role to a User or ServiceAccount.
- **ClusterRole** defines permissions for the entire cluster.
- **ClusterRoleBinding** assigns a ClusterRole across the whole cluster.
- Use **Role + RoleBinding** for namespace-specific access.
- Use **ClusterRole + ClusterRoleBinding** for cluster-wide access.