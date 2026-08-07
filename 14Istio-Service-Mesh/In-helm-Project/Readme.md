# Kubernetes NGINX Ingress to Istio Gateway API Migration Guide

A complete step-by-step guide to migrate an existing **Kubernetes application** from **NGINX Ingress** to the **Istio Gateway API**.

---

# Overview

Suppose your existing Kubernetes project is using **NGINX Ingress**.

### Before Migration

```text
                 User
                   │
                   ▼
            Ingress-NGINX
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
Frontend Service       Backend Service
        │                     │
        ▼                     ▼
 Frontend Pods         Backend Pods
```

### After Migration

```text
                 User
                   │
                   ▼
             Istio Gateway
                   │
                   ▼
               HTTPRoute
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
Frontend Service       Backend Service
        │                     │
        ▼                     ▼
Frontend Pod + Envoy   Backend Pod + Envoy
```

---

# Migration Phases

## Phase 0 – Verify Existing Kubernetes Resources

Check that your application is running correctly before starting the migration.

### List all resources

```bash
kubectl get all -n dev
```

### Check existing Ingress resources

```bash
kubectl get ingress -n dev
```

Example:

```text
frontend-ingress
backend-ingress
```

### Check Services

```bash
kubectl get svc -n dev
```

Example:

```text
frontend   ClusterIP   80
backend    ClusterIP   5000
mongodb    ClusterIP   27017
```

---

# Phase 1 – Install Istio

## 1. Download Istio

```bash
curl -L https://istio.io/downloadIstio | sh -
```

Example:

```text
istio-1.30.3
```

---

## 2. Enter the Istio Directory

```bash
cd istio-1.30.3
```

---

## 3. Add `istioctl` to PATH

```bash
export PATH=$PWD/bin:$PATH
```

Verify installation:

```bash
istioctl version
```

---

# Phase 2 – Install Gateway API CRDs

Istio Gateway API requires Gateway API Custom Resource Definitions.

Install them:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
```

Verify:

```bash
kubectl get crd | grep gateway
```

Expected output:

```text
gateways.gateway.networking.k8s.io
httproutes.gateway.networking.k8s.io
```

---

# Phase 3 – Install Istio Control Plane

Install Istio using the Demo profile.

```bash
istioctl install --set profile=demo -y
```

Verify:

```bash
kubectl get pods -n istio-system
```

Expected:

```text
istiod
istio-ingressgateway
istio-egressgateway
```

---

# Phase 4 – Enable Automatic Sidecar Injection

Enable Envoy sidecar injection for your application namespace.

```bash
kubectl label namespace dev istio-injection=enabled --overwrite
```

Verify:

```bash
kubectl get ns dev --show-labels
```

Expected:

```text
istio-injection=enabled
```

---

# Phase 5 – Restart Application Pods

Existing pods will **not** receive Envoy automatically.

Restart every deployment.

```bash
kubectl rollout restart deployment frontend -n dev

kubectl rollout restart deployment backend -n dev

kubectl rollout restart deployment mongodb -n dev
```

Verify:

```bash
kubectl get pods -n dev
```

Before:

```text
frontend   1/1
backend    1/1
mongodb    1/1
```

After:

```text
frontend   2/2
backend    2/2
mongodb    2/2
```

The second container is:

```text
istio-proxy (Envoy)
```

---

# Phase 6 – Disable NGINX Ingress

Since Istio Gateway will handle external traffic, disable the existing Ingress.

## Frontend

### Before

```yaml
ingress:
  enabled: true
```

### After

```yaml
ingress:
  enabled: false
```

---

## Backend

### Before

```yaml
ingress:
  enabled: true
```

### After

```yaml
ingress:
  enabled: false
```

---

# Phase 7 – Enable HTTPRoute

## Frontend `values.yaml`

```yaml
httpRoute:
  enabled: true

  parentRefs:
    - name: smc-gateway
      sectionName: http

  hostnames:
    - smc.local

  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /

      backendRefs:
        - name: frontend
          port: 80
```

---

## Backend `values.yaml`

```yaml
httpRoute:
  enabled: true

  parentRefs:
    - name: smc-gateway
      sectionName: http

  hostnames:
    - smc.local

  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api

      backendRefs:
        - name: backend
          port: 5000
```

---

# Phase 8 – Create an Istio Gateway

Create **gateway.yaml**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway

metadata:
  name: smc-gateway
  namespace: dev

spec:
  gatewayClassName: istio

  listeners:
    - name: http
      protocol: HTTP
      port: 80
      hostname: smc.local

      allowedRoutes:
        namespaces:
          from: Same
```

Apply it:

```bash
kubectl apply -f gateway.yaml
```

Verify:

```bash
kubectl get gateway -n dev
```

Expected:

```text
smc-gateway
```

---

# Phase 9 – Upgrade Helm Releases

Upgrade the Frontend chart.

```bash
helm upgrade frontend ./frontend -n dev
```

Upgrade the Backend chart.

```bash
helm upgrade backend ./backend -n dev
```

---

# Phase 10 – Verify the Migration

Check Gateway:

```bash
kubectl get gateway -n dev
```

Check HTTPRoutes:

```bash
kubectl get httproute -n dev
```

Check Pods:

```bash
kubectl get pods -n dev
```

Verify Envoy proxies:

```bash
istioctl proxy-status
```

---

# Phase 11 – Access the Application

Check the Istio Ingress Gateway service.

```bash
kubectl get svc -n istio-system
```

## If Using NodePort

Example:

```text
80:31727
```

Access:

```text
http://EC2-IP:31727
```

---

## If Using Port Forward

```bash
kubectl port-forward \
  -n istio-system \
  svc/istio-ingressgateway \
  8080:80 \
  --address=0.0.0.0
```

Open in browser:

```text
http://EC2-IP:8080
```

---

# Phase 12 – Install Istio Observability Add-ons

Install the official add-ons.

```bash
kubectl apply -f samples/addons
```

Verify:

```bash
kubectl get pods -n istio-system
```

Expected components:

```text
kiali
prometheus
grafana
jaeger
```

---

# Final Migration Architecture

## Before

```text
Browser
   │
   ▼
Ingress-NGINX
   │
   ▼
Services
   │
   ▼
Pods
```

---

## After

```text
Browser
   │
   ▼
Istio Gateway
   │
   ▼
HTTPRoute
   │
   ▼
Services
   │
   ▼
Pods
   │
   ▼
Envoy Sidecar
```

---

# Migration Checklist

* ✅ Existing Kubernetes application is running
* ✅ Install Istio
* ✅ Install Gateway API CRDs
* ✅ Enable namespace sidecar injection
* ✅ Restart application pods
* ✅ Verify Envoy sidecar injection
* ✅ Disable `ingress.enabled`
* ✅ Enable `httpRoute.enabled`
* ✅ Create Istio Gateway
* ✅ Deploy HTTPRoutes
* ✅ Upgrade Helm releases
* ✅ Test application routing
* ✅ Install Kiali, Prometheus, Grafana, and Jaeger

---

# Conclusion

Migrating from **NGINX Ingress** to the **Istio Gateway API** provides a modern, service-mesh-based networking architecture.

With this migration you gain:

* Gateway API support
* Envoy sidecar proxy
* Advanced traffic management
* Built-in security
* Better observability
* Production-ready service communication

This migration is the foundation for implementing advanced Istio features such as:

* VirtualService
* DestinationRule
* Canary Deployment
* Retry & Timeout
* Circuit Breaking
* mTLS
* Authorization Policies
* Service Observability
