To convert the `pod.yaml` to a Deployment type, you need to use the `kind: Deployment` resource. A Deployment manages Pods and ensures the desired number of replicas are running. Here's the converted `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### Explanation:
1. **`kind: Deployment`**  
   Specifies that this resource is a Deployment.

2. **`replicas: 3`**  
   Defines the desired number of Pod replicas.

3. **`selector`**  
   Specifies how the Deployment identifies the Pods it manages. The `matchLabels` must match the labels in the `template`.

4. **`template`**  
   Defines the Pod template, which includes:
   - **`spec.containers`**: The container configuration, similar to the original `pod.yaml`.

This Deployment will create and manage 3 replicas of the `nginx` Pod.

### Applying the Deployment

To apply the Deployment, save the above YAML to a file named `deployment.yaml` and run the following command:

```bash
kubectl apply -f deployment.yaml
```

### Verifying the Deployment
You can verify that the Deployment and its Pods are created successfully by running:

```bash 
kubectl get deployments
kubectl get pods
```

