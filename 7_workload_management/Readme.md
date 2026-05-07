Kubernetes also handles **different workload patterns**, not just “run a web app.”

Up to now, we mostly used:

* Deployments
* Stateless applications

But real systems also need:

* Batch processing
* Scheduled jobs
* System agents
* Databases
* Auto-scaling

That’s exactly what this phase covers.

---

# 🧠 Big Picture

Different applications behave differently.

| Workload Type       | Kubernetes Object |
| ------------------- | ----------------- |
| Web APIs            | Deployment        |
| One-time tasks      | Job               |
| Scheduled tasks     | CronJob           |
| Agent on every node | DaemonSet         |
| Databases           | StatefulSet       |
| Auto-scaling apps   | HPA               |

---

# 🔹 1. Jobs

---

# 👉 What is a Job?

A Job runs:

> A task until completion

Unlike Deployment:

* Deployment → runs forever
* Job → completes and exits

---

# 🧠 Mental Model

```text id="7i4qgq"
Deployment → "Keep app alive forever"
Job → "Run task once and finish"
```

---

# 🧪 Real-World Examples

* Database migration
* ETL processing
* Report generation
* Backup script
* ML batch inference

---

# 🧪 Example Job YAML

```yaml id="7rq4yl"
apiVersion: batch/v1
kind: Job
metadata:
  name: database-backup
spec:
  template:
    spec:
      containers:
        - name: backup
          image: busybox
          command: ["sh", "-c", "echo Backup completed"]

      restartPolicy: Never
```

---

# 🔥 What Happens?

1. Pod starts
2. Task runs
3. Pod exits
4. Job marked completed

---

# 🔹 Important Fields

---

## restartPolicy

Usually:

```yaml id="h7y58v"
restartPolicy: Never
```

OR:

```yaml id="f7ek1u"
OnFailure
```

---

## completions

How many successful runs needed.

```yaml id="50e7vp"
completions: 3
```

---

## parallelism

Run multiple pods simultaneously.

```yaml id="hj3my7"
parallelism: 5
```

---

# 🧠 Example

```text id="v5i3rw"
Process 1000 files
5 pods process in parallel
```

---

# 🔹 2. CronJobs

---

# 👉 What is it?

A CronJob runs Jobs:

> On a schedule

Equivalent to Linux cron.

---

# 🧠 Mental Model

```text id="f54pkd"
CronJob = scheduler for Jobs
```

---

# 🧪 Real-World Examples

* Nightly backups
* Daily reports
* Cleanup tasks
* Airflow DAG trigger
* Log archival

---

# 🧪 Example YAML

```yaml id="iwdzly"
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup

spec:
  schedule: "0 2 * * *"

  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: busybox
              command:
                - sh
                - -c
                - echo Nightly backup

          restartPolicy: OnFailure
```

---

# 🔥 Schedule Format

```text id="k9m07m"
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day
│ └──────── Hour
└────────── Minute
```

---

# 🧠 Example

```text id="ot0q4c"
0 2 * * *
```

Means:

> Run daily at 2 AM

---

# 🔥 Important Production Considerations

---

## concurrencyPolicy

Prevent overlapping jobs.

```yaml id="0m5m1f"
concurrencyPolicy: Forbid
```

---

## successfulJobsHistoryLimit

Cleanup old jobs.

```yaml id="8uohic"
successfulJobsHistoryLimit: 3
```

---

# 🔹 3. DaemonSets

---

# 👉 What is a DaemonSet?

Ensures:

> One Pod runs on EVERY node

---

# 🧠 Mental Model

```text id="3qmyc6"
Deployment:
3 replicas anywhere

DaemonSet:
1 pod per node
```

---

# 🧪 Real-World Examples

System-level agents:

* Log collectors
* Monitoring agents
* Security scanners
* Node exporters
* Fluentd
* Datadog agent

---

# 🧩 Example

Cluster:

```text id="fdu6es"
Node-1
Node-2
Node-3
```

DaemonSet creates:

```text id="71u92e"
Node-1 → agent pod
Node-2 → agent pod
Node-3 → agent pod
```

---

# 🧪 Example YAML

```yaml id="r6c6ho"
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent

spec:
  selector:
    matchLabels:
      app: fluentd

  template:
    metadata:
      labels:
        app: fluentd

    spec:
      containers:
        - name: fluentd
          image: fluent/fluentd
```

---

# 🔥 Important Insight

New node added?
👉 DaemonSet automatically deploys pod there.

