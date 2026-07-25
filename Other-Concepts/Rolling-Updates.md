# Kubernetes Rolling Update & Rollback

## Objective

Learn how Kubernetes updates applications without downtime using **Rolling Updates** and how to restore the previous version using **Rollback**.

---

# What is a Rolling Update?

A **Rolling Update** updates Pods **one by one** instead of stopping the entire application.

This ensures that users can continue using the application while the new version is being deployed.

---

# Before Update

Current Deployment

```
Deployment
Replicas = 5

Pod-1   nginx:1.14
Pod-2   nginx:1.14
Pod-3   nginx:1.14
Pod-4   nginx:1.14
Pod-5   nginx:1.14
```

All Pods are running version **1.14**.

---

# Update the Image

```bash
kubectl set image deployment/nginx-deployment \
nginx=nginx:1.15 -n nginx
```

---

# How Kubernetes Performs the Update

Kubernetes **does not delete all Pods at once**.

Instead, it replaces them one by one.

```
Old Pods                    New Pods

Pod-1 (1.14)  ❌  ─────►  Pod-1 (1.15) ✅

Pod-2 (1.14)  ❌  ─────►  Pod-2 (1.15) ✅

Pod-3 (1.14)  ❌  ─────►  Pod-3 (1.15) ✅

Pod-4 (1.14)  ❌  ─────►  Pod-4 (1.15) ✅

Pod-5 (1.14)  ❌  ─────►  Pod-5 (1.15) ✅
```

The next Pod is updated **only after** the previous new Pod becomes:

- Running
- Ready

---

# Scenario 1 - Invalid Image

Suppose you accidentally use a non-existing image.

```yaml
image: nginx:abc123
```

Kubernetes creates a new Pod.

```
Pod-1 (abc123)

ImagePullBackOff ❌
```

Existing Pods remain running.

```
Pod-1 (1.14) ✅

Pod-2 (1.14) ✅

Pod-3 (1.14) ✅

Pod-4 (1.14) ✅

Pod-5 (1.14) ✅
```

Result

- Old Pods continue serving users.
- Update pauses.
- Application stays available.

---

# Scenario 2 - Application Crash

The image downloads successfully, but the application crashes.

```
Pod-1 (1.15)

CrashLoopBackOff ❌
```

Kubernetes detects that the Pod is **not Ready**.

Result

- Old Pods continue running.
- Deployment stops progressing.
- Users still access the old version.

---

# Rollback

If the deployment fails, return to the previous version.

```bash
kubectl rollout undo deployment/nginx-deployment -n nginx
```

Kubernetes restores the last working Deployment.

---

# Useful Commands

Check Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment -n nginx
```

View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment -n nginx
```

Restart Deployment

```bash
kubectl rollout restart deployment/nginx-deployment -n nginx
```

Rollback

```bash
kubectl rollout undo deployment/nginx-deployment -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Describe Deployment

```bash
kubectl describe deployment nginx-deployment -n nginx
```

---

# Traditional Server vs Kubernetes

### Traditional Server

```
Stop Old Application ❌
        │
        ▼
Start New Version ❌ (Failed)
        │
        ▼
Website Down ❌
```

---

### Kubernetes

```
Old Pods Running ✅
        │
        ▼
Create New Pod
        │
        ▼
Ready?
   │
   ├── Yes ✅
   │      │
   │      ▼
   │ Delete One Old Pod
   │
   └── No ❌
          │
          ▼
Old Pods Continue Running
```

Result

- Zero or minimal downtime
- Safer deployments
- Automatic health checks
- Easy rollback

---

# Complete Rolling Update Flow

```
Deployment
      │
      ▼
Update Image
      │
      ▼
Create One New Pod
      │
      ▼
Is New Pod Ready?
      │
 ┌────┴────┐
 │         │
Yes       No
 │         │
 ▼         ▼
Delete     Keep Old Pods Running
One Old    Pause Update
Pod
 │
 ▼
Repeat Until All Pods Updated
```

---

# Learning Summary

✅ Learned Rolling Update

✅ Learned how Kubernetes updates Pods one by one

✅ Understood ImagePullBackOff

✅ Understood CrashLoopBackOff

✅ Learned Rollback

✅ Understood why Kubernetes provides high availability during deployments