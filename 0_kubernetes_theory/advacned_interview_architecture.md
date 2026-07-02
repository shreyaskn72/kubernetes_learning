Here is a compilation of high-frequency Kubernetes architecture interview questions, complete with the technical deep-dives expected by senior interviewers.
------------------------------
## Question 1: What is the exact sequence of events when a user runs kubectl apply -f deployment.yaml?## Answer:
The deployment lifecycle moves through a highly orchestrated pipeline across the control plane and worker nodes:

   1. API Gateway Execution: kubectl marshals the YAML into JSON and sends a POST request to kube-apiserver.
   2. Authentication & Authorization: The API server verifies user identity (via certificates, tokens, or OIDC) and runs an RBAC check to ensure the user has creation permissions in that namespace. [1] 
   3. Admission Controllers:
   * Mutating Webhooks: Run first to inject defaults (e.g., sidecars or storage classes).
      * Validating Webhooks: Run next to block deployment if it violates security or governance policies.
   4. ETCD Persistence: The validated Deployment object is saved to etcd. The API server responds with a success status to kubectl.
   5. Deployment Controller: The kube-controller-manager catches the new object via an active watch loop. It calculates that a new ReplicaSet is needed and posts a ReplicaSet manifest back to the API server.
   6. ReplicaSet Controller: This controller watches the API server, notices the new ReplicaSet, and creates individual Pod objects with blank nodeName fields.
   7. Scheduling: The kube-scheduler filters and scores all cluster nodes for these unassigned Pods. Once it picks the best node, it updates the Pod's nodeName field via a Binding request back to the API server. [2] 
   8. Node Execution: The kubelet running on the selected node detects a Pod assigned to it. It instructs the CRI (via gRPC) to pull the image and run the container, instructs the CNI to attach a cluster IP, and binds any necessary volumes via the CSI. [3] 

------------------------------
## Question 2: Why does etcd require an odd number of nodes? What happens if a network partition splits a 5-node etcd cluster into 2 and 3 nodes?## Answer:

* The Odd Number Requirement: etcd uses the Raft consensus algorithm, which dictates that any write action or leader election must be acknowledged by a strict majority (quorum) of total nodes. The mathematical formula for quorum is:
$$\text{Quorum} = \lfloor N / 2 \rfloor + 1$$ 
* A 3-node cluster requires 2 healthy nodes. It can tolerate 1 failure.
   * A 4-node cluster requires 3 healthy nodes. It can tolerate 1 failure.
   Adding a 4th node adds network overhead and higher latency without increasing fault tolerance. Therefore, clusters stick to odd numbers (3, 5, 7) for optimal performance.
* The Network Partition Behavior:
* The 3-node side: Contains a strict majority (3 > 5/2). This partition can successfully hold elections, accept incoming data writes, and continue operating normally.
   * The 2-node side: Cannot reach a majority (2 ≤ 5/2). This partition will immediately stop accepting write requests and reject updates to prevent data drift or split-brain scenarios.

------------------------------
## Question 3: What is the difference between iptables mode and IPVS mode in kube-proxy? Why choose one over the other?## Answer:

* iptables Mode: kube-proxy configures Linux sequential firewall rules to route traffic meant for a Service ClusterIP down to individual Pod IPs.
* The Bottleneck: iptables evaluations run in O(N) linear time. If a cluster scales to thousands of services, netfilter must scan a massive sequential list of rules for every packet, leading to CPU exhaustion and high latency.
* IPVS Mode: Built specifically for high-performance load balancing within the Linux kernel. It uses hash tables instead of sequential lists.
* The Performance Benefit: IPVS routes traffic in O(1) constant time regardless of scale. It supports advanced load-balancing algorithms (e.g., least connection, shortest expected delay) making it the mandatory choice for large-scale enterprise environments.

------------------------------
## Question 4: Explain the difference between Kubelet Eviction and Pod Preemption.## Answer:
They serve entirely different resource management purposes:

* Kubelet Eviction (Proactive Node Protection): This is a local node-level response to physical resource starvation (memory, disk space). When memory crosses a hard eviction threshold, the local kubelet proactively terminates Pods on its own node to protect kernel stability and prevent a Node crash. It picks pods based on their Quality of Service (QoS) class and resource consumption relative to requests. [4] 
* Pod Preemption (Control Plane Scheduling): This is a cluster-wide control plane response to an un-schedulable high-priority Pod. When a new Pod has a high PriorityClass but no node has room for it, the kube-scheduler deliberately deletes lower-priority Pods on a node to make room for the high-priority workload to land.

------------------------------
## Question 5: How do Headless Services differ architecturally from standard ClusterIP Services, and when would you use them?## Answer:

* Standard ClusterIP: The API server assigns a single, stable virtual IP address to the service. When a client calls this IP, kube-proxy randomly intercepts and loads balances the traffic down to one of the backend Pod endpoints. The client never learns the direct IP addresses of the target pods.
* Headless Service: Created by setting clusterIP: None in the Service manifest. The control plane completely skips assigning a virtual IP, and kube-proxy ignores the service entirely. [5] 
* Architectural Mechanics: CoreDNS creates multiple direct A records (or AAAA records) pointing straight to the internal IPs of all matching backend pods. A DNS lookup returns the entire list of target IPs.
* Use Case: Mandatory for stateful distributed systems (e.g., Kafka, Elasticsearch, PostgreSQL clusters) where individual member nodes need to address each other directly for data synchronization or cluster replication rather than traversing a generic proxy load balancer.

------------------------------

[1] [https://roadmap.sh](https://roadmap.sh/questions/kubernetes)
[2] [https://intellipaat.com](https://intellipaat.com/blog/interview-question/kubernetes-interview-questions-answers/)
[3] [https://www.secondtalent.com](https://www.secondtalent.com/interview-guide/kubernetes/)
[4] [https://www.edureka.co](https://www.edureka.co/blog/interview-questions/kubernetes-interview-questions/)
[5] [https://www.secondtalent.com](https://www.secondtalent.com/interview-guide/kubernetes/)
