# Kubernetes Autoscaling (HPA & VPA)

Kubernetes provides **Autoscaling** to automatically handle application load.

There are two main types:

- **HPA (Horizontal Pod Autoscaler)** → Scales the number of Pods.
- **VPA (Vertical Pod Autoscaler)** → Scales CPU and Memory of a Pod.

---

# 1. Horizontal Pod Autoscaler (HPA)

**HPA automatically increases or decreases the number of Pods based on CPU or Memory usage.**

## Example

Deployment:

```yaml
replicas: 2
```

Current Pods:

```text
Backend-1
Backend-2
```

Suddenly 10,000 users visit the application.

CPU usage becomes **80%**.

HPA automatically scales the deployment.

```text
Backend-1
Backend-2
Backend-3
Backend-4
Backend-5
```

When traffic becomes normal again:

```text
Backend-1
Backend-2
```

Pods are reduced automatically.

---

## HPA Scales

- ✅ Pods
- ❌ CPU
- ❌ Memory

---

## HPA Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: backend-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment

  minReplicas: 2
  maxReplicas: 10

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Meaning:

> If average CPU usage becomes greater than **70%**, Kubernetes automatically creates more Pods.

---

# 2. Vertical Pod Autoscaler (VPA)

**VPA automatically increases or decreases CPU and Memory of the existing Pod.**

Current resources:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

After VPA:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

The number of Pods remains the same.

---

## VPA Scales

- ❌ Pods
- ✅ CPU
- ✅ Memory

---

# HPA vs VPA

| HPA | VPA |
|------|-----|
| Increases Pods | Increases CPU/RAM |
| Horizontal Scaling | Vertical Scaling |
| Best for traffic | Best for resource optimization |

---

# Real Project

### React Frontend

- ✅ HPA
- ❌ VPA (Usually not required)

Reason:

Traffic increases, so more Pods are needed.

---

### Backend (Node.js / Django)

- ✅ HPA
- ✅ VPA (Sometimes)

Reason:

Backend handles API requests.

---

### MySQL / PostgreSQL / MongoDB

- ❌ HPA (Usually not used)
- ✅ VPA

Reason:

Databases are stateful applications. Increasing CPU and RAM is usually better than creating multiple Pods.

---

# Simple Diagram

## HPA

```text
2 Pods
   │
CPU 80%
   │
   ▼
5 Pods
```

---

## VPA

```text
1 Pod
CPU : 250m
RAM : 256Mi
      │
Heavy Load
      │
      ▼
1 Pod
CPU : 500m
RAM : 512Mi
```

---

# Metrics Server

HPA requires **Metrics Server** to collect CPU and Memory usage.

Without Metrics Server:

```text
HPA ❌ Not Working
```

With Metrics Server:

```text
Pods
 │
 ▼
Kubelet
 │
 ▼
Metrics Server
 │
 ▼
HPA
```

---

# Install Metrics Server (Kind Cluster)

Apply Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

# Edit Metrics Server Deployment

```bash
kubectl -n kube-system edit deployment metrics-server
```

Under:

```yaml
containers:
```

Add:

```yaml
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP
```

---

# Restart Metrics Server

```bash
kubectl -n kube-system rollout restart deployment metrics-server
```

---

# Verify Metrics Server

```bash
kubectl get pods -n kube-system
```

Example:

```text
NAME                              READY   STATUS
metrics-server-xxxxxxxxxx         1/1     Running
```

---

# How HPA Works

Every Kubernetes node has its own **Kubelet**.

Example:

```text
Control Plane
       │
       │
 ┌─────┴─────┐
 │           │
 ▼           ▼
Worker-1   Worker-2
Kubelet    Kubelet
```

Each Kubelet continuously collects CPU and Memory usage of Pods running on its node.

Example:

```text
Worker-1

Frontend
CPU : 20%
RAM : 150Mi

MySQL
CPU : 40%
RAM : 500Mi
```

```text
Worker-2

Backend-1
CPU : 80%

Backend-2
CPU : 75%

Backend-3
CPU : 82%
```

Metrics Server collects metrics from every Kubelet.

```text
Worker-1 Kubelet
        │
        ├─────────────┐
        │             │
Worker-2 Kubelet      │
        │             │
        └──────────► Metrics Server
```

Then HPA asks Metrics Server:

```text
Backend CPU Usage?
```

Metrics Server replies:

```text
Backend-1 = 80%

Backend-2 = 75%

Backend-3 = 82%

Average = 79%
```

HPA compares it with:

```yaml
averageUtilization: 70
```

Since **79% > 70%**, HPA tells Deployment to create more Pods.

---

# Complete HPA Flow

