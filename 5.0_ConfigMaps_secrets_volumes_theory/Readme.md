# Kubernetes ConfigMaps, Secrets, and Volumes: The Ultimate Guide


There are two problems Kubernetes solves:

### 1️⃣ Configuration

> “How do I pass config to my app without rebuilding images?”

👉 Solution:

* ConfigMaps
* Secrets

---

### 2️⃣ Storage

> “Where does my data live if pods keep dying?”

👉 Solution:

* Volumes (different types)
* Persistent storage (PV/PVC)

---

# 🔹 1. ConfigMaps (Non-sensitive configuration)

## 👉 What is it?

A **ConfigMap** stores plain configuration data:

* Environment variables
* Config files
* App settings

---

## 🧠 Mental Model

> ConfigMap = external config file for your app

---

## 🧪 Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

---

## 🔌 How to use in Pod

### Option 1: Environment Variables

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

---

### Option 2: Mount as File

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config

volumes:
  - name: config-volume
    configMap:
      name: app-config
```

👉 Your app reads config from file

---

## ⚡ Why ConfigMaps?

* No need to rebuild Docker image
* Change config dynamically
* Environment separation (dev/prod)

---

# 🔐 2. Secrets (Sensitive Data)

## 👉 What is it?

Same as ConfigMap, but for:

* Passwords
* API keys
* Tokens

---

## ⚠️ Important Truth

> Secrets are **base64 encoded, NOT encrypted by default**

👉 Use encryption at rest in production

---

## 🧪 Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=   # base64 encoded
```

---

## 🔌 Use in Pod

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

---

## 🔥 Real Insight

| Use                  | Tool      |
| -------------------- | --------- |
| Non-sensitive config | ConfigMap |
| Sensitive data       | Secret    |

---

# 💾 3. Volumes (Core Concept)

## 👉 Why volumes?

Pods are **ephemeral**
👉 If pod dies → data is lost

---

## 🧠 Mental Model

> Volume = storage attached to a Pod

---

# 🔸 A. emptyDir

## 👉 What is it?

* Temporary storage
* Lives as long as Pod exists

---

## 🧪 Example

```yaml
volumes:
  - name: temp-storage
    emptyDir: {}
```

---

## 🧠 Behavior

* Pod starts → volume created
* Pod dies → volume deleted

---

## ✅ Use cases

* Caching
* Temporary files
* Sharing data between containers in same pod

---

# 🔸 B. hostPath

## 👉 What is it?

* Uses **node’s local filesystem**

---

## 🧪 Example

```yaml
volumes:
  - name: host-storage
    hostPath:
      path: /data
```

---

## ⚠️ Problems

* Tied to specific node
* Not portable
* Risky in production

---

## ✅ Use cases

* Local development
* Debugging
* Special system-level workloads

---

# 🔥 Important Insight

| Volume Type | Lifetime      | Use          |
| ----------- | ------------- | ------------ |
| emptyDir    | Pod lifetime  | Temp storage |
| hostPath    | Node lifetime | Dev/debug    |

---

# 🏗️ 4. Persistent Storage (REAL WORLD)

Now the important part 👇

---

# 🔹 Persistent Volume (PV)

## 👉 What is it?

A **real storage resource** in cluster:

* Disk
* Cloud storage (EBS, Azure Disk, etc.)

---

## 🧠 Mental Model

> PV = actual storage (created by admin/cloud)

---

## 🧪 Example

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
```

---

# 🔹 Persistent Volume Claim (PVC)

## 👉 What is it?

A **request for storage**

---

## 🧠 Mental Model

> PVC = “I need 5GB storage”

---

## 🧪 Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

---

# 🔁 PV ↔ PVC Binding

```
PVC (request) → binds → PV (actual storage)
```

---

# 🔌 Use PVC in Pod

```yaml
volumes:
  - name: app-storage
    persistentVolumeClaim:
      claimName: pvc-storage
```

---

# 🔥 Real Flow

```
App → PVC → PV → Actual Disk
```

---

# 🧠 Why this design?

Separates:

* Developer (asks for storage)
* Admin/Cloud (provides storage)

---

# ⚡ Dynamic Provisioning (Modern Way)

👉 You don’t manually create PV

Instead:

* PVC triggers automatic storage creation

---

# 🧠 Real-World Example

For your Airflow use case:

| Component       | Storage  |
| --------------- | -------- |
| Logs            | PVC      |
| Metadata DB     | PVC      |
| Temporary files | emptyDir |

---

# 🔥 Common Mistakes

❌ Using emptyDir for databases
👉 Data loss

❌ Using hostPath in production
👉 Not scalable

❌ Hardcoding config in image
👉 Bad practice

---

# 🧠 Final Summary

| Concept   | Purpose              |
| --------- | -------------------- |
| ConfigMap | Non-sensitive config |
| Secret    | Sensitive config     |
| emptyDir  | Temp storage         |
| hostPath  | Node-based storage   |
| PV        | Actual storage       |
| PVC       | Storage request      |

---

# 🧩 One-Line Mental Model

> ConfigMaps/Secrets = “what your app needs to run”
> Volumes/PV/PVC = “where your app stores data”

---

If you want next, I can:

* Give a **full real-world YAML combining Deployment + ConfigMap + Secret + PVC**
* Or show **Airflow with persistent logs & DB using PVC**
* Or simulate **what happens when pod restarts with/without PVC (very important)**
