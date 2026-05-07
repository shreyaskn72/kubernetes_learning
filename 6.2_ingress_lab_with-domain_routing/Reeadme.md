This is one of the most important Kubernetes labs because it teaches:

* Services
* DNS-style routing
* Ingress
* NGINX Ingress Controller
* Host-based routing

We’ll build a real mini architecture:

```text
app1.local  → app1-service
app2.local  → app2-service
```

using:

* Kubernetes
* NGINX Ingress
* Host-based routing

---

# 🧪 LAB: Kubernetes Ingress with Domain Routing

We’ll use:

* Minikube
* NGINX Ingress Controller

---

# 🧱 Architecture

```text
Browser
   │
   ▼
NGINX Ingress Controller
   │
   ├── app1.local → app1-service → Pods
   │
   └── app2.local → app2-service → Pods
```

---

# ✅ Step 0: Start Minikube

```bash
minikube start
```

Verify:

```bash
kubectl get nodes
```

---

# ✅ Step 1: Enable NGINX Ingress Controller

Minikube provides built-in ingress addon.

```bash
minikube addons enable ingress
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

Wait until:

```text
Running
```

---

# ✅ Step 2: Create Namespace

```bash
kubectl create namespace ingress-lab
```

---

# ✅ Step 3: Create App 1 Deployment

Create file:

```bash
vi app1.yaml
```

---

## `app1.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: ingress-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1

  template:
    metadata:
      labels:
        app: app1

    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Hello from APP1"

          ports:
            - containerPort: 5678

---
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: ingress-lab
spec:
  selector:
    app: app1

  ports:
    - port: 80
      targetPort: 5678
```

---

Apply:

```bash
kubectl apply -f app1.yaml
```

---

# ✅ Step 4: Create App 2 Deployment

Create:

```bash
vi app2.yaml
```

---

## `app2.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: ingress-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2

  template:
    metadata:
      labels:
        app: app2

    spec:
      containers:
        - name: app2
          image: hashicorp/http-echo
          args:
            - "-text=Hello from APP2"

          ports:
            - containerPort: 5678

---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
  namespace: ingress-lab
spec:
  selector:
    app: app2

  ports:
    - port: 80
      targetPort: 5678
```

---

Apply:

```bash
kubectl apply -f app2.yaml
```

---

# ✅ Step 5: Verify Services & Pods

```bash
kubectl get all -n ingress-lab
```

You should see:

* app1 pods
* app2 pods
* services

---

# ✅ Step 6: Create Ingress Resource

Create:

```bash
vi ingress.yaml
```

---

## `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ingress-lab

spec:
  ingressClassName: nginx

  rules:
    - host: app1.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80

    - host: app2.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

---

Apply:

```bash
kubectl apply -f ingress.yaml
```

---

# ✅ Step 7: Verify Ingress

```bash
kubectl get ingress -n ingress-lab
```

You’ll see an ADDRESS.

---

# ✅ Step 8: Configure Local DNS (Hosts File)

We’ll map domains locally.

---

## Get Minikube IP

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

---

# Mac/Linux

Edit:

```bash
sudo vi /etc/hosts
```

Add:

```text
192.168.49.2 app1.local
192.168.49.2 app2.local
```

---

# Windows

Edit:

```text
C:\Windows\System32\drivers\etc\hosts
```

---

# ✅ Step 9: Test in Browser

Open:

```text
http://app1.local
```

Expected:

```text
Hello from APP1
```

---

Open:

```text
http://app2.local
```

Expected:

```text
Hello from APP2
```

---

# 🔥 What Just Happened?

---

# Request Flow

```text
Browser
   ↓
app1.local
   ↓
NGINX Ingress Controller
   ↓
Ingress Rule Match
   ↓
app1-service
   ↓
app1 Pods
```

---

# 🧠 Important Concepts Learned

---

# 🔹 Service

Provides stable backend access.

---

# 🔹 Ingress

Defines routing rules.

---

# 🔹 Ingress Controller

Actually handles HTTP traffic.

---

# 🔹 Host-based Routing

Different domains → different services.

---

# 🔥 Production Equivalent

Real-world:

```text
api.company.com → backend-api
airflow.company.com → airflow-web
admin.company.com → admin-service
```

Same concept.

---

# 🧪 Step 10: Debugging Commands

---

## Describe ingress

```bash
kubectl describe ingress app-ingress -n ingress-lab
```

---

## Check ingress controller

```bash
kubectl get pods -n ingress-nginx
```

---

## Check logs

```bash
kubectl logs -n ingress-nginx <nginx-pod>
```

---

# ⚡ Bonus: Path-based Routing

You can also do:

```text
/app1 → app1-service
/app2 → app2-service
```

Instead of domains.

---

# 🔐 Bonus: HTTPS/TLS

Production ingress usually includes:

```yaml
tls:
  - hosts:
      - app1.local
```

With cert-manager + Let’s Encrypt.

---

# 🧹 Cleanup

```bash
kubectl delete namespace ingress-lab
```

---

# 🧠 Final Mental Model

```text
Ingress = routing rules
Ingress Controller = traffic engine
Service = backend endpoint
Pods = application
```

---

# 🔥 What You Learned Practically

✅ Host-based routing

✅ Ingress resources

✅ NGINX Ingress Controller

✅ Service-to-pod routing

✅ DNS mapping

✅ Production-style traffic flow


---

Optional exercises:

* Extend this into:

  * HTTPS + TLS lab
  * Real domain with cert-manager
  * Azure Application Gateway Ingress Controller (AGIC)
  * Production multi-service architecture
  * Airflow ingress setup in AKS
