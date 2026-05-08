```text id="jlwmwd"
┌──────────────────────────────────────────────────────────────────────────────┐
│                 PRODUCTION GITOPS + CI/CD ON AKS                            │
└──────────────────────────────────────────────────────────────────────────────┘


                           ┌─────────────────┐
                           │   Developers    │
                           └─────────────────┘
                                     │
                     git push code / pull requests
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                         APPLICATION SOURCE REPOSITORY
═══════════════════════════════════════════════════════════════════════════════

                    ┌─────────────────────────────┐
                    │       GitHub/GitLab         │
                    │                             │
                    │  Application Source Code    │
                    │  Dockerfile                 │
                    │  Unit Tests                 │
                    └─────────────────────────────┘
                                     │
                          Webhook triggers CI
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                              CI PIPELINE
═══════════════════════════════════════════════════════════════════════════════

                  ┌────────────────────────────────┐
                  │     GitHub Actions/Jenkins     │
                  │                                │
                  │ 1. Run Unit Tests              │
                  │ 2. Run Security Scans          │
                  │ 3. Build Docker Image          │
                  │ 4. Tag Version                 │
                  │ 5. Push to Registry            │
                  └────────────────────────────────┘
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                         CONTAINER REGISTRY
═══════════════════════════════════════════════════════════════════════════════

                     ┌────────────────────────┐
                     │ Azure Container Reg.   │
                     │         (ACR)          │
                     │                        │
                     │ myapp:v1.0.0           │
                     │ myapp:v1.0.1           │
                     └────────────────────────┘
                                     │
                          Update deployment version
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                              GITOPS REPOSITORY
═══════════════════════════════════════════════════════════════════════════════

                    ┌─────────────────────────────┐
                    │    Infrastructure Repo      │
                    │                             │
                    │ Helm Charts                 │
                    │ Kustomize Overlays          │
                    │ ArgoCD Applications         │
                    │ Environment Configs         │
                    │                             │
                    │ dev/                        │
                    │ qa/                         │
                    │ prod/                       │
                    └─────────────────────────────┘
                                     │
                            Git becomes SOURCE OF TRUTH
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                             GITOPS CONTROLLER
═══════════════════════════════════════════════════════════════════════════════

                         ┌────────────────────┐
                         │       ArgoCD       │
                         │       / Flux       │
                         │                    │
                         │ Watches Git Repo   │
                         │ Detects Changes    │
                         │ Detects Drift      │
                         │ Syncs Cluster      │
                         └────────────────────┘
                                     │
                                     ▼


═══════════════════════════════════════════════════════════════════════════════
                           AKS KUBERNETES CLUSTER
═══════════════════════════════════════════════════════════════════════════════

┌───────────────────────────────────────────────────────────────────────────┐
│                                                                           │
│  Namespace: dev                                                           │
│  ─────────────────────────────────────────────                            │
│  Deployment → Pods                                                        │
│  Service                                                                  │
│  Ingress                                                                  │
│                                                                           │
│                                                                           │
│  Namespace: qa                                                            │
│  ─────────────────────────────────────────────                            │
│  Deployment → Pods                                                        │
│  Service                                                                  │
│  Ingress                                                                  │
│                                                                           │
│                                                                           │
│  Namespace: prod                                                          │
│  ─────────────────────────────────────────────                            │
│  Deployment → Pods                                                        │
│  Service                                                                  │
│  Ingress                                                                  │
│  HPA                                                                       │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
                            OBSERVABILITY & ALERTING
═══════════════════════════════════════════════════════════════════════════════

              ┌──────────────────────────────────────┐
              │ Prometheus + Grafana + Alertmanager │
              │                                      │
              │ Metrics                              │
              │ Dashboards                           │
              │ Alerts                               │
              └──────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
                              SECURITY LAYER
═══════════════════════════════════════════════════════════════════════════════

     ┌──────────────────────────────────────────────────────────────┐
     │                                                              │
     │ Azure AD Authentication                                      │
     │ Kubernetes RBAC                                              │
     │ Workload Identity                                            │
     │ Azure Key Vault                                              │
     │ Image Scanning                                               │
     │ Admission Controllers                                        │
     │ Signed Container Images                                      │
     │                                                              │
     └──────────────────────────────────────────────────────────────┘
```

