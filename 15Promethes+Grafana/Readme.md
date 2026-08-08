# Prometheus + Grafana Monitoring on Kind Kubernetes

This guide explains how to install **Prometheus + Grafana** on an existing **Kind Kubernetes cluster** using **Helm**, without changing or redeploying the existing application.

We will use the **kube-prometheus-stack** Helm chart.

---

# Architecture

```text
Your Existing Project
        │
        │ Metrics
        ▼
   Prometheus
        │
        │ PromQL
        ▼
     Grafana
        │
        ▼
   📊 Dashboards
```

The `kube-prometheus-stack` provides:

```text
kube-prometheus-stack
│
├── Prometheus
├── Grafana
├── Alertmanager
├── Node Exporter
└── kube-state-metrics
```

---

# Prerequisites

Make sure you already have:

* Kind
* kubectl
* Helm
* An existing Kubernetes project running on Kind

---

# Step 1 — Check Kind Cluster

Check your Kubernetes nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME                    STATUS   ROLES
kind-control-plane      Ready    control-plane
kind-worker             Ready    <none>
kind-worker2            Ready    <none>
```

Check your existing project:

```bash
kubectl get pods -A
```

Make sure your application pods are running.

---

# Step 2 — Check Helm

Verify Helm:

```bash
helm version
```

If Helm is already installed, continue to the next step.

---

# Step 3 — Add Prometheus Helm Repository

Add the Prometheus Community repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update Helm repositories:

```bash
helm repo update
```

Verify that the chart is available:

```bash
helm search repo prometheus-community/kube-prometheus-stack
```

You should see something similar to:

```text
prometheus-community/kube-prometheus-stack
```

---

# Step 4 — Create Monitoring Namespace

Keep monitoring components separate from the application.

Create the namespace:

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get ns
```

You should see:

```text
monitoring
```

---

# Step 5 — Install Prometheus + Grafana

Install the `kube-prometheus-stack`:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

This installation provides:

```text
Prometheus
Grafana
Alertmanager
Node Exporter
kube-state-metrics
Prometheus Operator
```

---

# Step 6 — Check Monitoring Pods

Check the monitoring pods:

```bash
kubectl get pods -n monitoring
```

You should eventually see pods similar to:

```text
alertmanager-monitoring-kube-prometheus-stack-alertmanager
monitoring-grafana
monitoring-kube-prometheus-stack-operator
monitoring-kube-state-metrics
monitoring-prometheus-node-exporter
prometheus-monitoring-kube-prometheus-stack-prometheus
```

Wait until the important pods show:

```text
STATUS: Running
```

---

# Step 7 — Check Helm Installation

Check Helm releases:

```bash
helm list -n monitoring
```

Expected:

```text
NAME        NAMESPACE    STATUS
monitoring  monitoring   deployed
```

You can also check all monitoring resources:

```bash
kubectl get all -n monitoring
```

---

# Step 8 — Access Grafana

Because we are using **Kind**, the easiest way to access Grafana during development is through port-forwarding.

First check the services:

```bash
kubectl get svc -n monitoring
```

You should see something similar to:

```text
monitoring-grafana    ClusterIP    10.x.x.x    80
```

Start port-forwarding:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Now open:

```text
http://localhost:3000
```

---

# Step 9 — Get Grafana Password

The Helm chart creates a Kubernetes Secret containing the Grafana admin password.

Run:

```bash
kubectl get secret -n monitoring monitoring-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

This will return the password:

```text
xxxxxxxxxxxxxxxx
```

Grafana login:

```text
Username: admin
Password: <password returned by command>
```

---

# Step 10 — Access Prometheus

Find the Prometheus service:

```bash
kubectl get svc -n monitoring | grep prometheus
```

You should see something similar to:

```text
monitoring-kube-prometheus-stack-prometheus
```

Port-forward Prometheus:

```bash
kubectl port-forward -n monitoring \
  svc/monitoring-kube-prometheus-stack-prometheus 9090:9090
```

Open:

```text
http://localhost:9090
```

You should now see the Prometheus UI.

---

# Step 11 — Verify Prometheus Targets

Inside Prometheus, go to:

```text
Status
   ↓
