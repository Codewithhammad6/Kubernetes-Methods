# Kubernetes Probes

## What is a Probe?

A **Probe** is a **Health Check** in Kubernetes.

Kubernetes continuously checks whether your application is healthy and takes action if something goes wrong.

Simple question Kubernetes asks:

> **"Is my application working correctly?"**

If the answer is **No**, Kubernetes performs the required action automatically.

---

# Types of Probes

Kubernetes has **3 types of Probes**.

1. Liveness Probe
2. Readiness Probe
3. Startup Probe

---

# 1. Liveness Probe

### Purpose

Liveness Probe checks whether the application is **still alive**.

If the application hangs or crashes, Kubernetes automatically restarts the Pod.

---

## Example

```text
Node.js App

↓

Liveness Probe

↓

Is application responding?
```

If Yes

```text
Application Running

↓

Nothing happens
```

If No

```text
Application Hung

↓

Restart Pod
```

---

## Example YAML

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 3
```

### Meaning

```text
Wait 10 seconds

↓

Check every 5 seconds

↓

Fail 3 consecutive times

↓

Restart Pod
```

---

# 2. Readiness Probe

### Purpose

Readiness Probe checks whether the application is **ready to receive user requests**.

If the application is not ready, Kubernetes **does not send traffic** to that Pod.

---

## Example

Application takes **20 seconds** to start.

```text
Application

Starting...

↓

Readiness Probe

↓

Not Ready
```

Result:

```text
Service

❌ No Traffic
```

After the application becomes ready:

```text
Application Ready

↓

Readiness Probe

↓

Ready

↓

Service

✅ Send Traffic
```

---

## Example YAML

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 3
```

### Meaning

```text
Wait 5 seconds

↓

Check every 5 seconds

↓

If application is Ready

↓

Service sends traffic

↓

Otherwise

↓

No traffic
```

---

# 3. Startup Probe

### Purpose

Startup Probe checks whether the application has **finished starting**.

It is mainly used for **slow-starting applications**.

---

## Example

```text
MongoDB

Starting...

↓

Startup Probe

↓

Wait until application starts
```

If the application is still starting, Kubernetes waits instead of restarting the Pod.

---

## Example YAML

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80

  periodSeconds: 10
  failureThreshold: 30
```

### Meaning

```text
Every 10 seconds

↓

Check application

↓

Allow 30 failures

↓

Total wait = 300 seconds (5 minutes)

↓

If still not started

↓

Restart Pod
```

---

# All Three Probes Together

Production applications often use all three.

```yaml
containers:
- name: nginx
  image: nginx:latest

  ports:
  - containerPort: 80

  startupProbe:
    httpGet:
      path: /
      port: 80
    periodSeconds: 10
    failureThreshold: 30

  livenessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 10
    periodSeconds: 5

  readinessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 5
    periodSeconds: 5
```

---

# Complete Flow

```text
Pod Starts

      │

      ▼

Startup Probe

      │

Application Started?

      │

  Yes ▼

Readiness Probe

      │

Application Ready?

      │

  Yes ▼

Service Sends Traffic

      │

      ▼

Liveness Probe

      │

Application Healthy?

      │

  Yes ▼

Application Keeps Running

      │

  No ▼

Restart Pod
```

---

# Probe Comparison

| Probe           | Question                              | Action                     |
| --------------- | ------------------------------------- | -------------------------- |
| Liveness Probe  | Is the application still running?     | Restart the Pod            |
| Readiness Probe | Can the application receive requests? | If not, don't send traffic |
| Startup Probe   | Has the application started?          | Wait until it starts       |

---

# Which Applications Use Which Probe?

| Application       | Liveness | Readiness | Startup      |
| ----------------- | -------- | --------- | ------------ |
| Nginx             | ✅ Yes    | ✅ Yes     | ❌ Usually No |
| React Frontend    | ✅ Yes    | ✅ Yes     | ❌ Usually No |
| Node.js / Express | ✅ Yes    | ✅ Yes     | ⚠️ Sometimes |
| MongoDB           | ✅ Yes    | ✅ Yes     | ✅ Yes        |
| MySQL             | ✅ Yes    | ✅ Yes     | ✅ Yes        |
| Kafka             | ✅ Yes    | ✅ Yes     | ✅ Yes        |
| Spring Boot       | ✅ Yes    | ✅ Yes     | ✅ Yes        |

---

# MERN Stack Example

## Frontend (React + Nginx)

```text
Liveness Probe   ✅ Yes

Readiness Probe  ✅ Yes

Startup Probe    ❌ Usually Not Needed
```

---

## Backend (Node.js + Express)

```text
Liveness Probe   ✅ Yes

Readiness Probe  ✅ Yes

Startup Probe    ⚠️ Sometimes
```

If your backend starts in **2–3 seconds**, Startup Probe is usually not required.

If your backend performs database migrations, cache loading, or heavy initialization, Startup Probe is recommended.

---

## MongoDB

```text
Liveness Probe   ✅ Yes

Readiness Probe  ✅ Yes

Startup Probe    ✅ Yes
```

---

# Production Rule

Most production applications use:

```text
Liveness Probe

        +

Readiness Probe
```

For slow-starting applications like:

* MongoDB
* MySQL
* Kafka
* Elasticsearch
* Spring Boot

Also add:

```text
Startup Probe
```

---

# Quick Revision

```text
Liveness Probe
↓

Application Alive?

↓

Readiness Probe
↓

Ready for Traffic?

↓

Startup Probe
↓

Needed only for Slow Starting Apps
```

---

# One-Line Revision

* **Liveness Probe** → Restart the Pod if the application becomes unhealthy.
* **Readiness Probe** → Send traffic only when the application is ready.
* **Startup Probe** → Wait until the application starts.
