The `pod.yaml` file is a Kubernetes manifest file written in YAML format. It defines the configuration for creating a Pod in a Kubernetes cluster. Here's a breakdown of its contents:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### Explanation:

1. **`apiVersion: v1`**  
   Specifies the API version of the Kubernetes resource. `v1` is used for core resources like Pods.

2. **`kind: Pod`**  
   Indicates the type of Kubernetes resource being defined. In this case, it is a Pod.

3. **`metadata`**  
   Contains metadata about the Pod, such as its name:
   - `name: nginx` sets the name of the Pod to `nginx`.

4. **`spec`**  
   Defines the specification of the Pod, including its containers:
   - **`containers`**: A list of containers that will run inside the Pod.
     - **`name: nginx`**: The name of the container.
     - **`image: nginx:1.14.2`**: Specifies the container image to use. Here, it uses the `nginx` image with version `1.14.2`.
     - **`ports`**: Defines the ports exposed by the container.
       - **`containerPort: 80`**: Specifies that the container listens on port `80`.

### Summary:
This file creates a Pod named `nginx` with a single container running the `nginx:1.14.2` image. The container exposes port `80` for HTTP traffic.