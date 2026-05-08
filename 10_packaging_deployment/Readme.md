
Up to now, you learned:

* how Kubernetes works
* workloads
* networking
* security
* observability

Now the problem becomes:

```text id="80ql3i"
How do we deploy applications reliably,
consistently,
and automatically
across environments?
```

This phase is about:

* packaging applications
* environment management
* deployment automation
* Git-driven infrastructure

This is the foundation of:

* DevOps
* Platform Engineering
* Enterprise Kubernetes

---

# 📦 PHASE 8: Packaging & Deployment

---

# 🧠 Big Picture

Real production systems need:

* reusable deployments
* versioned releases
* automated pipelines
* rollback mechanisms
* environment consistency

Kubernetes packaging & deployment tools solve this.

---

# 🧩 High-Level Flow

```text id="3p7wod"
Developer writes code
        ↓
CI pipeline builds image
        ↓
Helm/Kustomize defines deployment
        ↓
Git stores desired state
        ↓
ArgoCD/Flux syncs cluster
        ↓
Kubernetes deploys application
```

---

# 🔹 1. Helm (VERY IMPORTANT)

Helm is:

> The package manager for Kubernetes

Equivalent ideas:

* apt (Ubuntu)
* yum (RHEL)
* npm (Node.js)

But for Kubernetes applications.

---

# 🧠 Problem Helm Solves

Without Helm:

* huge YAML duplication
* hard environment management
* difficult upgrades
* painful versioning

---

# 🧩 Example Without Helm

You may have:

* dev.yaml
* qa.yaml
* prod.yaml

All mostly duplicated.

---

# ✅ Helm Solution

Use:

* templates
* variables
* reusable charts

---

# 🔹 Core Helm Concepts

| Concept     | Meaning                        |
| ----------- | ------------------------------ |
| Chart       | Kubernetes application package |
| Release     | Installed instance of chart    |
| values.yaml | Configuration values           |
| Template    | Dynamic YAML                   |

---

# 🧠 Mental Model

```text id="i2ejtp"
Chart = application blueprint
Release = deployed application
```

---

# 🔹 Helm Chart Structure

```text id="i4gkgi"
myapp-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│    ├── deployment.yaml
│    ├── service.yaml
│    └── ingress.yaml
```

---

# 🔹 values.yaml

Centralized configuration.

Example:

```yaml id="9cwexu"
replicaCount: 3

image:
  repository: nginx
  tag: latest

service:
  port: 80
```

---

# 🔹 Template Example

```yaml id="dhz42p"
replicas: {{ .Values.replicaCount }}
```

Helm dynamically injects values.

---

# 🔥 Why Helm Is Huge

---

## ✅ Reusable deployments

---

## ✅ Easy upgrades

```bash id="pj2n8o"
helm upgrade
```

---

## ✅ Rollbacks

```bash id="2xdr0k"
helm rollback
```

---

## ✅ Environment-specific values

```text id="4mryiw"
values-dev.yaml
values-prod.yaml
```

---

# 🧪 Install Helm App

```bash id="mqqb0y"
helm install airflow apache-airflow/airflow
```

---

# 🔥 Real Production Usage

Most enterprise Kubernetes apps ship as:

> Helm Charts

Examples:

* Airflow
* Prometheus
* Grafana
* Istio
* NGINX ingress

---

# ⚠️ Helm Limitation

Helm uses:

```text id="f4v1iy"
templating
```

Complex templates can become hard to maintain.

---

# 🔹 2. Kustomize

Alternative to Helm.

Built directly into `kubectl`.

---

# 🧠 Philosophy

Helm:

```text id="v09qpt"
Template generation
```

Kustomize:

```text id="c4c7f4"
Patch & overlay existing YAML
```

---

# 🔹 Core Idea

Start with:

```text id="up7zof"
base configuration
```

Then apply:

```text id="w9s0v6"
environment overlays
```

---

# 🧩 Structure

```text id="7b2fny"
k8s/
├── base/
│    ├── deployment.yaml
│    └── service.yaml
│
├── overlays/
│    ├── dev/
│    └── prod/
```

---

# 🔹 Base Deployment

Common YAML.

---

# 🔹 Prod Overlay

Only patches differences.

Example:

* more replicas
* production image
* ingress enabled

---

# 🧪 Example kustomization.yaml

```yaml id="p71hnx"
resources:
  - ../../base

replicas:
  - name: api
    count: 5
```

---

# 🔥 Benefits

✅ Cleaner than giant templates
✅ Native kubectl support
✅ Easier patching
✅ GitOps friendly

---

# 🧠 Helm vs Kustomize

| Feature              | Helm           | Kustomize         |
| -------------------- | -------------- | ----------------- |
| Templates            | Yes            | No                |
| Package manager      | Yes            | No                |
| Environment overlays | Moderate       | Excellent         |
| Community ecosystem  | Huge           | Smaller           |
| Complexity           | Can grow large | Cleaner structure |

---

# 🔥 Production Reality

Many companies use:

```text id="0jfv4w"
Helm + Kustomize together
```

Example:

```text id="43e72s"
Helm generates manifests
Kustomize customizes environment
```

---

# 🔹 3. CI/CD with Kubernetes

This is where deployment automation starts.

---

# 🧠 What is CI/CD?

---

# CI = Continuous Integration

Automates:

* build
* test
* image creation

---

# CD = Continuous Deployment/Delivery

Automates:

* deployment
* rollout
* rollback

---

# 🧩 Production Pipeline

