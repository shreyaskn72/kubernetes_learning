```text
┌─────────────────────────────────────────────────────────────────────┐
│                    PRODUCTION KUBERNETES CLUSTER                   │
└─────────────────────────────────────────────────────────────────────┘


                               INTERNET
                                   │
                                   ▼
                         Ingress / Load Balancer
                                   │
                                   ▼
                     ┌────────────────────────┐
                     │  API Deployment        │
                     │  (Stateless App)       │
                     └────────────────────────┘
                                   │
                      HPA monitors CPU/Memory
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼
      ┌────────────┐       ┌────────────┐       ┌────────────┐
      │ API Pod-1  │       │ API Pod-2  │       │ API Pod-3  │
      └────────────┘       └────────────┘       └────────────┘


=====================================================================


                    SCHEDULED / BATCH WORKLOADS

      ┌────────────────────────────────────────────────────┐
      │                    CronJob                         │
      │      (Runs every day at 2 AM for backups)          │
      └────────────────────────────────────────────────────┘
                                   │
                                   ▼
                         Creates Kubernetes Job
                                   │
                                   ▼
                         ┌─────────────────┐
                         │   Backup Job    │
                         └─────────────────┘
                                   │
                                   ▼
                         ┌─────────────────┐
                         │ Backup Pod Run  │
                         │ Task Completes  │
                         └─────────────────┘


=====================================================================


                    CLUSTER-WIDE SYSTEM AGENTS

         DaemonSet ensures ONE pod runs on EVERY node


        ┌─────────────────┐      ┌─────────────────┐
        │     Node-1      │      │     Node-2      │
        │                 │      │                 │
        │ Fluentd Agent   │      │ Fluentd Agent   │
        │ Monitoring Pod  │      │ Monitoring Pod  │
        └─────────────────┘      └─────────────────┘


=====================================================================


                    STATEFUL APPLICATIONS

                 StatefulSet (Database Cluster)

      ┌──────────────────────────────────────────────┐
      │              PostgreSQL StatefulSet          │
      └──────────────────────────────────────────────┘

                    Ordered & Stable Pods
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼

      ┌────────────┐       ┌────────────┐       ┌────────────┐
      │ postgres-0 │       │ postgres-1 │       │ postgres-2 │
      └────────────┘       └────────────┘       └────────────┘
             │                     │                     │
             ▼                     ▼                     ▼
      ┌────────────┐       ┌────────────┐       ┌────────────┐
      │   PVC-0    │       │   PVC-1    │       │   PVC-2    │
      │ Persistent │       │ Persistent │       │ Persistent │
      │   Volume   │       │   Volume   │       │   Volume   │
      └────────────┘       └────────────┘       └────────────┘


=====================================================================


                    HORIZONTAL POD AUTOSCALER (HPA)

                      Metrics Server monitors:
                           - CPU Usage
                           - Memory Usage
                           - Custom Metrics

                                   │
                                   ▼

                  High Traffic → Scale Out Automatically

                      2 Pods  ─────►  10 Pods


                  Low Traffic → Scale In Automatically

                     10 Pods  ─────►  2 Pods
```

# 🧠 Architecture Understanding

---

# 🔹 Deployment + HPA

Used for:

* APIs
* Web apps
* Frontends

Characteristics:

* Stateless
* Scalable
* Self-healing

---

# 🔹 CronJob + Job

Used for:

* Backups
* Reports
* ETL pipelines
* Cleanup tasks

Characteristics:

* Time-based execution
* Runs to completion

---

# 🔹 DaemonSet

Used for:

* Monitoring agents
* Log collectors
* Security scanners

Characteristics:

* One pod per node
* Automatic on new nodes

---

# 🔹 StatefulSet

Used for:

* Databases
* Kafka
* Elasticsearch
* Redis clusters

Characteristics:

* Stable identity
* Stable storage
* Ordered startup/shutdown

---

# 🔥 Real Production Mapping

| Real System         | Kubernetes Workload |
| ------------------- | ------------------- |
| Backend API         | Deployment + HPA    |
| Nightly DB Backup   | CronJob             |
| Datadog Agent       | DaemonSet           |
| PostgreSQL Cluster  | StatefulSet         |
| Airflow Scheduler   | Deployment          |
| Airflow Workers     | Deployment/HPA      |
| Airflow Metadata DB | StatefulSet         |

---

# 🧠 Ultimate Mental Model

```text
Deployment  → scalable stateless apps
Job         → one-time tasks
CronJob     → scheduled tasks
DaemonSet   → node-level agents
StatefulSet → stateful systems
HPA         → automatic scaling
```