```text
Pods
 │
 ▼
Kubelet
 │
 ▼
Metrics Server
 │
 ▼
HPA
 │
 ▼
Deployment
 │
 ▼
ReplicaSet
 │
 ▼
New Pods
```

---

# Remember

- **Kubelet** → Collects Pod CPU & Memory usage.
- **Metrics Server** → Collects metrics from all Kubelets.
- **HPA** → Decides whether Pods should increase or decrease.
- **Deployment / ReplicaSet** → Creates or removes Pods automatically.








---

# Apache in Kubernetes

Apache is a **Web Server**.

It receives requests from the browser and serves the website to users.

Example:

```text
User Browser
      │
      ▼
Apache Web Server
      │
      ▼
Website (HTML, CSS, JS, PHP)
```

---

## What does Apache do?

When a user opens:

```text
http://example.com
```

The request goes to Apache.

Apache:

- Receives the request
- Reads website files
- Sends the response back to the browser

---

## Common Uses of Apache

- Static Websites (HTML, CSS, JavaScript)
- PHP Applications (Laravel, WordPress)
- Reverse Proxy
- SSL (HTTPS)
- Virtual Hosts
- Load Balancing

---

## Apache vs Nginx

| Apache | Nginx |
|---------|-------|
| Older and very popular | Modern and high performance |
| Mostly used with PHP | Common with React, Node.js and Kubernetes |
| Supports `.htaccess` | Does not support `.htaccess` |
| Process/Thread based | Event-driven |
| Slower under heavy traffic | Faster under heavy traffic |

---

# Apache Architecture

```text
Browser
    │
    ▼
Apache Service
    │
    ▼
Apache Pod
```

---

# Project Structure

```text
k8s/
│
├── namespace.yml
├── deployment.yml
└── service.yml
```

---

# Namespace

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: apache
```

Apply:

```bash
kubectl apply -f namespace.yml
```

Check:

```bash
kubectl get ns
```

---

# Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: apache-deployment
  namespace: apache

spec:
  replicas: 1

  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache

    spec:
      containers:
      - name: apache
        image: httpd:latest

        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yml
```

Check:

```bash
kubectl get pods -n apache
```

---

# Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: apache-service
  namespace: apache

spec:
  selector:
    app: apache

  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f service.yml
```

Check:

```bash
kubectl get svc -n apache
```

---

# Access Apache in Browser

Since the service type is **ClusterIP**, it cannot be accessed directly from outside the cluster.

Use Port Forwarding:

```bash
kubectl port-forward svc/apache-service 30080:80 -n apache --address=0.0.0.0
```

Open:

```text
http://<EC2-Public-IP>:30080
```

You should see the default Apache page.

---

# Create HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: php-apache

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache

  minReplicas: 1
  maxReplicas: 10

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa.yml
```

Check:

```bash
kubectl get hpa -n apache
```

---

# Generate Load

Open a BusyBox pod:

```bash
kubectl run -it --rm load-generator \
  --image=busybox \
  -n apache \
  -- sh
```

Inside the pod run:

```bash
while true; do
  wget -q -O- http://apache-service > /dev/null
done
```

This continuously sends requests to the Apache Service.

---

# Monitor HPA

Watch HPA:

```bash
kubectl get hpa -n apache -w
```

Watch Pods:

```bash
kubectl get pods -n apache -w
```

When CPU usage increases, HPA automatically creates new Pods.

Example:

```text
Before

apache-pod-1
```

```text
After

apache-pod-1
apache-pod-2
apache-pod-3
```

Traffic is automatically distributed among all Pods.

---

# Troubleshooting

Check HPA:

```bash
kubectl get hpa -n apache
```

Check Pod CPU/Memory:

```bash
kubectl top pods -n apache
```

Describe HPA:

```bash
kubectl describe hpa php-apache -n apache
```

If `kubectl top` does not work, verify that **Metrics Server** is installed and running.

---

# Delete Load Generator

Stop the load test:

```bash
kubectl delete pod load-generator -n apache
```

---

# Complete Flow

```text
Browser
      │
      ▼
Apache Service
      │
      ▼
Apache Pod
      │
      ▼
CPU Usage Increases
      │
      ▼
Metrics Server
      │
      ▼
HPA
      │
      ▼
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
More Apache Pods
```

---

# Summary

- **Apache** serves web pages.
- **Deployment** manages Apache Pods.
- **Service** provides a stable network endpoint.
- **Port Forward** allows browser access.
- **Metrics Server** collects CPU and RAM usage.
- **HPA** automatically increases or decreases Pods based on CPU utilization.
- **BusyBox** is used to generate load for HPA testing.









# Vertical Pod Autoscaler (VPA) in Kubernetes

## What is VPA?

