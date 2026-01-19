## Limitations of docker in production and why we need kubernetes ?

While Docker is a powerful tool for containerization, it has several limitations when it comes to production environments:
1. **Single Host Limitation**: Docker is primarily designed to run containers on a single host. In production, applications often need to be distributed across multiple hosts for scalability and high availability.
2. **No auto healing**: If a container crashes, Docker does not automatically restart it or replace it with a new instance.
2. **No auto scaling**: Docker does not have built-in capabilities to automatically scale containers up or down based on demand.
2. **Networking Complexity**: Docker's networking model can become complex in production environments, especially when dealing with multiple containers that need to communicate across different hosts.
3. **Storage Management**: Managing persistent storage for Docker containers can be challenging in production, particularly when containers need to share data or maintain state across restarts.
4. **Orchestration**: Docker lacks built-in orchestration capabilities, making it difficult to manage large-scale deployments, handle load balancing, and ensure fault tolerance without additional tools.
7. No api agateway: Docker does not provide a built-in API gateway to manage and route traffic to different services.
8. **Security Concerns**: Docker containers share the host OS kernel, which can lead to security vulnerabilities if not properly managed. In production, isolating workloads is crucial for security.
9. **No firewalling**: Docker does not provide built-in firewall capabilities to control traffic between containers or between containers and the outside world.
10. **No service discovery**: Docker does not have built-in service discovery mechanisms to help containers find and communicate with each other.


To address these limitations, production environments often use container orchestration platforms like Kubernetes, which provide features such as multi-host networking, storage management, auto-scaling, service discovery, and enhanced security measures.


## Kubernetes architecture


The schematic of Kubernetes architecture is as follows:

```
                +---------------------+
                |      Master Node    |
                |---------------------|
                |  API Server         |
                |  Scheduler          |
                |  etcd               |
                |  Controller Manager |
                |Cloud Controller Mgr |
                +---------------------+
                          |
        ---------------------------------------
        |                                     | 
+---------------------+           +---------------------+
|     Worker Node 1   |           |     Worker Node 2   |
|---------------------|           |---------------------|
|  Kubelet            |           |  Kubelet            |
|  Container Runtime  |           |  Container Runtime  |
|  Kube-Proxy         |           |  Kube-Proxy         |
|  Pods               |           |  Pods               |
+---------------------+           +---------------------+
```


Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications. Its architecture consists of several key components:
1. **Master Node**: The master node is the control plane of the Kubernetes cluster. It manages the cluster and coordinates all activities, including scheduling, scaling, and updates. Key components of the master node include:
   - **API Server**: The API server is the front-end for the Kubernetes control plane. It exposes the Kubernetes API and serves as the main entry point for all administrative tasks.
   - **Scheduler**: The scheduler is responsible for assigning workloads (pods) to nodes based on resource availability and other constraints.
   - **etcd**: etcd is a distributed key-value store that stores all cluster data, including configuration data, state information, and metadata.
   - **Controller Manager**: The controller manager is responsible for maintaining the desired state of the cluster by running various controllers that monitor the state of resources and make necessary adjustments.
   - **Cloud Controller Manager**: This component allows Kubernetes to interact with the underlying cloud provider's API to manage resources like load balancers, storage, and networking.

2. **Worker Nodes**: Worker nodes are the machines that run the containerized applications. Each worker node contains several key components:
    - **Kubelet**: The kubelet is an agent that runs on each worker node and communicates with the master node. It ensures that the containers are running in the desired state as specified by the master.
    - **Container Runtime**: The container runtime is responsible for running the containers on the worker node. Kubernetes supports various container runtimes, including Docker, containerd, and CRI-O.
    - **Kube-Proxy**: Kube-proxy is responsible for maintaining network rules on the worker nodes, allowing communication between pods and services within the cluster.
    - **Pods**: Pods are the smallest deployable units in Kubernetes. A pod can contain one or more containers that share the same network namespace and storage.


