# Helm Summary

## 1. What is Helm?

Helm is the most popular package manager for Kubernetes.

A simple definition:

> Helm is a package manager, template engine, and release manager for Kubernetes.

Similar to:

- apt for Ubuntu
- yum for CentOS
- npm for Node.js
- Maven for Java

Helm manages Kubernetes applications using **Charts**.

---

## 2. Why Do We Need Helm?

Without Helm:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml
kubectl apply -f ingress.yaml
```

Problems:

- Too many YAML files
- Hard to manage different environments
- No version history
- No easy rollback

Helm solves these problems.

---

## 3. Core Purpose of Helm

### A. Package Manager

A Helm Chart is similar to a software package.

Example:

```bash
helm install redis bitnami/redis
```

Instead of manually creating:

- Deployment
- Service
- ConfigMap
- Secret
- PVC

Helm installs everything together.

---

### B. Template Engine

Helm supports variables.

Template:

```yaml
image: {{ .Values.image.tag }}
```

Values:

```yaml
image:
  tag: v1.0.0
```

Rendered result:

```yaml
image: v1.0.0
```

Different environments can use different values files:

```text
values-dev.yaml
values-sit.yaml
values-prod.yaml
```

Same chart, different configurations.

---

### C. Release Management

Helm tracks deployment history.

```bash
helm install order-prod ...
```

Creates:

```text
Release: order-prod
Revision: 1
```

After upgrades:

```text
Revision 1
Revision 2
Revision 3
```

View history:

```bash
helm history order-prod
```

Rollback:

```bash
helm rollback order-prod 1
```

---

## 4. Important Concepts

### Chart

A package containing:

```text
templates/
values.yaml
Chart.yaml
```

Think:

```text
Chart = Blueprint
```

---

### Release

An installed instance of a chart.

Example:

```bash
helm install order-dev order-chart
helm install order-prod order-chart
```

Result:

```text
Release:
├── order-dev
└── order-prod
```

Think:

```text
Chart = Class
Release = Object
```

---

### Namespace

Kubernetes isolation boundary.

Example:

```bash
helm install order-prod order-chart \
  -n prod
```

`-n` means namespace.

---

### Values

Environment-specific configuration.

Example:

```yaml
replicaCount: 3

image:
  tag: v1.2.0
```

---

## 5. Core Commands

### Install

```bash
helm install order-prod company/order-service \
  -n prod \
  -f values-prod.yaml
```

---

### Upgrade

```bash
helm upgrade order-prod company/order-service \
  -n prod \
  -f values-prod.yaml
```

Used when:

- Chart changed
- Values changed
- Both changed

---

### History

```bash
helm history order-prod
```

Shows revisions.

---

### Rollback

```bash
helm rollback order-prod 2
```

Rollback to a previous revision.

---

### Uninstall

```bash
helm uninstall order-prod
```

Removes the release.

---

## 6. How Upgrade Works

```text
Chart + Values
       ↓
Render Templates
       ↓
Generate YAML
       ↓
Send to Kubernetes API Server
       ↓
Kubernetes Updates Resources
```

Helm does not execute kubectl.

Both Helm and kubectl communicate directly with the Kubernetes API Server.

---

## 7. How Helm Stores Versions

Helm stores release history inside Kubernetes Secrets.

Example:

```text
sh.helm.release.v1.order-prod.v1
sh.helm.release.v1.order-prod.v2
sh.helm.release.v1.order-prod.v3
```

Each revision contains:

- Chart
- Values
- Rendered manifests
- Metadata

This enables rollback.

---

## 8. Helm in Microservices

Typical enterprise pattern:

```text
order-service
    ↓
order-chart
    ↓
order-prod release
```

```text
payment-service
    ↓
payment-chart
    ↓
payment-prod release
```

Usually:

```text
1 Service
    =
1 Chart
    =
1 Release
```

Benefits:

- Independent deployment
- Independent scaling
- Independent rollback
- Independent versioning
