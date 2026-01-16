Below are the **most commonly used Kubernetes commands related to `deployment.yaml`**, grouped by purpose. These are the commands you’ll use 90% of the time while working with Deployments.

---

## 1. Create / Apply a Deployment

### Apply deployment from YAML (recommended)

```bash
kubectl apply -f deployment.yaml
```

### Create deployment (first time only)

```bash
kubectl create -f deployment.yaml
```

> ✅ `apply` is preferred because it supports updates and is idempotent.

---

## 2. Get / View Deployment Information

### List all deployments

```bash
kubectl get deployments
```

### Get deployment in a namespace

```bash
kubectl get deployments -n <namespace>
```

### Get deployment details

```bash
kubectl get deployment <deployment-name>
```

### Describe deployment (very important for debugging)

```bash
kubectl describe deployment <deployment-name>
```

---

## 3. Check Pods Created by Deployment

### List pods

```bash
kubectl get pods
```

### Pods with wide output (node, IP)

```bash
kubectl get pods -o wide
```

### Pods using label selector

```bash
kubectl get pods -l app=<label-name>
```

---

## 4. Update Deployment

### Re-apply updated YAML

```bash
kubectl apply -f deployment.yaml
```

### Update image (without editing YAML)

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image>:<tag>
```

Example:

```bash
kubectl set image deployment/flask-app flask-app=myrepo/flask:v2
```

---

## 5. Scale Deployment

### Scale replicas

```bash
kubectl scale deployment <deployment-name> --replicas=3
```

### Check replicas

```bash
kubectl get deployment <deployment-name>
```

---

## 6. Rollout & Version Management (Very Important)

### Check rollout status

```bash
kubectl rollout status deployment <deployment-name>
```

### View rollout history

```bash
kubectl rollout history deployment <deployment-name>
```

### Rollback to previous version

```bash
kubectl rollout undo deployment <deployment-name>
```

### Rollback to specific revision

```bash
kubectl rollout undo deployment <deployment-name> --to-revision=2
```

---

## 7. Edit Deployment Live (Quick Fix)

### Edit deployment in editor

```bash
kubectl edit deployment <deployment-name>
```

> ⚠️ Changes here are **not saved in YAML file**. Use only for quick debugging.

---

## 8. Logs & Debugging

### View logs of a pod

```bash
kubectl logs <pod-name>
```

### Logs from a specific container

```bash
kubectl logs <pod-name> -c <container-name>
```

### Stream logs

```bash
kubectl logs -f <pod-name>
```

---

## 9. Delete Deployment

### Delete using YAML

```bash
kubectl delete -f deployment.yaml
```

### Delete by name

```bash
kubectl delete deployment <deployment-name>
```

---

## 10. Validate & Dry Run (Best Practice)

### Validate YAML without applying

```bash
kubectl apply -f deployment.yaml --dry-run=client
```

### Show generated output

```bash
kubectl apply -f deployment.yaml --dry-run=client -o yaml
```

---

## 11. Useful One-Liners

### Get deployment YAML from cluster

```bash
kubectl get deployment <deployment-name> -o yaml
```

### Watch deployment changes live

```bash
kubectl get deployments -w
```

---

## Typical Workflow (Real Life)

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl rollout status deployment my-app
kubectl logs <pod-name>
```

---
These commands cover the essential operations you'll perform on Kubernetes Deployments. Mastering them will significantly enhance your efficiency in managing containerized applications on Kubernetes.


