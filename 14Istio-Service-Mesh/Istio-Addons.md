# Istio Addons Installation (Kiali, Grafana, Prometheus & Jaeger)

This guide explains how to install Istio observability addons and verify that everything is working correctly.

---

# Architecture

```
                     Browser
                        │
                        ▼
              Kiali / Grafana / Jaeger
                        │
                        ▼
                  Istio Control Plane
                        │
                        ▼
                  Prometheus Metrics
                        │
                        ▼
                Bookinfo Application
```

---

# Prerequisites

- Running Kubernetes Cluster
- Istio Installed
- Bookinfo Application Running
- `istioctl` Installed

Verify

```bash
kubectl get nodes

kubectl get pods -n istio-system

istioctl version
```

---

# Step 1: Install Istio Addons

Install all observability components.

```bash
kubectl apply -f samples/addons
```

## Purpose

This installs:

- Kiali
- Grafana
- Prometheus
- Jaeger

Expected Output

```text
serviceaccount/grafana created
deployment.apps/grafana created

serviceaccount/kiali created
deployment.apps/kiali created

serviceaccount/prometheus created
deployment.apps/prometheus created

serviceaccount/jaeger created
deployment.apps/jaeger created
```

---

# Step 2: Verify Resources

Check whether Kubernetes created all resources.

```bash
kubectl get all -n istio-system
```

Expected

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 3: Verify Pods

```bash
kubectl get pods -n istio-system
```

Initially you may see

```text
Pending

ContainerCreating
```

After a few seconds they should become

```text
Running
```

Expected

```text
grafana

kiali

prometheus

jaeger

istiod

istio-ingressgateway
```

---

# Step 4: Wait for Deployments

Wait until every deployment is ready.

## Grafana

```bash
kubectl rollout status deployment/grafana -n istio-system
```

Expected

```text
deployment "grafana" successfully rolled out
```

---

## Kiali

```bash
kubectl rollout status deployment/kiali -n istio-system
```

Expected

```text
deployment "kiali" successfully rolled out
```

---

## Prometheus

```bash
kubectl rollout status deployment/prometheus -n istio-system
```

Expected

```text
deployment "prometheus" successfully rolled out
```

---

## Jaeger

```bash
kubectl rollout status deployment/jaeger -n istio-system
```

Expected

```text
deployment "jaeger" successfully rolled out
```

---

# Step 5: Verify Pods Again

```bash
kubectl get pods -n istio-system
```

Expected

```text
NAME                                  READY   STATUS

grafana-xxxxx                         1/1     Running

jaeger-xxxxx                          1/1     Running

kiali-xxxxx                           1/1     Running

prometheus-xxxxx                      2/2     Running

istiod-xxxxx                          1/1     Running

istio-ingressgateway-xxxxx            1/1     Running
```

---

# Step 6: Verify Services

```bash
kubectl get svc -n istio-system
```

Expected

```text
grafana

kiali

prometheus

tracing

istiod

istio-ingressgateway
```

---

# Step 7: Open Kiali Dashboard

Port Forward

```bash
kubectl port-forward svc/kiali -n istio-system 20001:20001 --address=0.0.0.0
```

Open Browser

```
http://EC2_PUBLIC_IP:20001
```

---

# Step 8: Open Grafana Dashboard

Port Forward

```bash
kubectl port-forward svc/grafana -n istio-system 3000:3000 --address=0.0.0.0
```

Open Browser

```
http://EC2_PUBLIC_IP:3000
```

---

# Step 9: Open Prometheus

Port Forward

```bash
kubectl port-forward svc/prometheus -n istio-system 9090:9090 --address=0.0.0.0
```

Open Browser

```
http://EC2_PUBLIC_IP:9090
```

---

# Step 10: Open Jaeger Dashboard

Port Forward

```bash
kubectl port-forward svc/tracing -n istio-system 16686:80 --address=0.0.0.0
```

Open Browser

```
http://EC2_PUBLIC_IP:16686
```

---

# Step 11: Generate Traffic

Before opening Kiali, generate some traffic.

Open the Bookinfo application several times.

```
http://EC2_PUBLIC_IP:8080/productpage
```

Refresh the page 15–20 times.

---

# Step 12: Open Kiali Graph

Open

```
http://EC2_PUBLIC_IP:20001
```

Navigate to

```
Graph

↓

Namespace

↓

default

↓

Display
```

You should see

```
Ingress Gateway
        │
        ▼
Productpage
   ├────────► Details
   ├────────► Reviews
   │              │
   │              ▼
   └──────────► Ratings
```

This graph represents the communication between your microservices.

---

# Step 13: Verify Istio Proxy

```bash
istioctl proxy-status
```

Expected

```text
bookinfo-gateway-istio

productpage

details

ratings

reviews-v1

reviews-v2

reviews-v3
```

All proxies should be connected to **istiod**.

---

# Step 14: Verify Installation

```bash
istioctl verify-install
```

Expected

```text
✔ No issues found
```

---

# Step 15: Analyze Configuration

```bash
istioctl analyze
```

Expected

```text
✔ No validation issues found
```

---

# Complete Workflow

```
Install Addons

        │

        ▼

kubectl get pods

        │

        ▼

Pods Created

        │

        ▼

kubectl rollout status

        │

        ▼

Deployments Ready

        │

        ▼

kubectl get svc

        │

        ▼

Port Forward Dashboards

        │

        ▼

Open Browser

        │

        ▼

Generate Bookinfo Traffic

        │

        ▼

Open Kiali

        │

        ▼

Visualize Service Mesh

        │

        ▼

Open Grafana

        │

        ▼

View Metrics

        │

        ▼

Open Prometheus

        │

        ▼

Query Metrics

        │

        ▼

Open Jaeger

        │

        ▼

View Distributed Traces
```

---

# Useful Commands

```bash
kubectl get pods -n istio-system

kubectl get svc -n istio-system

kubectl get deploy -n istio-system

kubectl rollout status deployment/kiali -n istio-system

kubectl rollout status deployment/grafana -n istio-system

kubectl rollout status deployment/prometheus -n istio-system

kubectl rollout status deployment/jaeger -n istio-system

istioctl proxy-status

istioctl analyze

istioctl verify-install

kubectl logs deploy/kiali -n istio-system

kubectl logs deploy/prometheus -n istio-system

kubectl logs deploy/grafana -n istio-system

kubectl logs deploy/jaeger -n istio-system
```

---

# Summary

- Install Istio observability addons.
- Verify Pods, Services, and Deployments.
- Wait for all deployments to become ready.
- Expose dashboards using `kubectl port-forward`.
- Generate traffic by accessing the Bookinfo application.
- Open Kiali to visualize the service mesh.
- Use Grafana to monitor metrics.
- Use Prometheus to query metrics.
- Use Jaeger to inspect distributed traces.
- Verify the installation using `istioctl verify-install`.
- Analyze the mesh configuration using `istioctl analyze`.