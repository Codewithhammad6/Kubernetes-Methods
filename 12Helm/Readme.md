# 🚀 Helm in Kubernetes

Helm is the **Package Manager for Kubernetes**. It helps package, install, upgrade, and manage Kubernetes applications using reusable templates called **Helm Charts**.

---

## 📚 Table of Contents

- [What is Helm?](#-what-is-helm)
- [Why Helm?](#-why-helm)
- [Helm Architecture](#-helm-architecture)
- [Prerequisites](#-prerequisites)
- [Install Helm](#-install-helm)
- [Create Your First Chart](#-create-your-first-chart)
- [Chart Structure](#-chart-structure)
- [Chart.yaml](#-chartyaml)
- [version vs appVersion](#-version-vs-appversion)
- [values.yaml](#-valuesyaml)
- [Update Service Template](#-update-service-template)
- [Package a Chart](#-package-a-chart)
- [Install a Release](#-install-a-release)
- [Check Kubernetes Resources](#-check-kubernetes-resources)
- [Upgrade a Release](#-upgrade-a-release)
- [Rollback](#-rollback)
- [Release Revisions](#-release-revisions)
- [Release History](#-release-history)
- [List Releases](#-list-releases)
- [Show Values & Manifest](#-show-values--manifest)
- [Uninstall a Release](#-uninstall-a-release)
- [Helm Workflow](#-helm-workflow)
- [Production Best Practice](#-production-best-practice)
- [Useful Commands](#-useful-commands)
- [Summary](#-summary)

---

# 📦 What is Helm?

Helm is the **Package Manager for Kubernetes**.

Similar to:

- **apt** → Ubuntu
- **yum** → CentOS
- **npm** → Node.js
- **pip** → Python

Instead of managing multiple Kubernetes YAML files manually, Helm packages them into a reusable **Chart**.

---

# ❓ Why Helm?

## Without Helm

You must create and apply every Kubernetes resource manually.

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f ingress.yml
kubectl apply -f configmap.yml
kubectl apply -f secret.yml
```

---

## With Helm

Everything can be deployed with a single command.

```bash
helm install my-app apache-helm
```

---

# 🏗️ Helm Architecture

```text
Developer
      │
      ▼
Helm Chart
      │
      ▼
Helm
      │
      ▼
Kubernetes API
      │
      ▼
Cluster
```

---

# 📋 Prerequisites

Before using Helm, make sure you have:

- Docker
- kubectl
- Kubernetes Cluster
- Helm

---

# ⚙️ Install Helm

### Download the installation script

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

### Give execute permission

```bash
chmod 700 get_helm.sh
```

### Install Helm

```bash
./get_helm.sh
```

### Verify Installation

```bash
helm version
```

Example Output

```text
version.BuildInfo{
Version:"v3.x.x"
}
```

---

# 🎯 Create Your First Chart

```bash
helm create apache-helm
```

---

# 📂 Chart Structure

```text
apache-helm/

├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

---

# 📄 Chart.yaml

Contains metadata about the Helm Chart.

Example:

```yaml
apiVersion: v2
name: apache-helm
description: Apache Helm Chart
type: application
version: 0.1.0
appVersion: "1.16.0"
```

---

# 🔄 version vs appVersion

## version

Represents the **Helm Chart version**.

```yaml
version: 0.1.0
```

Whenever the chart changes:

```text
0.1.0
   ↓
0.1.1
```

---

## appVersion

Represents the **Application version**.

```yaml
appVersion: "1.16.0"
```

Example:

```text
Apache 2.4
    ↓
Apache 2.5
```

or

```text
My App v1
    ↓
My App v2
```

---

# ⚙️ values.yaml

Stores configurable values used by templates.

```yaml
replicaCount: 2

image:
  repository: httpd
  tag: "2.4"

service:
  type: ClusterIP
  port: 80
  targetPort: 80
```

---

# ✏️ Update Service Template

Open:

```text
templates/service.yaml
```

Replace:

```yaml
targetPort: {{ .Values.service.targetPort }}
```

Now `targetPort` is controlled from `values.yaml`.

---

# 📦 Package a Chart

```bash
helm package apache-helm/
```

Output:

```text
apache-helm-0.1.0.tgz
```

---

# 🚀 Install a Release

## Development

```bash
helm install dev-app apache-helm \
--namespace dev-apache \
--create-namespace
```

Release Name

```text
dev-app
```

Namespace

```text
dev-apache
```

---

## Production

```bash
helm install prd-app apache-helm \
--namespace prd-apache \
--create-namespace
```

Same chart installed twice:

```text
apache-helm
      │
      ├── dev-app
      │      └── dev-apache
      │
      └── prd-app
             └── prd-apache
```

---

# 🔍 Check Kubernetes Resources

## Namespaces

```bash
kubectl get ns
```

Example:

```text
dev-apache
prd-apache
```

---

## Pods

Development

```bash
kubectl get pods -n dev-apache
```

Production

```bash
kubectl get pods -n prd-apache
```

---

## Deployments

```bash
kubectl get deployment -n dev-apache

kubectl get deployment -n prd-apache
```

---

# ⬆️ Upgrade a Release

Before:

```yaml
replicaCount: 2
```

After:

```yaml
replicaCount: 3
```

Upgrade:

```bash
helm upgrade prd-app apache-helm -n prd-apache
```

Verify:

```bash
kubectl get pods -n prd-apache
```

Example:

```text
3 Running Pods
```

---

# 🔙 Rollback

Example:

Revision 1

```text
2 Replicas
```

Revision 2

```text
3 Replicas
```

Rollback:

```bash
helm rollback prd-app 1 -n prd-apache
```

Verify:

```bash
kubectl get pods -n prd-apache
```

Result:

```text
Back to 2 Replicas
```

---

# 📝 Release Revisions

Every upgrade creates a new revision.

```text
Revision 1
     │
 Install
     │
Revision 2
     │
 Upgrade
     │
Revision 3
     │
 Rollback
```

---

# 📜 Release History

```bash
helm history prd-app -n prd-apache
```

Example:

```text
REVISION

1
2
3
```

---

# 📦 List Releases

```bash
helm list
```

Specific Namespace:

```bash
helm list -n dev-apache

helm list -n prd-apache
```

---

# 📄 Show Values & Manifest

Current values:

```bash
helm get values prd-app -n prd-apache
```

Rendered manifests:

```bash
helm get manifest prd-app -n prd-apache
```

---

# 🗑️ Uninstall a Release

Development

```bash
helm uninstall dev-app -n dev-apache
```

Production

```bash
helm uninstall prd-app -n prd-apache
```

Only the release is removed.

The chart remains available.

---

# 🔁 Helm Workflow

```text
Install Helm
      │
      ▼
Create Chart
      │
      ▼
Edit values.yaml
      │
      ▼
Edit Templates
      │
      ▼
Package Chart
      │
      ▼
Install Release
      │
      ▼
Upgrade Release
      │
      ▼
Rollback (if needed)
      │
      ▼
Uninstall
```

---

# 🏭 Production Best Practice

Instead of modifying `values.yaml` every time, create separate values files.

```text
apache-helm/

├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-stage.yaml
├── values-prod.yaml
└── templates/
```

### Development

```bash
helm install dev-app apache-helm \
-n dev-apache \
-f values-dev.yaml
```

### Staging

```bash
helm install stage-app apache-helm \
-n stage-apache \
-f values-stage.yaml
```

### Production

```bash
helm install prd-app apache-helm \
-n prd-apache \
-f values-prod.yaml
```

Each environment now has its own configuration.

| Environment | Replicas |
|-------------|----------|
| Development | 1 |
| Staging | 2 |
| Production | 3 |

---

# 🛠️ Useful Commands

### Check Version

```bash
helm version
```

### Create Chart

```bash
helm create apache-helm
```

### Package Chart

```bash
helm package apache-helm/
```

### Install

```bash
helm install dev-app apache-helm
```

### Upgrade

```bash
helm upgrade dev-app apache-helm
```

### Rollback

```bash
helm rollback dev-app 1
```

### History

```bash
helm history dev-app
```

### List Releases

```bash
helm list
```

### Show Values

```bash
helm get values dev-app
```

### Show Manifest

```bash
helm get manifest dev-app
```

### Uninstall

```bash
helm uninstall dev-app
```

---

# 📌 Summary

- Helm is the **Package Manager** for Kubernetes.
- A **Helm Chart** packages Kubernetes resources into reusable templates.
- `helm create` generates a chart scaffold.
- `values.yaml` stores configurable values.
- Templates use values from `values.yaml`.
- `helm install` deploys a release.
- A single chart can be installed multiple times using different release names and namespaces.
- `helm upgrade` updates an existing release.
- `helm rollback` restores a previous revision.
- `helm history` shows release revisions.
- `helm uninstall` removes a release.
- In production, use dedicated values files (`values-dev.yaml`, `values-stage.yaml`, `values-prod.yaml`) instead of modifying `values.yaml` for every environment.