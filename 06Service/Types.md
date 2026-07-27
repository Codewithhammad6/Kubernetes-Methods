# Kubernetes Service Types

The Service YAML is almost the same in both **Kind** and **Kubeadm**.

The main difference is **how you access the Service**.

---

# 1) ClusterIP Service

## YAML

Create **service-clusterip.yml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: nginx

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

Apply

```bash
kubectl apply -f service-clusterip.yml
```

Check

```bash
kubectl get svc -n nginx
```

Example

```text
NAME             TYPE        CLUSTER-IP
nginx-service    ClusterIP   10.96.70.111
```

---

# Flow

```text
Pod
 │
 ▼
ClusterIP Service
 │
 ▼
Another Pod
```

A browser cannot access a ClusterIP Service directly.

---

# Use Cases

- Backend APIs
- Databases
- Internal communication
- Microservices

---

# 2) NodePort Service

## YAML

Create **service-nodeport.yml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: nginx

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

Apply

```bash
kubectl apply -f service-nodeport.yml
```

Check

```bash
kubectl get svc -n nginx
```

Example

```text
NAME             TYPE       PORT(S)
nginx-service    NodePort   80:30080/TCP
```

---

# NodePort in Kind

Flow

```text
Browser
   │
   ▼
Laptop / EC2 :30080
   │
   ▼
Kind Node
   │
   ▼
NodePort Service
   │
   ▼
Pods
```

### Why do we use `extraPortMappings`?

When creating a Kind cluster, the Kubernetes node itself runs inside a Docker container.

To access a NodePort from your laptop or EC2 host, you usually map the host port to the Kind node container.

Example **config.yml**

```yaml
extraPortMappings:
- containerPort: 30080
  hostPort: 30080
```

Then create the cluster

```bash
kind create cluster --name mycluster --config config.yml
```

---

# NodePort in Kubeadm

Flow

```text
Browser
   │
   ▼
Worker Node IP :30080
   │
   ▼
NodePort Service
   │
   ▼
Pods
```

Access

```text
http://WORKER-IP:30080
```

No `extraPortMappings` are required because Kubernetes runs directly on the Worker Node.

---

# 3) LoadBalancer Service

> **Works automatically on cloud-managed Kubernetes (EKS, AKS, GKE).**
>
> On a self-managed **kubeadm** cluster, it **does not work automatically** unless you install a load balancer solution (such as MetalLB). It also does not work automatically on Kind.

## YAML

Create **service-loadbalancer.yml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: nginx

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
```

Apply

```bash
kubectl apply -f service-loadbalancer.yml
```

Check

```bash
kubectl get svc -n nginx
```

Example (Cloud)

```text
NAME             TYPE           EXTERNAL-IP
nginx-service    LoadBalancer   xxxxx.elb.amazonaws.com
```

---

# LoadBalancer Flow

```text
Internet User
      │
      ▼
Cloud Load Balancer
      │
      ▼
Kubernetes Service
      │
      ▼
Pods
```

---

# Kind vs Kubeadm

| Service Type | Kind | Kubeadm |
|--------------|------|----------|
| ClusterIP | ✅ Same | ✅ Same |
| NodePort | Requires `extraPortMappings` in most cases | Directly accessible using Worker Node IP |
| LoadBalancer | Requires additional setup (e.g. MetalLB) | Requires additional setup (e.g. MetalLB) |

---

# Managed Kubernetes (Cloud)

| Platform | LoadBalancer Support |
|----------|----------------------|
| Amazon EKS | ✅ Automatic |
| Azure AKS | ✅ Automatic |
| Google GKE | ✅ Automatic |
| Kind | ❌ Extra setup required |
| Kubeadm | ❌ Extra setup required |

---

# Quick Revision

```
ClusterIP
        │
        ▼
Internal Communication

NodePort
        │
        ▼
Node IP + Port

LoadBalancer
        │
        ▼
Internet → Cloud Load Balancer → Service → Pods
```

---

# One-Line Revision

- **ClusterIP** → Communication inside the Kubernetes cluster.
- **NodePort** → Access the application using the Node IP and Port.
- **LoadBalancer** → Exposes the application to the Internet using a cloud load balancer.
- **Kind** → NodePort usually requires `extraPortMappings`.
- **Kubeadm** → NodePort works directly with the Worker Node IP.
- **EKS / AKS / GKE** → LoadBalancer works automatically with cloud integration.