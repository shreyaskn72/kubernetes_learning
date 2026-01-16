## Setup for running kubernetes


To run kubernetes you should have either minikube or kind or k3s or docker desktop installed on your machine. You can also use cloud providers like GKE, EKS, AKS etc to run kubernetes clusters.

## To install minkube follow the instructions in the link:


https://minikube.sigs.k8s.io/docs/start/

## To install kind follow the instructions in the link:

https://kind.sigs.k8s.io/docs/user/quick-start/

## To install k3s follow the instructions in the link:
https://k3s.io/

## To install docker desktop follow the instructions in the link:

https://www.docker.com/products/docker-desktop/

## To verify kubernetes installation run the command:

```
kubectl version --client
```

This should return the kubectl client version installed on your machine.

## To verify kubernetes cluster is running run the command:

```
kubectl cluster-info
```

This should return the kubernetes cluster information if the cluster is running.

## To verify kubectl is configured to communicate with the cluster run the command:

```
kubectl config get-contexts
```

This should return the list of contexts configured in kubectl. The current context should point to the running cluster.

Example output would be like:

```
CURRENT   NAME
*         shreyasaks
          docker-desktop
          kind-kind
          minikube        

```


## To switch context to the desired cluster run the command:

```
kubectl config use-context <context-name>
```

For example:

```
kubectl config use-context minikube
```

This will switch the context to minikube cluster.

## To verify the current context run the command:

```
kubectl config current-context
```


Example output would be like:

```
minikube
```

## For more kubectl commands and usage refer the link:

https://kubernetes.io/docs/tasks/tools/