# 🧠 End-to-End Flow Explanation

---

# 🔹 Step 1 — Developer Pushes Code

Developer commits:

* application code
* Dockerfile
* tests

to:

```text id="6vhl8f"
GitHub/GitLab
```

---

# 🔹 Step 2 — CI Pipeline Starts

CI system:

* GitHub Actions
* Jenkins
* GitLab CI

runs automatically.

---

# 🔹 Step 3 — CI Performs

```text id="ntsqgo"
✓ Unit tests
✓ Security scans
✓ Docker build
✓ Image tagging
✓ Push to ACR
```

---

# 🔹 Step 4 — Container Image Stored

Image pushed to:

```text id="27r6w8"
Azure Container Registry (ACR)
```

Example:

```text id="bjlwm2"
myapp:v1.0.1
```

---

# 🔹 Step 5 — GitOps Repo Updated

Pipeline updates:

* Helm values
* Kustomize overlay

Example:

```yaml id="jlwm3"
image:
  tag: v1.0.1
```

committed to:

```text id="jlwm4"
GitOps repository
```

---

# 🔹 Step 6 — ArgoCD Detects Change

ArgoCD continuously watches Git.

Detects:

```text id="jlwm5"
desired state changed
```

---

# 🔹 Step 7 — ArgoCD Syncs Cluster

ArgoCD applies:

* Deployments
* Services
* Ingress
* ConfigMaps

to AKS cluster.

---

# 🔹 Step 8 — Kubernetes Performs Rollout

Deployment controller:

* creates new pods
* rolling update
* readiness checks
* old pods terminated safely

---

# 🔹 Step 9 — Monitoring & Alerts

Prometheus:

* scrapes metrics

Grafana:

* dashboards

Alertmanager:

* sends alerts

---

# 🔥 Key Production Concepts

---

# Git = Source of Truth

Cluster state should come from Git only.

---

# Immutable Images

Never modify running containers.

Deploy new versions instead.

---

# Declarative Infrastructure

Desired state:

```yaml id="jlwm6"
replicas: 5
```

Kubernetes continuously reconciles.

---

# Drift Detection

If someone manually changes cluster:

```text id="jlwm7"
ArgoCD marks OutOfSync
```

---

# Multi-Environment Deployment

Same app deployed differently:

| Environment | Replica Count |
| ----------- | ------------- |
| dev         | 1             |
| qa          | 2             |
| prod        | 10            |

Using:

* Helm values
* Kustomize overlays

---

# 🔥 Production Benefits

| Benefit         | Why Important              |
| --------------- | -------------------------- |
| Repeatability   | Same deployment every time |
| Rollback        | Easy recovery              |
| Auditability    | Git history                |
| Security        | Controlled deployments     |
| Scalability     | Multi-cluster automation   |
| Drift Detection | Prevent config mismatch    |

---

# 🧠 Real Enterprise Stack

| Layer          | Common Tool              |
| -------------- | ------------------------ |
| Source Control | GitHub/GitLab            |
| CI             | GitHub Actions/Jenkins   |
| Registry       | Azure Container Registry |
| Packaging      | Helm                     |
| Customization  | Kustomize                |
| GitOps         | ArgoCD                   |
| Kubernetes     | AKS                      |
| Monitoring     | Prometheus + Grafana     |
| Secrets        | Azure Key Vault          |

---

# 🔥 Ultimate Mental Model

```text id="jlwm8"
Developers write code
        ↓
CI builds immutable image
        ↓
Git stores deployment intent
        ↓
GitOps syncs desired state
        ↓
Kubernetes reconciles cluster
        ↓
Observability monitors runtime
```
