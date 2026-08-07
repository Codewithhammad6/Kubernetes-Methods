# Istio Gateway & Envoy Sidecar Architecture

## Overview

Istio is a **Service Mesh** that manages communication between microservices running inside a Kubernetes cluster.

Instead of every application implementing features like security, logging, retries, or load balancing, Istio handles these automatically using **Envoy Sidecar Proxies**.

---

# Architecture

```text
                    🌍 User
                       │
                       ▼
                Istio Gateway
                       │
                 (Entry Point)
                       │
                       ▼
               Frontend Service
                       │
                       ▼
        ┌─────────────────────────┐
        │      Frontend Pod        │
        │                         │
        │  ┌──────────────┐       │
        │  │ Frontend App │       │
        │  ├──────────────┤       │
        │  │ Envoy Proxy  │       │
        │  └──────────────┘       │
        └─────────────────────────┘
                       │
                       ▼
               Backend Service
                       │
                       ▼
        ┌─────────────────────────┐
        │      Backend Pod         │
        │                         │
        │  ┌──────────────┐       │
        │  │ Backend App  │       │
        │  ├──────────────┤       │
        │  │ Envoy Proxy  │       │
        │  └──────────────┘       │
        └─────────────────────────┘
                       │
                       ▼
               MongoDB Service
                       │
                       ▼
        ┌─────────────────────────┐
        │      MongoDB Pod         │
        │                         │
        │  ┌──────────────┐       │
        │  │   MongoDB    │       │
        │  ├──────────────┤       │
        │  │ Envoy Proxy  │       │
        │  └──────────────┘       │
        └─────────────────────────┘
```

---

# Components

## 1. User

The client (browser, mobile app, or API consumer) sends an HTTP request to the application.

Example:

```
https://myapp.com
```

The request does **not** directly reach the application.

---

## 2. Istio Gateway

The Istio Gateway is the **entry point** of the Kubernetes cluster.

Its responsibilities include:

- Receiving external traffic
- Listening on HTTP/HTTPS ports
- TLS termination
- Routing requests to the correct application
- Connecting external traffic to the Istio Service Mesh

Every external request first reaches the Istio Gateway.

```
Internet
    │
    ▼
Istio Gateway
```

---

## 3. HTTPRoute / VirtualService

After receiving a request, the Gateway checks routing rules.

Example:

```
/      → Frontend Service

/api   → Backend Service
```

The Gateway decides which Kubernetes Service should receive the request.

---

## 4. Kubernetes Service

A Kubernetes Service provides a stable network endpoint.

Examples:

```
Frontend Service
Backend Service
MongoDB Service
```

The Service selects one of the available Pods.

```
Frontend Service
        │
        ▼
Frontend Pod
```

---

# Pod Structure

Every application Pod contains **two containers**.

Example:

```
Frontend Pod

├── Frontend Application
└── Envoy Sidecar
```

```
Backend Pod

├── Backend Application
└── Envoy Sidecar
```

```
MongoDB Pod

├── MongoDB
└── Envoy Sidecar
```

The Envoy Proxy is deployed automatically by Istio.

---

# What is an Envoy Sidecar?

The Envoy Sidecar is a lightweight proxy running alongside every application container.

It intercepts **all incoming and outgoing network traffic**.

Applications communicate through Envoy instead of directly communicating with each other.

---

# Request Flow

## Step 1

User opens the application.

```
Browser
   │
   ▼
Istio Gateway
```

---

## Step 2

Gateway checks routing rules.

```
HTTPRoute

↓

Frontend Service
```

---

## Step 3

The request reaches the Frontend Pod.

```
Frontend Service
        │
        ▼
Frontend Envoy
        │
        ▼
Frontend Application
```

---

## Step 4

Frontend calls the Backend API.

```
Frontend App
      │
      ▼
Frontend Envoy
      │
      ▼
Backend Envoy
      │
      ▼
Backend Application
```

---

## Step 5

Backend queries MongoDB.

```
Backend App
      │
      ▼
Backend Envoy
      │
      ▼
MongoDB Envoy
      │
      ▼
MongoDB
```

---

## Step 6

The response travels back using the same path.

```
MongoDB
   ▲
Envoy
   ▲
Backend
   ▲
Envoy
   ▲
Frontend
   ▲
Envoy
   ▲
Gateway
   ▲
Browser
```

---

# Why Every Pod Has an Envoy Proxy?

Every Pod receives its own Envoy Sidecar.

Example:

```
Frontend Pod 1
├── Frontend
└── Envoy

Frontend Pod 2
├── Frontend
└── Envoy

Frontend Pod 3
├── Frontend
└── Envoy

Backend Pod 1
├── Backend
└── Envoy

Backend Pod 2
├── Backend
└── Envoy

MongoDB Pod
├── MongoDB
└── Envoy
```

Each Pod has its own independent proxy.

---

# Responsibilities of Envoy Proxy

Envoy automatically provides:

- Service Discovery
- Load Balancing
- mTLS Encryption
- Authentication
- Authorization
- Retry Policies
- Timeouts
- Circuit Breaking
- Traffic Routing
- Logging
- Metrics
- Distributed Tracing
- Fault Injection
- Health Checking
- Observability

Developers do not need to implement these features in application code.

---

# Without Istio

```
Frontend
      │
      ▼
Backend
      │
      ▼
MongoDB
```

Applications communicate directly.

The developer must implement:

- Security
- Retry Logic
- Logging
- Monitoring
- TLS
- Load Balancing

---

# With Istio

```
Frontend
     │
Envoy Proxy
     │
Backend
     │
Envoy Proxy
     │
MongoDB
```

Istio automatically manages communication.

---

# Real-Life Analogy

Imagine an airport.

```
Passenger
     │
Security Check
     │
Airplane
```

The passenger cannot board the plane without passing security.

Similarly,

```
Application
     │
Envoy Proxy
     │
Network
```

Every request must pass through Envoy before reaching another service.

---

# Key Points

- Istio Gateway is the external entry point of the cluster.
- Gateway receives all incoming traffic.
- HTTPRoute or VirtualService decides where requests should go.
- Kubernetes Services forward traffic to Pods.
- Every Pod has its own Envoy Sidecar.
- Envoy intercepts all incoming and outgoing traffic.
- Applications never communicate directly.
- Istio provides security, traffic management, observability, retries, and load balancing automatically.

---

# Complete Flow

```
User
 │
 ▼
Istio Gateway
 │
 ▼
HTTPRoute / VirtualService
 │
 ▼
Frontend Service
 │
 ▼
Frontend Envoy
 │
 ▼
Frontend Application
 │
 ▼
Frontend Envoy
 │
 ▼
Backend Envoy
 │
 ▼
Backend Application
 │
 ▼
Backend Envoy
 │
 ▼
MongoDB Envoy
 │
 ▼
MongoDB
```

---

# Summary

- **Istio Gateway** receives external traffic.
- **HTTPRoute / VirtualService** decides the destination.
- **Services** route requests to Pods.
- **Every Pod contains an Envoy Sidecar.**
- **Envoy controls all network communication.**
- Applications focus only on business logic, while Istio manages networking, security, observability, and traffic control.