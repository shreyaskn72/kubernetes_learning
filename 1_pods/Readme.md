# Managing Pods in Kubernetes

This document provides essential commands to manage Pods in a Kubernetes cluster using `kubectl`.

## Creating a Pod

To create a Pod in Kubernetes using a YAML configuration file (`pod.yaml`), use the following command:

```bash
kubectl create -f pod.yaml
```

## Listing Pods

To list all the Pods in the Kubernetes cluster, use:
```bash

kubectl get pods
```


## Viewing Detailed Pod Information
To get detailed information about all the Pods in the cluster, run:
```bash
kubectl get pods -o wide
```

## Describing a specific Pod
To describe a specific Pod, replace <pod-name> with the name of the Pod and execute:
```bash
kubectl describe pod <pod-name>
```

## Viewing Pod Logs
To view the logs of a specific Pod, replace <pod-name> with the name of the Pod and run:
```bash
kubectl logs <pod-name>
```

## Deleting a Pod
To delete a Pod using the YAML configuration file (`pod.yaml`), use the following command:
```bash
kubectl delete -f pod.yaml
```

For more information on managing Pods in Kubernetes, refer to the official documentation: [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/).