**Vertical Pod Autoscaler (VPA)** automatically adjusts the **CPU** and **Memory** resources of a Pod based on its actual usage.

Unlike HPA, which increases or decreases the number of Pods, VPA changes the resource allocation of existing Pods.

---


# How VPA Works

```
                +--------------------+
                |     Application    |
                +---------+----------+
                          |
                          v
                     Metrics Server
                          |
                          v
                   VPA Recommender
                          |
                          v
               CPU & Memory Analysis
                          |
                          v
             Recommended Resource Values
                          |
                          v
              VPA Updater (Auto Mode)
                          |
                          v
                 Restart Pod (If Needed)
                          |
                          v
             Pod Starts with New Resources
```

---

# Step 1: Verify Metrics Server

Check whether the Metrics Server is running correctly.

```bash
kubectl top nodes
```

```bash
kubectl top pods -A
```

If CPU and Memory usage are displayed, the Metrics Server is working properly.

---

# Step 2: Install Vertical Pod Autoscaler

Clone the Kubernetes Autoscaler repository.

```bash
git clone https://github.com/kubernetes/autoscaler.git
```

Move into the VPA directory.

```bash
cd autoscaler/vertical-pod-autoscaler
```

Install VPA.

```bash
./hack/vpa-up.sh
```

---

# Step 3: Verify Installation

```bash
kubectl get pods -n kube-system
```

You should see:

- vpa-admission-controller
- vpa-recommender
- vpa-updater

All Pods should be in the **Running** state.

---

# Step 4: Apache Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: apache-deployment
  namespace: apache

spec:
  replicas: 1

  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache

    spec:
      containers:
      - name: apache
        image: httpd:2.4

        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"

          limits:
            cpu: "500m"
            memory: "512Mi"
```

Apply Deployment.

```bash
kubectl apply -f deployment.yml
```

---

# Step 5: Create VPA

Create **vpa.yml**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler

metadata:
  name: apache-vpa
  namespace: apache

spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: apache-deployment

  updatePolicy:
    updateMode: Auto
```

Apply the configuration.

```bash
kubectl apply -f vpa.yml
```

---

# Step 6: Verify VPA

```bash
kubectl get vpa -n apache
```

View detailed recommendations.

```bash
kubectl describe vpa apache-vpa -n apache
```

Example Output:

```
Target

CPU : 250m
Memory : 256Mi

Lower Bound

CPU : 100m
Memory : 128Mi

Upper Bound

CPU : 500m
Memory : 512Mi
```

---

# Step 7: Generate Load

Create a BusyBox Pod.

```bash
kubectl run -it --rm load-generator \
  --image=busybox \
  -n apache \
  -- sh
```

Inside the container, continuously send requests.

```sh
while true; do
    wget -q -O- http://apache-service
done
```

This generates traffic so VPA can analyze resource usage.

---

# VPA Update Modes

## Off

- Only provides recommendations.
- Does not modify Pods.

```yaml
updatePolicy:
  updateMode: Off
```

---

## Initial

- Applies recommended resources only when a new Pod is created.
- Running Pods are not modified.

```yaml
updatePolicy:
  updateMode: Initial
```

---

## Auto

- Continuously monitors CPU and Memory usage.
- Automatically restarts Pods when necessary.
- New Pods start with updated CPU and Memory values.

```yaml
updatePolicy:
  updateMode: Auto
```

---

# How VPA Makes Decisions

```
Pod Running
      │
      ▼
Metrics Server Collects CPU & Memory
      │
      ▼
VPA Recommender Analyzes Usage
      │
      ▼
Recommendation Generated
      │
      ▼
Auto Mode?
      │
 ┌────┴─────┐
 │          │
Yes         No
 │          │
 ▼          ▼
Restart     Only Recommendation
Pod
 │
 ▼
Pod Starts with Updated Resources
```

---

# Useful Commands

Apply Deployment

```bash
kubectl apply -f deployment.yml
```

Apply VPA

```bash
kubectl apply -f vpa.yml
```

List VPAs

```bash
kubectl get vpa -n apache
```

Describe VPA

```bash
kubectl describe vpa apache-vpa -n apache
```

View Pod Metrics

```bash
kubectl top pods -n apache
```

View Node Metrics

```bash
kubectl top nodes
```

Delete VPA

```bash
kubectl delete -f vpa.yml
```

---

# Summary

- VPA automatically adjusts CPU and Memory resources.
- It relies on the Metrics Server for usage data.
- The VPA Recommender analyzes resource consumption.
- In **Auto** mode, Pods are restarted with updated resources.
- In **Off** mode, only recommendations are generated.
- In **Initial** mode, recommendations are applied only to newly created Pods.