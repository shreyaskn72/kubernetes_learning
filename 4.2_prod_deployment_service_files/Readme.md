Let’s build a **real-world, production-style example** — not toy YAML.
We’ll deploy a **backend API service (like what Airflow or any microservice would use)** with:

* Proper labels
* Rolling updates
* Health checks
* Resource limits
* Clean Service exposure

---

# 🧱 1. `deployment.yaml` (Production-style)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: prod
  labels:
    app: user-service
    tier: backend
spec:
  replicas: 3

  # Deployment selects Pods using this
  selector:
    matchLabels:
      app: user-service

  template:
    metadata:
      labels:
        app: user-service
        tier: backend
        version: v1
    spec:
      containers:
        - name: user-service-container
          image: myrepo/user-service:v1
          
          ports:
            - containerPort: 8080

          # Health Checks
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10

          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5

          # Resource Management
          resources:
            requests:
              cpu: "200m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          # Environment variables
          env:
            - name: ENV
              value: "production"

      restartPolicy: Always

  # Rolling update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

---

# 🧠 Explanation (Deployment)

## 🔹 metadata

```yaml
name: user-service
namespace: prod
```

* Runs in **prod namespace**
* Good practice for environment isolation

---

## 🔹 replicas

```yaml
replicas: 3
```

* Always keep **3 Pods running**

---

## 🔹 selector (CRITICAL)

```yaml
matchLabels:
  app: user-service
```

👉 Deployment manages ONLY pods with this label

---

## 🔹 template (Pod definition)

### Labels:

```yaml
labels:
  app: user-service
  version: v1
```

👉 These labels are used by:

* Deployment (to manage pods)
* Service (to route traffic)

---

## 🔹 Container

```yaml
image: myrepo/user-service:v1
```

* Versioned image → enables rolling updates

---

## 🔹 Health Checks

### Liveness Probe

```yaml
/health
```

👉 If fails → container restarted

---

### Readiness Probe

```yaml
/ready
```

👉 If fails → traffic NOT sent to pod

---

## 🔹 Resources

```yaml
requests:
limits:
```

👉 Prevents:

* One pod consuming entire node
* Unstable performance

---

## 🔹 Rolling Update Strategy

```yaml
maxUnavailable: 1
maxSurge: 1
```

👉 Meaning:

* At least 2 pods always available
* Update happens gradually

---

# 🌐 2. `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: prod
spec:
  type: ClusterIP

  selector:
    app: user-service   # 🔑 Connects to Pods

  ports:
    - name: http
      port: 80
      targetPort: 8080
```

---

# 🧠 Explanation (Service)

## 🔹 type: ClusterIP

```yaml
type: ClusterIP
```

* Internal service
* Used inside cluster (microservices communication)

---

## 🔹 selector (MOST IMPORTANT)

```yaml
selector:
  app: user-service
```

👉 Service finds Pods with:

```
app=user-service
```

---

## 🔹 ports

```yaml
port: 80
targetPort: 8080
```

👉 Flow:

```
Service:80 → Pod:8080
```

---

# 🔁 End-to-End Flow

```
Client (inside cluster)
        ↓
Service (user-service:80)
        ↓ (label selector)
Pods (3 replicas)
        ↓
Container (port 8080)
```

---

# 🔥 Real-World Behavior

---

## 🚀 Deployment

* Keeps 3 pods always running
* Replaces failed pods automatically
* Updates version without downtime

---

## 🌐 Service

* Provides stable DNS:

```
user-service.prod.svc.cluster.local
```

* Load balances across pods

---

## 🔁 Rolling Update Example

Change:

```yaml
image: myrepo/user-service:v2
```

👉 Kubernetes:

1. Starts new pods (v2)
2. Stops old pods (v1)
3. No downtime

---

# 🧪 Bonus: External Access Version

If you want external access:

```yaml
type: NodePort
```

OR (cloud):

```yaml
type: LoadBalancer
```

---

# ⚠️ Common Production Mistakes

❌ Missing readiness probe
→ Traffic goes to unhealthy pods

❌ No resource limits
→ Node crashes

❌ Wrong labels
→ Service gets 0 pods

❌ Using Pod instead of Deployment
→ No scaling, no healing

---

# 🧠 Final Mental Model

```
Deployment → manages Pods (desired state)
Service → exposes Pods (stable access)
Labels → connect everything
```

---

# The flow of how a Deployment manages Pods and how a Service routes traffic to those Pods can be visualized as follows:
![Deployment Service Pod Flow](../4.0_pods_deployment_service_summary/deployment_service_pod_flow.png)
