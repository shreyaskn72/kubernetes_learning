Let’s build a **real-world, production-style Kubernetes setup** that combines:

* ✅ ConfigMap (app config)
* ✅ Secret (sensitive data)
* ✅ PVC (persistent storage)
* ✅ Deployment (your app)
* ✅ Service (exposure)

This is very close to what you’d actually run for a backend service (or even something like Airflow components).

---

# 🧱 Full Combined YAML (Multi-resource)


user-service.yaml

```yaml
# -------------------------------
# 1. ConfigMap (Non-sensitive config)
# -------------------------------
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-config
  namespace: prod
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

---
# -------------------------------
# 2. Secret (Sensitive data)
# -------------------------------
apiVersion: v1
kind: Secret
metadata:
  name: user-service-secret
  namespace: prod
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=   # base64 for "password"

---
# -------------------------------
# 3. PersistentVolumeClaim (Storage request)
# -------------------------------
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: user-service-pvc
  namespace: prod
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
# -------------------------------
# 4. Deployment
# -------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: prod
  labels:
    app: user-service
spec:
  replicas: 3

  selector:
    matchLabels:
      app: user-service

  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service-container
          image: nginx:latest   # replace with your app image

          ports:
            - containerPort: 80

          # Inject ConfigMap values
          env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: user-service-config
                  key: APP_ENV

            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: user-service-config
                  key: LOG_LEVEL

            # Inject Secret
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: user-service-secret
                  key: DB_PASSWORD

          # Mount persistent storage
          volumeMounts:
            - name: app-storage
              mountPath: /usr/share/nginx/html/data

      volumes:
        - name: app-storage
          persistentVolumeClaim:
            claimName: user-service-pvc

---
# -------------------------------
# 5. Service
# -------------------------------
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: prod
spec:
  type: ClusterIP

  selector:
    app: user-service

  ports:
    - port: 80
      targetPort: 80
```

---

# 🧠 Deep Explanation (How Everything Connects)

---

# 🔹 1. ConfigMap → App Configuration

```yaml
APP_ENV: production
LOG_LEVEL: info
```

👉 Injected into container as environment variables:

```
APP_ENV=production
LOG_LEVEL=info
```

---

## 🔥 Why this matters

* You can change config without rebuilding image
* Different configs for dev/prod

---

# 🔐 2. Secret → Sensitive Data

```yaml
DB_PASSWORD: cGFzc3dvcmQ=
```

👉 Injected as:

```
DB_PASSWORD=password
```

---

## ⚠️ Real-world note

* Use **external secret managers** in production (Vault, AWS Secrets Manager)

---

# 💾 3. PVC → Persistent Storage

```yaml
storage: 5Gi
```

👉 Kubernetes:

* Allocates storage dynamically (if StorageClass exists)
* Attaches it to pods

---

## Mounted here:

```yaml
mountPath: /usr/share/nginx/html/data
```

👉 Data written here:

* Survives pod restart
* Survives rescheduling

---

# 🚀 4. Deployment → Runs the App

## Key parts:

### Replicas

```yaml
replicas: 3
```

👉 Always 3 pods running

---

### Selector

```yaml
matchLabels:
  app: user-service
```

👉 Controls which pods belong to this deployment

---

### Volumes + Config + Secret

This is where everything connects:

| Feature | Source    | Injected into |
| ------- | --------- | ------------- |
| Config  | ConfigMap | env vars      |
| Secret  | Secret    | env vars      |
| Storage | PVC       | filesystem    |

---

# 🌐 5. Service → Stable Access

```yaml
selector:
  app: user-service
```

👉 Routes traffic to all 3 pods

---

## Internal DNS:

```
user-service.prod.svc.cluster.local
```

---

# 🔁 Full Flow (End-to-End)

```
Client (inside cluster)
        ↓
Service (user-service)
        ↓
Pods (3 replicas)
        ↓
Container:
   - Reads config from ConfigMap
   - Reads password from Secret
   - Writes data to PVC
```

---

# 🔥 What Happens in Real Scenarios

---

## 🔄 Pod Restart

* ConfigMap + Secret → re-injected
* PVC → data remains

---

## 📈 Scaling

```bash
kubectl scale deployment user-service --replicas=5
```

* New pods automatically:

  * Get config
  * Get secrets
  * Mount same PVC (if allowed)

---

## ⚠️ Note on PVC

* `ReadWriteOnce` → only one node can mount at a time
* For multi-node write → need different storage class

---

# ❌ Common Mistakes

---

## ❌ Hardcoding config in image

👉 Use ConfigMap

---

## ❌ Putting passwords in ConfigMap

👉 Use Secret

---

## ❌ Using emptyDir for important data

👉 Use PVC

---

## ❌ Label mismatch

👉 Service won’t find pods

---

# 🧠 Final Mental Model

```
ConfigMap → configuration
Secret → sensitive config
PVC → persistent storage
Deployment → runs app
Service → exposes app
```

---

To use this follow the [run_book.md](run_book.md) instructions to apply the YAML and see everything in action!

