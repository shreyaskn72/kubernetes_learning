This document provides basic commands to manage Pods in a Kubernetes cluster using `kubectl`.


Below is command to create a Pod in Kubernetes using a YAML configuration file named `pod.yaml`.


```bash
kubectl create -f pod.yaml
```


Below is the command to list all the Pods in the Kubernetes cluster.
```bash

kubectl get pods
```

Below is the command to get detailed information about all the Pods in the Kubernetes cluster.
```bash
kubectl get pods -o wide
```

Below is the command to delete a Pod in Kubernetes using a YAML configuration file named `pod.yaml`.
```bash
kubectl delete -f pod.yaml
```

For more information on managing Pods in Kubernetes, refer to the official documentation: [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/).