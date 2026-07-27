# Kubernetes CronJob

## Objective

Learn how a **CronJob** works in Kubernetes and how it is used for scheduled tasks.

---

# What is a CronJob?

A **CronJob** is a Kubernetes resource that runs a **Job automatically on a schedule**.

Unlike a Job, which runs only once, a CronJob creates a new Job whenever the scheduled time arrives.

---

# CronJob Rule

> **"Run a Job automatically at the scheduled time."**

---

# CronJob Architecture

```
CronJob
    │
    ▼
Creates Job
    │
    ▼
Creates Pod
    │
    ▼
Runs Container
    │
    ▼
Task Completed
```

---

# Example 1 - Basic CronJob

This example simply prints the current date and a message every minute.

Create **cronjob-basic.yml**

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: hello

spec:
  schedule: "* * * * *"

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28

            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster

          restartPolicy: OnFailure
```

Apply

```bash
kubectl apply -f cronjob-basic.yml
```

---

# Flow

```
Every Minute
      │
      ▼
CronJob
      │
      ▼
Creates Job
      │
      ▼
Creates Pod
      │
      ▼
Runs

date
echo Hello

      │
      ▼
Completed
```

---

# Example 2 - Backup CronJob (Production Example)

This CronJob copies data from one folder to another every 2 minutes.

Create **cronjob-backup.yml**

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: backup-jobcron
  namespace: nginx

spec:
  schedule: "*/2 * * * *"

  jobTemplate:
    spec:
      template:
        metadata:
          name: min-backup
          labels:
            app: minute-backup

        spec:
          containers:
          - name: backup-container
            image: busybox:latest

            command:
            - sh
            - -c
            - >
              echo "Backup Started";
              mkdir -p /backups &&
              mkdir -p /demo-data &&
              cp -r /demo-data /backups &&
              echo "Backup Completed";

            volumeMounts:
            - name: data-volume
              mountPath: /demo-data

            - name: backup-volume
              mountPath: /backups

          restartPolicy: OnFailure

          volumes:
          - name: data-volume
            hostPath:
              path: /demo-data
              type: DirectoryOrCreate

          - name: backup-volume
            hostPath:
              path: /backups
              type: DirectoryOrCreate
```

Apply

```bash
kubectl apply -f cronjob-backup.yml
```

---

# Complete Backup Flow

```
Worker Node

/demo-data
/backups
      │
      ▼

CronJob
      │
      ▼
Creates Job
      │
      ▼
Creates Pod
      │
      ▼
Creates Container
      │
      ▼
Kubernetes reads volumes
      │
      ▼
Kubernetes mounts folders
      │
      ▼
Runs Backup Command
      │
      ▼
Backup Saved
      │
      ▼
Container Exits
      │
      ▼
Pod Deleted
```

---

# Understanding volumes

`volumes` defines **where the storage comes from**.

Example

```yaml
volumes:
- name: data-volume
  hostPath:
    path: /demo-data

- name: backup-volume
  hostPath:
    path: /backups
```

Meaning

```
data-volume
      │
      ▼
Worker:/demo-data

backup-volume
      │
      ▼
Worker:/backups
```

Kubernetes now knows where the data exists.

---

# Understanding volumeMounts

`volumeMounts` defines **where the storage should appear inside the container**.

```yaml
volumeMounts:
- name: data-volume
  mountPath: /demo-data

- name: backup-volume
  mountPath: /backups
```

Meaning

```
Worker Node

/demo-data
/backups
      │
      ▼

Container

/demo-data
/backups
```

The container now accesses the Worker Node folders.

---

# Backup Process

Container runs

```bash
cp -r /demo-data /backups
```

Container sees

```
/demo-data
      │
      ▼
/backups
```

Actually the data is copied between folders on the Worker Node.

---

# Difference Between volumes and volumeMounts

| volumes | volumeMounts |
|----------|--------------|
| Defines the storage source | Defines where the storage appears inside the container |
| Uses hostPath, PVC, Secret, ConfigMap, etc. | Uses mountPath |
| Worker Node side | Container side |

---

# Simple CronJob vs Backup CronJob

| Simple CronJob | Backup CronJob |
|----------------|----------------|
| Prints a message | Copies real data |
| No Storage | Uses Storage |
| No Volumes | Uses `volumes` |
| No Mounting | Uses `volumeMounts` |
| Beginner Example | Production Example |

---

# Common Schedule Examples

| Schedule | Meaning |
|----------|---------|
| `* * * * *` | Every minute |
| `*/2 * * * *` | Every 2 minutes |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour |
| `0 0 * * *` | Every day at midnight |
| `0 2 * * *` | Every day at 2:00 AM |

---

# Useful Commands

Apply

```bash
kubectl apply -f cronjob-backup.yml
```

Check CronJobs

```bash
kubectl get cronjobs -n nginx
```

Check Jobs

```bash
kubectl get jobs -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Delete CronJob

```bash
kubectl delete cronjob backup-jobcron -n nginx
```

---

# Interview Question

### Why do we need both `volumes` and `volumeMounts`?

**Answer:**

> `volumes` defines the storage source (hostPath, PVC, ConfigMap, Secret, etc.), while `volumeMounts` defines where that storage appears inside the container. Kubernetes first identifies the storage using `volumes` and then mounts it inside the container using `volumeMounts`.

---

# Learning Summary

✅ Learned CronJob

✅ Learned Job creation

✅ Learned scheduled execution

✅ Learned `volumes`

✅ Learned `volumeMounts`

✅ Learned production backup example

---

# One-Line Revision

- **Job** → Runs once.
- **CronJob** → Runs a Job automatically on a schedule.
- **volumes** → Defines the storage source.
- **volumeMounts** → Mounts that storage inside the container.





# Additional

By using this command we get by default 3 pods

Check Pods

```bash
kubectl get pods -n nginx
```

# Increase Job History Limits

In addition to the schedule, a CronJob can control how many completed and failed Jobs Kubernetes keeps.

```yaml
spec:
  schedule: "*/2 * * * *"

  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 3
```