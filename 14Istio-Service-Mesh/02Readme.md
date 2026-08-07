# Istio Bookinfo Setup on Kind Kubernetes Cluster

## Architecture

```
                    Internet / Browser
                           │
                           ▼
                Istio Ingress Gateway
                           │
                           ▼
                      HTTPRoute Rules
                           │
                           ▼
                  Productpage Service
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
       Envoy Proxy                  Application
            │
            ▼
  Reviews → Ratings → Details
```

---

# Prerequisites

- Docker Installed
- Kind Installed
- kubectl Installed
- Running Kind Cluster

Verify Cluster

```bash
kind get clusters
kubectl get nodes
kubectl cluster-info
```

Expected Output

```text
kind

NAME
kind-control-plane
kind-worker
```

---

# Step 1: Download Istio

Download the latest Istio release.

```bash
curl -L https://istio.io/downloadIstio | sh -
```

### Purpose

This downloads the complete Istio package.

Verify

```bash
ls
```

Output

```text
istio-1.30.3
```

---

# Step 2: Enter Istio Directory

```bash
cd istio-1.30.3
```

Verify

```bash
ls
```

Expected

```text
bin
samples
manifests
tools
```

### Folder Description

| Folder | Purpose |
|---------|----------|
| bin | Contains the `istioctl` CLI |
| manifests | Kubernetes YAML files used during installation |
| samples | Sample applications (Bookinfo, Sleep, Httpbin) |
| tools | Additional utilities |

---

# Step 3: Configure istioctl

Add istioctl to PATH.

```bash
export PATH=$PWD/bin:$PATH
```

### Purpose

Allows the terminal to recognize the `istioctl` command.

Verify

```bash
which istioctl
```

Example

```text
/home/ubuntu/istio-1.30.3/bin/istioctl
```

Check Version

```bash
istioctl version
```

Output

```text
client version: 1.30.x
```

---

# Step 4: Install Gateway API CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
```

## Purpose

Gateway API adds new Kubernetes resource types.

Without CRDs Kubernetes only understands

- Pod
- Service
- Deployment
- Secret
- ConfigMap

After installation Kubernetes also understands

- Gateway
- GatewayClass
- HTTPRoute
- TCPRoute
- ReferenceGrant

Verify

```bash
kubectl get crd | grep gateway
```

Expected

```text
gatewayclasses.gateway.networking.k8s.io
gateways.gateway.networking.k8s.io
httproutes.gateway.networking.k8s.io
```

---

# Step 5: Install Istio

```bash
istioctl install --set profile=demo -y
```

## Purpose

Installs the complete Istio control plane.

Installed Components

- istiod
- Ingress Gateway
- Webhooks
- ConfigMaps
- Secrets
- RBAC
- Services
- Deployments

---

Verify Namespace

```bash
kubectl get ns
```

Expected

```text
istio-system
```

---

Verify Pods

```bash
kubectl get pods -n istio-system
```

Expected

```text
istiod

istio-ingressgateway
```

---

Verify Services

```bash
kubectl get svc -n istio-system
```

Expected

```text
istiod

istio-ingressgateway
```

---

Verify Deployments

```bash
kubectl get deploy -n istio-system
```

Expected

```text
istiod

istio-ingressgateway
```

---

# Step 6: Enable Automatic Sidecar Injection

```bash
kubectl label namespace default istio-injection=enabled
```

## Purpose

Automatically inject Envoy Sidecar into every new Pod created inside the namespace.

Without Injection

```
Pod
```

With Injection

```
Pod

+

Envoy Sidecar
```

Verify

```bash
kubectl describe namespace default
```

Expected

```text
Labels

istio-injection=enabled
```

---

# Step 7: Deploy Bookinfo Application

Deploy Sample Application

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

Components Deployed

- productpage
- reviews
- ratings
- details

Verify

```bash
kubectl get pods
```

Example

```text
productpage
reviews
ratings
details
```

Check Sidecar

```bash
kubectl get pods
```

READY should show

```text
2/2
```

or

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'
```

Example

```text
productpage istio-proxy
```

---

# Step 8: Deploy Gateway and HTTPRoute

```bash
kubectl apply -f samples/bookinfo/gateway-api/bookinfo-gateway.yaml
```

## Purpose

Creates

- Gateway
- HTTPRoute

These resources tell Istio how incoming traffic should reach Bookinfo.

Verify Gateway

```bash
kubectl get gateway
```

Example

```text
bookinfo-gateway
```

Verify HTTPRoute

```bash
kubectl get httproute
```

Example

```text
bookinfo
```

---

# Step 9: Change Gateway Service Type

```bash
kubectl annotate gateway bookinfo-gateway \
networking.istio.io/service-type=ClusterIP \
--namespace=default
```

## Purpose

Configure the generated Istio Gateway Service as ClusterIP.

Verify

```bash
kubectl get svc
```

Example

```text
bookinfo-gateway-istio   ClusterIP
```

---

# Step 10: Access Bookinfo using Port Forward

```bash
kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```

Local Access

```
http://localhost:8080/productpage
```

---

To allow external access

```bash
kubectl port-forward svc/bookinfo-gateway-istio 8080:80 --address=0.0.0.0
```

Access

```
http://EC2_PUBLIC_IP:8080/productpage
```

> Make sure TCP Port **8080** is allowed in the EC2 Security Group.

---

# Step 11: Verify Proxy Connection

```bash
istioctl proxy-status
```

Expected

```text
productpage

reviews

ratings

details
```

Status

```text
SYNCED
```

This confirms every Envoy Proxy is connected with `istiod`.

---

# Step 12: Verify Installation

```bash
istioctl verify-install
```

Expected

```text
✔ No issues found
```

---

# Step 13: Analyze Configuration

```bash
istioctl analyze
```

Expected

```text
✔ No validation issues found
```

---

# Complete Traffic Flow

```
Browser

        │

        ▼

EC2 Public IP : 8080

        │

        ▼

kubectl Port Forward

        │

        ▼

bookinfo-gateway-istio Service

        │

        ▼

Istio Gateway

        │

        ▼

HTTPRoute

        │

        ▼

Productpage Service

        │

        ▼

Productpage Pod

        │

 ┌──────┴────────┐

 ▼               ▼

Application   Envoy Proxy

        │

        ▼

Reviews

        │

        ▼

Ratings

        │

        ▼

Details

        │

        ▼

Response

        │

        ▼

Envoy Proxy

        │

        ▼

Gateway

        │

        ▼

Browser
```

---

# Useful Verification Commands

```bash
kubectl get nodes

kubectl get ns

kubectl get pods -A

kubectl get svc -A

kubectl get gateway

kubectl get httproute

kubectl get gatewayclass

kubectl get crd | grep gateway

kubectl describe namespace default

istioctl version

istioctl proxy-status

istioctl analyze

istioctl verify-install
```

---

# Summary

- Download Istio package.
- Configure `istioctl`.
- Install Gateway API CRDs.
- Install Istio Control Plane.
- Enable automatic Sidecar Injection.
- Deploy the Bookinfo sample application.
- Create Gateway and HTTPRoute.
- Access the application through the Istio Ingress Gateway.
- Verify Envoy proxy connections and Istio installation.