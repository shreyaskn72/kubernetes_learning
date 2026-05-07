

# 🧪 🧱 Lab: Deploy App with ConfigMap + Secret + PVC

---

# ✅ Step 0: Pre-checks

Make sure cluster is running:

```bash
kubectl get nodes
```

👉 You should see at least one node in `Ready` state

---

# ✅ Step 1: Create Namespace

```bash
kubectl create namespace prod
```

Verify:

```bash
kubectl get ns
```

---

# ✅ Step 2: Save YAML File

Create a file:

```bash
vi user-service.yaml
```

Paste the full YAML (the one I gave earlier).

---

# ✅ Step 3: Apply Everything

```bash
kubectl apply -f user-service.yaml
```

---

# ✅ Step 4: Verify Resources

---

## 🔹 Check ConfigMap

```bash
kubectl get configmap -n prod
```

---

## 🔹 Check Secret

```bash
kubectl get secret -n prod
```

---

## 🔹 Check PVC

```bash
kubectl get pvc -n prod
```

👉 Expected:

```
STATUS: Bound
```

---

## 🔹 Check Pods

```bash
kubectl get pods -n prod
```

👉 Wait until:

```
Running
```

---

## 🔹 Check Deployment

```bash
kubectl get deployment -n prod
```

---

## 🔹 Check Service

```bash
kubectl get svc -n prod
```

---

# 🧪 Step 5: Test Inside Cluster

Run a temporary test pod:

```bash
kubectl run test-client -n prod --rm -it --image=busybox -- /bin/sh
```

Inside pod:

```sh
wget -qO- user-service
```

👉 You should see nginx response (HTML output)

---

# 🔐 Step 6: Verify ConfigMap + Secret Inside Pod

---

## Get a pod name:

```bash
kubectl get pods -n prod
```

---

## Exec into pod:

```bash
kubectl exec -it <pod-name> -n prod -- /bin/sh
```

---

## Check environment variables:

```sh
echo $APP_ENV
echo $LOG_LEVEL
echo $DB_PASSWORD
```

👉 Expected:

```
production
info
password
```

---

# 💾 Step 7: Test Persistent Storage

Inside pod:

```sh
cd /usr/share/nginx/html/data
echo "hello kubernetes" > test.txt
cat test.txt
```

---

## Now delete the pod:

Exit pod, then:

```bash
kubectl delete pod <pod-name> -n prod
```

---

## Wait for new pod:

```bash
kubectl get pods -n prod
```

---

## Exec into new pod:

```bash
kubectl exec -it <new-pod-name> -n prod -- /bin/sh
```

---

## Check file again:

```sh
cat /usr/share/nginx/html/data/test.txt
```

👉 ✅ File should still exist → PVC working

---

# 🌐 Step 8: Access from Host (minikube)

---

## Get service:

```bash
kubectl get svc -n prod
```

---

## Run:

```bash
minikube service user-service -n prod
```

👉 Opens browser automatically

---

# 🔁 Step 9: Scale the App

```bash
kubectl scale deployment user-service --replicas=5 -n prod
```

Check:

```bash
kubectl get pods -n prod
```

---

# 🔄 Step 10: Rolling Update

Edit deployment:

```bash
kubectl edit deployment user-service -n prod
```

Change:

```
image: nginx:latest → nginx:1.25
```

---

Watch rollout:

```bash
kubectl rollout status deployment user-service -n prod
```

---

# 🔥 Step 11: Break Something (Important Learning)

---

## Delete a pod:

```bash
kubectl delete pod <pod-name> -n prod
```

👉 Kubernetes will recreate it automatically

---

# 🧹 Step 12: Cleanup

```bash
kubectl delete namespace prod
```

---

# 🧠 What You Just Learned (Important)

You practically verified:

---

## ✅ ConfigMap

* Injected config into container

---

## ✅ Secret

* Injected sensitive data

---

## ✅ PVC

* Data survives pod restart

---

## ✅ Deployment

* Self-healing
* Scaling
* Rolling updates

---

## ✅ Service

* Stable access to pods

---

# ⚡ Bonus Debug Commands (Very Useful)

```bash
kubectl describe pod <pod-name> -n prod
kubectl logs <pod-name> -n prod
kubectl get all -n prod
```

---

# 🧠 Final Mental Model After Lab

```
ConfigMap + Secret → configure app
PVC → persist data
Deployment → manage pods
Service → expose pods
```