Targets
```

You should see targets such as:

```text
kube-state-metrics
node-exporter
kubernetes-apiservers
kubernetes-nodes
```

The important thing is:

```text
State = UP
```

If the target is `UP`, Prometheus is successfully scraping metrics from that target.

---

# Step 12 — Test Kubernetes Metrics

In the Prometheus query interface, try:

### Check available targets

```promql
up
```

This shows whether Prometheus targets are available.

### Container CPU

```promql
container_cpu_usage_seconds_total
```

### Container Memory

```promql
container_memory_working_set_bytes
```

### Kubernetes Pod Information

```promql
kube_pod_info
```

If these queries return data, Prometheus is successfully collecting Kubernetes metrics.

---

# Step 13 — Open Grafana

Open:

```text
http://localhost:3000
```

Login using:

```text
Username: admin
Password: <Grafana password>
```

The `kube-prometheus-stack` normally configures Prometheus as a Grafana datasource automatically.

Therefore, you usually **do not need to manually add Prometheus as a datasource**.

---

# Step 14 — Explore Grafana Dashboards

Go to:

```text
Dashboards
```

The stack provides Kubernetes monitoring dashboards.

You can monitor things such as:

```text
Kubernetes Cluster
─────────────────────────────
CPU Usage
Memory Usage
Pod Count
Node Count
Network Traffic
Disk Usage
```

You can drill down through:

```text
Cluster
   ↓
Node
   ↓
Namespace
   ↓
Pod
```

---

# Existing Project + Monitoring

Suppose your existing Kind project looks like this:

```text
Kind Cluster
│
├── frontend
│     └── Pods
│
├── backend
│     └── Pods
│
├── mongodb
│     └── Pod
│
└── ingress
```

After installing monitoring:

```text
Kind Cluster
│
├── Your Application
│   ├── frontend
│   ├── backend
│   ├── mongodb
│   └── ingress
│
└── monitoring
    │
    ├── Prometheus
    ├── Grafana
    ├── Alertmanager
    ├── Node Exporter
    └── kube-state-metrics
```

Installing the monitoring stack **does not require you to redeploy your existing application**.

---

# Two Types of Monitoring

It is important to understand that Kubernetes monitoring has two major areas.

## 1. Kubernetes / Infrastructure Metrics

Examples:

```text
Node CPU
Node RAM
Pod CPU
Pod RAM
Pod Restarts
Deployment Replicas
Node Status
Network Traffic
```

The `kube-prometheus-stack` handles a large portion of these metrics automatically.

---

## 2. Application Metrics

Your own application can expose application-specific metrics.

For example, a Node.js/Express backend can expose:

```text
GET /metrics
```

It can provide metrics such as:

```text
HTTP Requests
Request Duration
5xx Errors
Active Requests
Request Rate
```

Prometheus can then scrape these application metrics.

---

# Future Architecture

Once application metrics are added, the architecture can become:

```text
                    Kind Cluster
                         │
              ┌──────────┴──────────┐
              │                     │
        Application             Monitoring
              │                     │
        ┌─────┴─────┐        ┌──────┴──────┐
        │           │        │             │
    Frontend     Backend  Prometheus     Grafana
                     │         │
                     │         │
                  /metrics ────┘
```

This allows Grafana to display both:

```text
Infrastructure Metrics
        +
Application Metrics
```

---

# Important

For the first setup, **do not worry about `/metrics`**.

First make sure:

```text
Prometheus
    ↓
Kubernetes Metrics
    ↓
Grafana
    ↓
Dashboards
```

is working correctly.

After that, add application-level metrics from your Node/Express backend.

---

# Complete Installation Commands

If Kind, kubectl, and Helm are already installed, the main installation is:

```bash
# Add Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update repositories
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring

# Check pods
kubectl get pods -n monitoring

# Check services
kubectl get svc -n monitoring
```

---

# Access Grafana

Run:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open:

```text
http://localhost:3000
```

---

# Access Prometheus

Run:

```bash
kubectl port-forward -n monitoring \
  svc/monitoring-kube-prometheus-stack-prometheus 9090:9090
```

Open:

```text
http://localhost:9090
```

---

# Final Architecture

After successful installation:

```text
                         KIND CLUSTER
                              │
              ┌───────────────┴───────────────┐
              │                               │
        APPLICATION                      MONITORING
              │                               │
      ┌───────┼───────┐             ┌────────┼────────┐
      │       │       │             │        │        │
   Frontend Backend MongoDB     Prometheus Grafana Alertmanager
                                  │
                         ┌────────┴─────────┐
                         │                  │
                    Node Exporter     kube-state-metrics
```

The basic monitoring flow is:

```text
Kubernetes
    │
    │ Metrics
    ▼
Prometheus
    │
    │ PromQL
    ▼
Grafana
    │
    ▼
Dashboards
```

---

# Next Step

Once this setup is working, the next step is to add **application-level monitoring**:

```text
Node/Express Backend
        │
        │ /metrics
        ▼
    Prometheus
        │
        │ PromQL
        ▼
     Grafana
        │
        ▼
Custom Application Dashboard
```

This is where you can monitor your own application's:

```text
Request Rate
Response Time
Error Rate
5xx Errors
Active Requests
CPU
Memory
```
