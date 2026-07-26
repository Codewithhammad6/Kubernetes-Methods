# Kubernetes Job

## Objective

Learn how Kubernetes **Job** works and when it should be used.

---

# What is a Job?

A **Job** is a Kubernetes resource that runs a task **only once**.

Unlike a Deployment, a Job **does not keep running forever**.

Once the task completes successfully, the Job finishes.

---

# Job Rule

> **"Run the task until it completes successfully."**

If the Pod fails, Kubernetes creates a new Pod and retries until the Job succeeds (according to its policy).

---

# Job Architecture

```
Job
 │
 ▼
Pod
 │
 ▼
Container
 │
 ▼
Task Executes
 │
 ▼
Completed ✅
```

---

# Job YAML

Create **job.yml**

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: backup-job
  namespace: nginx

spec:
  completions: 1
  parallelism: 1

  template:
    metadata:
      labels:
        app: backup-task

    spec:
      containers:
      - name: backup
        image: busybox:latest
        command: ["sh", "-c", "echo Hello && sleep 10"]

      restartPolicy: Never
```

---

# Apply Job

```bash
kubectl apply -f job.yml
```

---

# Verify Job

Check Jobs

```bash
kubectl get jobs -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

Describe Job

```bash
kubectl describe job backup-job -n nginx
```

View Logs

```bash
kubectl logs <POD_NAME> -n nginx
```

Delete Job

```bash
kubectl delete job backup-job -n nginx
```

---

# How This Job Works

```
Job Created
      │
      ▼
Creates One Pod
      │
      ▼
Runs

echo Hello
sleep 10

      │
      ▼
Task Finished
      │
      ▼
Pod Status = Completed ✅
```

---

# Understanding Important Fields

## completions

Defines **how many successful task completions** are required.

```yaml
completions: 1
```

Meaning:

```
Run the task only once.
```

Example

```
Job
 │
 └── Pod
      │
      └── Completed ✅
```

---

## parallelism

Defines **how many Pods can run at the same time**.

```yaml
parallelism: 1
```

Meaning

Only **one Pod** runs at a time.

---

# Different Scenarios

## Scenario 1

```yaml
completions: 1
parallelism: 1
```

```
Only one Pod runs.

Job
 │
 └── Pod-1 ✅
```

---

## Scenario 2

```yaml
completions: 5
parallelism: 1
```

```
Five tasks run one after another.

Pod-1 ✅
Pod-2 ✅
Pod-3 ✅
Pod-4 ✅
Pod-5 ✅
```

---

## Scenario 3

```yaml
completions: 5
parallelism: 2
```

```
Two Pods run together.

Round 1

Pod-1 ✅
Pod-2 ✅

Round 2

Pod-3 ✅
Pod-4 ✅

Round 3

Pod-5 ✅
```

---

# Real Use Cases

Jobs are commonly used for one-time tasks.

Examples:

- Database Backup
- Database Restore
- Data Migration
- Import CSV Files
- Export Reports
- Generate PDF Files
- Send Emails
- Run Scripts

---

# Deployment vs Job

| Deployment | Job |
|------------|-----|
| Runs continuously | Runs once |
| Keeps Pods alive | Stops after completion |
| Used for Applications | Used for Tasks |
| Frontend, Backend APIs | Backup, Migration, Import |

---

# Resource Hierarchy

```
Job
 │
 └── Pod
      │
      └── Container
            │
            └── Task
```

---

# Useful Commands

Apply

```bash
kubectl apply -f job.yml
```

Check Jobs

```bash
kubectl get jobs -n nginx
```

Check Pods

```bash
kubectl get pods -n nginx
```

View Logs

```bash
kubectl logs <pod-name> -n nginx
```

Delete Job

```bash
kubectl delete job backup-job -n nginx
```

---

# Learning Summary

✅ Learned Job

✅ Understood `completions`

✅ Understood `parallelism`

✅ Learned one-time task execution

✅ Learned real production use cases

---

# One-Line Revision

- **Deployment** → Keeps applications running forever.
- **Job** → Runs a task until it completes successfully.





# Real Backup Example

Suppose you have **5 databases**.

```
DB-1
DB-2
DB-3
DB-4
DB-5
```

You want Kubernetes to create a backup for each database.

---

## Scenario 1

```yaml
completions: 5
parallelism: 1
```

Only **one backup** runs at a time.

```
Time

DB-1 Backup ✅

↓

DB-2 Backup ✅

↓

DB-3 Backup ✅

↓

DB-4 Backup ✅

↓

DB-5 Backup ✅
```

Result

- Total Backups = **5**
- Running at the same time = **1**

---

## Scenario 2

```yaml
completions: 5
parallelism: 2
```

Two backups run simultaneously.

```
Round 1

DB-1 Backup ✅
DB-2 Backup ✅

↓

Round 2

DB-3 Backup ✅
DB-4 Backup ✅

↓

Round 3

DB-5 Backup ✅
```

Result

- Total Backups = **5**
- Running at the same time = **2**

---

## Scenario 3

```yaml
completions: 5
parallelism: 5
```

All backups start together.

```
DB-1 Backup ✅
DB-2 Backup ✅
DB-3 Backup ✅
DB-4 Backup ✅
DB-5 Backup ✅
```

Result

- Total Backups = **5**
- Running at the same time = **5**

---

# Summary

| completions | parallelism | Result |
|-------------|------------:|--------|
| 1 | 1 | One backup runs once |
| 5 | 1 | Five backups run one by one |
| 5 | 2 | Five backups run two at a time |
| 5 | 5 | Five backups run all together |