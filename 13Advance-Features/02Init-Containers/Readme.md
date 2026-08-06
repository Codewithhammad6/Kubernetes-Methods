# Init Containers in Kubernetes

## What is an Init Container?

An **Init Container** is a special container that runs **before the main application container starts**.

It performs initialization tasks such as:

- Waiting for another service (MongoDB, MySQL, Redis)
- Creating directories
- Downloading configuration files
- Running database migrations
- Setting file permissions
- Checking dependencies

After the Init Container completes successfully, Kubernetes starts the main application container.

---

# Execution Flow

```
Pod Created
      │
      ▼
Init Container Starts
      │
      ▼
Initialization Task
      │
      ▼
Success?
      │
 ┌────┴────┐
 │         │
No         Yes
 │          │
Retry      Main Container Starts
```

---

# Why do we use Init Containers?

Imagine this architecture:

```
Frontend
    │
    ▼
Backend
    │
    ▼
MongoDB
```

When Kubernetes starts, all Pods are created at almost the same time.

Sometimes:

- Backend starts first
- MongoDB is still starting
- Backend tries to connect
- Connection fails

Instead of letting the backend fail, we make it wait.

This is the job of an Init Container.

---

# Real Life Example

Without Init Container

```
Backend
    │
    ▼
Connect MongoDB
    │
    ▼
MongoDB Not Ready
    │
    ▼
Application Crash
```

---

With Init Container

```
Backend Pod
      │
      ▼
Init Container
      │
      ▼
Wait for MongoDB
      │
      ▼
MongoDB Ready
      │
      ▼
Backend Starts
```

---

# Where do we write Init Containers?

Inside **Deployment.yaml**

Example:

```yaml
spec:
  template:
    spec:

      initContainers:

      containers:
```

Init Containers are written **before** the `containers:` section.

---

# Example 1 (Port Check)

This is the easiest and most common example.

```yaml
initContainers:
  - name: wait-for-mongodb
    image: busybox:1.36
    command:
      - sh
      - -c
      - |
        until nc -z mongodb 27017; do
          echo "Waiting for MongoDB..."
          sleep 5
        done
```

What does this do?

- Check if MongoDB port 27017 is open
- If not, wait 5 seconds
- Keep checking
- Start Backend only after MongoDB becomes available

---

# Example 2 (Authentication Check)

Production environments often verify that MongoDB is actually accepting authenticated connections.

```yaml
initContainers:
  - name: wait-for-mongodb
    image: mongo:7.0
    command:
      - sh
      - -c
      - |
        until mongosh "mongodb://admin:admin123@mongodb:27017/admin?authSource=admin" \
          --eval "db.adminCommand('ping')"; do
          echo "Waiting for MongoDB..."
          sleep 5
        done
```

This checks:

- MongoDB is running
- Username is correct
- Password is correct
- Authentication works
- Database is accepting requests

---

# Which One Should You Use?

## Learning / Small Projects

```
nc -z mongodb 27017
```

Advantages

- Simple
- Fast
- Easy to understand

---

## Production

```
mongosh
db.adminCommand("ping")
```

Advantages

- Verifies authentication
- Ensures database is actually ready
- More reliable

---

# Do we need Username and Password?

## Port Check

No.

```yaml
until nc -z mongodb 27017
```

Only checks if the port is open.

---

## MongoDB Authentication Check

Yes.

```yaml
mongodb://admin:admin123@mongodb:27017/admin?authSource=admin
```

---

# Common Use Cases

- Wait for MongoDB
- Wait for MySQL
- Wait for PostgreSQL
- Wait for Redis
- Download configuration files
- Run database migrations
- Create directories
- Generate certificates
- Set file permissions

---

# Init Container vs Main Container

| Init Container | Main Container |
|---------------|----------------|
| Runs first | Runs after Init Container |
| Runs only once | Runs continuously |
| Performs setup | Runs the application |
| Must finish successfully | Starts only after Init Container completes |

---

# Kubernetes Commands

## List Pods

```bash
kubectl get pods -n dev
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name> -n dev
```

Look for:

```
Init Containers:
```

---

## View Init Container Logs

```bash
kubectl logs <pod-name> -c wait-for-mongodb -n dev
```

Example:

```
Waiting for MongoDB...
Waiting for MongoDB...
MongoDB Ready.
```

---

## Check Pod YAML

```bash
kubectl get pod <pod-name> -o yaml -n dev
```

---

## Check if Init Container Exists

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.initContainers[*].name}'
```

Example Output

```
wait-for-mongodb
```

---

## Check Events

```bash
kubectl describe pod <pod-name> -n dev
```

Example

```
Pulling image
Created container wait-for-mongodb
Started container wait-for-mongodb
Init container completed
```

---

# If Init Container Fails

Example

```
kubectl get pods
```

Output

```
backend
STATUS: Init:0/1
```

or

```
STATUS: Init:CrashLoopBackOff
```

Meaning:

- Init Container failed
- Main application has not started yet

---

# Real Project Example (MERN)

```
React Frontend
       │
       ▼
Express Backend
       │
       ▼
Init Container
       │
       ▼
Wait for MongoDB
       │
       ▼
MongoDB Ready
       │
       ▼
Backend Starts
```

---

# Key Points

- Init Container runs before the application container.
- It runs only once.
- All Init Containers must complete successfully.
- The main container does not start until every Init Container finishes.
- Commonly used to wait for databases and perform initialization tasks.
- Production environments often use authenticated health checks instead of simple port checks.