---

# 🔹 4. StatefulSets (VERY IMPORTANT)

This is one of the hardest Kubernetes concepts.

---

# 👉 Problem with Deployments

Deployments assume:

* Pods are interchangeable
* Stateless

Good for:

* APIs
* Frontends

BAD for:

* Databases

---

# Why?

Databases need:

* Stable identity
* Stable storage
* Ordered startup

---

# 🔹 What is StatefulSet?

StatefulSet manages:

> Stateful applications

---

# 🧠 Mental Model

```text id="oj4s4k"
Deployment Pods:
random, replaceable

StatefulSet Pods:
unique identities
```

---

# 🧩 Example

Deployment pods:

```text id="ehxlfc"
app-pod-xyz
app-pod-abc
```

Can change anytime.

---

# StatefulSet pods:

```text id="byvj8z"
db-0
db-1
db-2
```

Stable forever.

---

# 🔥 Key Features

---

## Stable Pod Names

```text id="jlmlvw"
postgres-0
postgres-1
```

Never changes.

---

## Stable Persistent Storage

Each pod gets its own PVC.

```text id="lg6v1m"
postgres-0 → pvc-0
postgres-1 → pvc-1
```

---

## Ordered Deployment

```text id="mbt0e9"
db-0 starts first
then db-1
then db-2
```

Important for clustered DBs.

---

# 🧪 Real-World Use Cases

* PostgreSQL
* MySQL
* Kafka
* Zookeeper
* Elasticsearch
* Redis clusters

---

# 🧪 Example YAML

```yaml id="ut5sdp"
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres

spec:
  serviceName: postgres

  replicas: 2

  selector:
    matchLabels:
      app: postgres

  template:
    metadata:
      labels:
        app: postgres

    spec:
      containers:
        - name: postgres
          image: postgres

          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data

  volumeClaimTemplates:
    - metadata:
        name: data

      spec:
        accessModes: ["ReadWriteOnce"]

        resources:
          requests:
            storage: 10Gi
```

---

# 🔥 Important Difference

Deployment:

```text id="h6jkt8"
Shared stateless replicas
```

StatefulSet:

```text id="th4gbi"
Unique persistent identities
```

---

# 🔹 5. Horizontal Pod Autoscaler (HPA)

Now scaling.

---

# 👉 What is HPA?

Automatically scales pods based on:

* CPU
* Memory
* Custom metrics

---

# 🧠 Mental Model

```text id="4i6b1q"
More traffic?
→ Add pods

Less traffic?
→ Remove pods
```

---

# 🧪 Example

```yaml id="nsf1d0"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment

  minReplicas: 2
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu

        target:
          type: Utilization
          averageUtilization: 70
```

---

# 🔥 What Happens?

If CPU > 70%

```text id="h5o8yq"
2 pods → 5 pods
```

Traffic reduces:

```text id="te6cm6"
5 pods → 2 pods
```

---

# 🔥 Requirements

Needs:

* Metrics Server
* Resource requests/limits

---

# 🧠 Real-World Example

Traffic spike:

```text id="p1x2vo"
Morning office hours
```

HPA scales:

```text id="qjvukw"
API pods: 3 → 15
```

Night:

```text id="mkxz54"
15 → 3
```

Cost optimization 🚀

---

# 🧩 Real Production Architecture

```text id="x6kffq"
Internet
   ↓
Ingress
   ↓
API Deployment
   ↓
HPA scales pods automatically

Background Processing:
CronJobs + Jobs

Cluster Monitoring:
DaemonSets

Database:
StatefulSet + PVC
```

---

# ⚡ Common Beginner Mistakes

---

## ❌ Using Deployment for databases

👉 Use StatefulSet

---

## ❌ Forgetting PVC in StatefulSet

👉 Data loss

---

## ❌ Running node agents via Deployment

👉 Use DaemonSet

---

## ❌ No resource limits with HPA

👉 Autoscaling inaccurate

---

# 🧠 Final Mental Model

| Object      | Purpose            |
| ----------- | ------------------ |
| Job         | One-time task      |
| CronJob     | Scheduled task     |
| DaemonSet   | One pod per node   |
| StatefulSet | Stateful workloads |
| HPA         | Automatic scaling  |

---

# 🔥 One-Line Summary

> Kubernetes workload objects exist because different applications have different lifecycle, scaling, identity, and persistence requirements.

---

Schematic explanation of all workload types:

[schematic_explanation.md](schematic_explanation.md)