```text id="6iww32"
Developer Pushes Code
          ↓
GitHub/GitLab
          ↓
CI Pipeline Runs
 ├── Unit Tests
 ├── Build Docker Image
 ├── Push Image to Registry
 └── Security Scan
          ↓
CD Pipeline Deploys to Kubernetes
```

---

# 🔥 Typical Tools

| Purpose  | Tools                              |
| -------- | ---------------------------------- |
| CI       | GitHub Actions, Jenkins, GitLab CI |
| Registry | ACR, ECR, Docker Hub               |
| CD       | ArgoCD, Flux, Helm                 |

---

# 🔹 Example CI Pipeline

---

# Step 1: Build Image

```bash id="9nglg0"
docker build -t myapp:v1 .
```

---

# Step 2: Push to Registry

```bash id="6m2ymv"
docker push myapp:v1
```

---

# Step 3: Deploy to Kubernetes

```bash id="s3xkfy"
helm upgrade --install myapp ./chart
```

---

# 🔥 Production Enhancements

CI usually includes:

* vulnerability scanning
* linting
* automated tests
* image signing

---

# 🔹 4. GitOps (VERY IMPORTANT)

One of the biggest modern Kubernetes concepts.

---

# 🧠 Traditional Deployment Problem

Traditional CI/CD:

```text id="hslh9y"
Pipeline directly modifies cluster
```

Problems:
❌ configuration drift
❌ poor auditability
❌ hard rollback
❌ hidden cluster changes

---

# ✅ GitOps Philosophy

> Git is the SINGLE source of truth

---

# 🧠 Core GitOps Principle

Desired cluster state lives in Git.

---

# 🔥 Flow

```text id="hjl0nm"
Git Repository
       ↓
ArgoCD/Flux watches Git
       ↓
Detects changes
       ↓
Syncs Kubernetes cluster
```

---

# 🧩 Important Insight

Cluster becomes:

```text id="pw9d8s"
Declarative
```

---

# 🔹 Example

You update:

```yaml id="q6ylrb"
replicas: 3 → 5
```

in Git.

ArgoCD detects:

```text id="4v32d6"
Git changed
```

Then applies automatically.

---

# 🔥 Huge Benefits

---

## ✅ Full audit trail

Git history = deployment history

---

## ✅ Easy rollback

```text id="rj1m1e"
git revert
```

---

## ✅ Drift detection

If someone manually changes cluster:

```text id="yjlwm1"
ArgoCD marks OutOfSync
```

---

## ✅ Disaster recovery

Recreate cluster from Git.

---

# 🔹 ArgoCD

Most popular GitOps tool.

---

# 🧠 What ArgoCD Does

Continuously:

* monitors Git repo
* compares desired vs actual state
* synchronizes cluster

---

# 🧩 ArgoCD Architecture

```text id="vqjq1q"
Git Repo
    ↓
ArgoCD Controller
    ↓
Kubernetes Cluster
```

---

# 🔥 Features

✅ Auto-sync
✅ Rollbacks
✅ Drift detection
✅ UI dashboard
✅ Multi-cluster support

---

# 🔹 Flux

Another GitOps tool.

More lightweight and Kubernetes-native.

---

# 🧠 ArgoCD vs Flux

| Feature    | ArgoCD    | Flux            |
| ---------- | --------- | --------------- |
| UI         | Excellent | Minimal         |
| Simplicity | Easier    | More Git-native |
| Popularity | Very high | High            |

---

# 🔥 Real Production GitOps Flow

```text id="wb20j8"
Developer pushes code
        ↓
CI builds image
        ↓
Image pushed to registry
        ↓
GitOps repo updated
        ↓
ArgoCD detects change
        ↓
Cluster updated automatically
```

---

# 🧠 Important Architecture Separation

Production systems usually separate:

---

# Application Code Repo

Contains:

```text id="fhl1hv"
source code
Dockerfile
tests
```

---

# GitOps Repo

Contains:

```text id="hcvxy3"
Helm charts
Kustomize overlays
deployment configs
```

---

# 🔥 Real Enterprise Architecture

```text id="70hzfg"
Developers
    │
    ▼
Application Repo
    │
    ▼
CI Pipeline
    │
Build + Scan + Push Image
    │
    ▼
Container Registry (ACR)
    │
    ▼
Update GitOps Repo
    │
    ▼
ArgoCD / Flux
    │
    ▼
AKS Cluster
```

---

# 🔥 Common Beginner Mistakes

---

## ❌ kubectl apply manually in production

Causes drift.

---

## ❌ Storing secrets in Git

Major security risk.

---

## ❌ Huge copy-paste YAMLs

Hard to maintain.

---

## ❌ No rollback strategy

Dangerous deployments.

---

## ❌ Mixing app code + infra badly

Creates operational chaos.

---

# 🧠 Production Best Practices

---

## Packaging

✅ Helm charts

---

## Environment management

✅ Kustomize overlays

---

## CI/CD

✅ Automated pipelines

---

## Deployment

✅ GitOps

---

## Security

✅ Signed images + scanning

---

## Recovery

✅ Git as source of truth

---

# 🧠 Final Mental Model

| Component   | Purpose                   |
| ----------- | ------------------------- |
| Helm        | Package Kubernetes apps   |
| Kustomize   | Environment customization |
| CI/CD       | Automated build & deploy  |
| GitOps      | Git-driven cluster state  |
| ArgoCD/Flux | GitOps controllers        |

---

# 🔥 One-Line Summary

> Modern Kubernetes deployment is about packaging applications consistently, automating delivery pipelines, and managing cluster state declaratively through GitOps.

---

The runbook is as below:
[Runbook](runbook.md)