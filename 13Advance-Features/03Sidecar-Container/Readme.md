# Sidecar Container in Kubernetes

## What is a Sidecar Container?

A **Sidecar Container** is an additional container that runs **alongside the main application container inside the same Pod**.

Unlike an Init Container, a Sidecar **does not exit** after completing a task. It keeps running as long as the Pod is running.

---

# Execution Flow

```
Pod Created
      │
      ▼
Main Container Starts
      │
      ▼
Sidecar Container Starts
      │
      ▼
Both Containers Run Together
```

---

# Why do we use Sidecar Containers?

A Sidecar performs supporting tasks for the main application without changing the application's code.

Common tasks include:

- Log collection
- Monitoring
- Proxy
- File synchronization
- Certificate renewal

---

# Real Example

Suppose you have a MERN backend.

```
Backend
```

The backend continuously writes logs.

```
Server Started

Database Connected

User Login

Attendance Updated

Student Created
```

Instead of modifying the backend to send these logs somewhere, we add a Sidecar.

```
Backend
     │
     ▼
app.log
     │
     ▼
Fluent Bit
     │
     ▼
Elasticsearch
```

The backend only writes logs.

The Sidecar reads those logs and sends them to another system.

---

# Pod Architecture

```
Pod
│
├── Backend Container
│
└── Fluent Bit Sidecar
```

Both containers share the same Pod.

---

# Shared Volume

Usually both containers share one volume.

```
Pod

Backend
    │
    ▼
 /logs/app.log

───────────────
 Shared Volume
───────────────

    ▲
    │

Fluent Bit
```

Backend writes the logs.

Fluent Bit reads the same logs.

---

# Sidecar vs Init Container

| Init Container | Sidecar Container |
|---------------|-------------------|
| Runs before application | Runs with application |
| Runs only once | Runs continuously |
| Exits after work | Keeps running |
| Used for initialization | Used for supporting tasks |

---

# Common Use Cases

## 1. Log Collection

```
Backend

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

---

## 2. Monitoring

```
Backend

↓

Prometheus Exporter
```

---

## 3. Service Mesh

```
Application

↓

Envoy Proxy

↓

Other Services
```

---

## 4. File Synchronization

```
Backend

↓

Shared Folder

↓

S3
```

---

## 5. Certificate Renewal

```
Backend

↓

TLS Certificate

↑

Sidecar renews certificate
```

---

# Fluent Bit

Fluent Bit is an open-source lightweight log collector.

Its job is:

- Read logs
- Process logs
- Send logs to another system

Examples:

- Elasticsearch
- OpenSearch
- Loki
- Splunk
- AWS CloudWatch
- Azure Monitor

Fluent Bit itself does **not** require any account.

Only the destination service may require one.

---

# Without Fluent Bit

```
Backend

↓

Logs

↓

Pod Deleted

❌ Logs Lost
```

---

# With Fluent Bit

```
Backend

↓

Logs

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana

✅ Logs Stored
```

---

# Where do we write Sidecar?

Inside the **containers** section of Deployment.

```
spec:
  template:
    spec:

      containers:
```

Sidecar is simply another container in the same list.

---

# Kubernetes Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 3

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:

      containers:

      - name: backend
        image: hammadch123/backend:latest

        ports:
          - containerPort: 5000

        volumeMounts:
          - name: logs-volume
            mountPath: /logs

      - name: fluent-bit
        image: fluent/fluent-bit:latest

        volumeMounts:
          - name: logs-volume
            mountPath: /logs

      volumes:
        - name: logs-volume
          emptyDir: {}
```

---

# Helm Example

## values.yaml

```yaml
sidecars:
  - name: fluent-bit
    image: fluent/fluent-bit:latest

    volumeMounts:
      - name: logs-volume
        mountPath: /logs

volumes:
  - name: logs-volume
    emptyDir: {}
```

---

## deployment.yaml

Main container

```yaml
containers:
  - name: {{ .Chart.Name }}
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Sidecar

```yaml
{{- with .Values.sidecars }}
{{ toYaml . | nindent 8 }}
{{- end }}
```

Volumes

```yaml
{{- with .Values.volumes }}
volumes:
  {{- toYaml . | nindent 8 }}
{{- end }}
```

---

# Complete Flow

```
User

↓

Frontend

↓

Backend

↓

/logs/app.log

↓

Shared Volume

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana Dashboard
```

---

# How to Check Sidecar

## List Pods

```bash
kubectl get pods -n dev
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name> -n dev
```

You should see:

```
Containers:
  backend
  fluent-bit
```

---

## View Backend Logs

```bash
kubectl logs <pod-name> -c backend -n dev
```

---

## View Sidecar Logs

```bash
kubectl logs <pod-name> -c fluent-bit -n dev
```

---

## View Pod YAML

```bash
kubectl get pod <pod-name> -o yaml -n dev
```

---

## List Container Names

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'
```

Example Output

```
backend fluent-bit
```

---

# Production Architecture

```
                    Users
                      │
                      ▼
                  Ingress
                      │
                      ▼
                  Backend Pod
         ┌─────────────────────────┐
         │                         │
         │  Backend Container      │
         │           │             │
         │           ▼             │
         │       /logs/app.log     │
         │           │             │
         │     Shared Volume       │
         │           │             │
         │           ▼             │
         │  Fluent Bit Sidecar     │
         └───────────┬─────────────┘
                     │
                     ▼
               Elasticsearch
                     │
                     ▼
                  Kibana
```

---

# Key Points

- Sidecar runs alongside the main application.
- Both containers share the same Pod.
- Both containers can share the same volume.
- Sidecar keeps running until the Pod stops.
- Commonly used for logging, monitoring, proxying, and file synchronization.
- Fluent Bit is one of the most common Sidecar containers used in Kubernetes.
- Sidecar is written inside the `containers:` section of `deployment.yaml`.
- In Helm, Sidecars are usually configured through `values.yaml` and rendered in `deployment.yaml`.