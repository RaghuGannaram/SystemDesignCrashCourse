# Kubernetes Infrastructure Hierarchy

## Cluster (Top Level)

-   `Role`: The entire Kubernetes infrastructure is encapsulated within a cluster. A cluster consists of multiple worker nodes and a control plane (master components) that manage the entire system.
-   `Manages`: The control plane and worker nodes.
-   `Purpose`: A Kubernetes cluster is the highest-level abstraction that defines a group of nodes (computing resources) used to run applications.

## Control Plane (Master Node)

-   `Role`: The control plane manages the cluster, orchestrating all the worker nodes, pods, and services running in the cluster.
-   `Components`:
    -   `API Server`: The entry point for all Kubernetes commands. It exposes the Kubernetes API.
    -   `Controller Manager`: Manages controllers that handle operations like deployments, replica sets, and nodes.
    -   `Scheduler`: Assigns workloads (pods) to nodes based on resource availability.
    -   `etcd`: The distributed key-value store used to store all the cluster data.
-   `Manages`: Worker nodes and cluster-wide configuration.

## Worker Plane (Worker Nodes)

-   `Role`: A node is a physical or virtual machine that runs workloads in the cluster. Each node contains components required to run pods.
-   `Components`:
    -   `kubelet`: The agent that runs on each node, communicating with the control plane to ensure the desired state of the node.
    -   `Kube-proxy`: Manages networking, ensuring that each pod has a unique IP address and that services can communicate with them.
    -   `Container Runtime`: The software (e.g., Docker, containerd) that actually runs the containers.
-   `Manages`: Pods and containers running on the node.
-   `Managed by`: The control plane.

---

## Key Kubernetes Objects Hierarchy (Inside the Cluster)

### Namespace

-   `Role`: Namespaces allow dividing a single Kubernetes cluster into multiple virtual clusters, essentially grouping resources for easier management.
-   `Manages`: Pods, services, and other resources within its scope.
-   `Purpose`: To provide isolation and resource allocation within the cluster (e.g., separate environments for dev, staging, and production).
-   `Managed by`: Cluster-wide configuration.

### Workload Objects (Running Applications)

-

## Definition of Workload:

-   `Definition`:

    -   In Kubernetes, a `workload` refers to an application running on the cluster.
    -   It represents the resources used to process and manage tasks, whether they are long-running services, batch jobs, or other forms of computing tasks.
    -   It can be thought of as an abstraction that describes an application’s deployment and behavior within the cluster.
    -   It is essentially the "work" your containers do, whether it's a web server, database, or batch job.

-   `Types`:

    -   `Deployment`:

        -   `Purpose`: Used for running stateless applications, such as web servers or APIs.
        -   `How it works`: It manages ReplicaSets and ensures that a specified number of pod replicas are running at all times.
        -   `Example`: A deployment might manage three pods running an Nginx web server, ensuring that even if one pod crashes, it is replaced.

    -   `StatefulSet`:

        -   `Purpose`: Used for running stateful applications, like databases or services that require persistent storage and stable network identities.
        -   `How it works`: It manages pods with stable, unique identities, which ensures predictable deployment and scaling.
        -   `Example`: Managing a MySQL or Cassandra database where each instance has a stable identity and needs persistent storage.

    -   `DaemonSet`:

        -   `Purpose`: Ensures that a copy of a pod runs on all or selected nodes. Often used for tasks like logging, monitoring, or other node-level functions.
        -   `How it works`: Whenever a new node is added to the cluster, the DaemonSet automatically adds the defined pod to it.
        -   `Example`: Running a log collection agent on every node to collect system logs.

    -   `Job`:

        -   `Purpose`: Used for running short-lived, batch processing tasks that need to run to completion.
        -   `How it works`: It creates one or more pods to perform the job, and once the task is complete, the pods are terminated.
        -   `Example`: Running a job to process data, such as generating reports or running batch processing scripts.

    -   `CronJob`:
        -   `Purpose`: Used to schedule jobs to run at specified times, similar to cron jobs in Linux.
        -   `How it works`: A CronJob creates jobs on a defined schedule, such as every hour or daily at a specific time.
        -   `Example`: Scheduling backups of a database every night at midnight.

-   `Managed by`: Namespaces and higher-level controllers.

### ReplicaSet

-   `Role`: Ensures the specified number of pod replicas are running. It is controlled by a higher-level controller (usually a deployment or stateful set).
-   `Managed by`: Deployments, StatefulSets, DaemonSets.

### Pod

-   `Role`: The smallest deployable unit in Kubernetes. A pod encapsulates one or more containers that share resources like network and storage.
-   `Managed by`: ReplicaSets, DaemonSets, Jobs, CronJobs.

### Container

-   `Role`: A container runs the actual application inside a pod. It is isolated from other containers but can communicate with containers in the same pod via `localhost`.
-   `Managed by`: Pods.
-   `Container Runtime`: The software that runs the containers, e.g., Docker, containerd.

---

## Networking and Service Hierarchy

### Service

-   `Role`: Exposes a set of pods to other services or external users, providing load balancing, IP address stability, and network routing.
-   `Types`:
    -   `ClusterIP`: Internal service exposing pods within the cluster.
    -   `NodePort`: Exposes a service to external traffic by mapping a port on each node.
    -   `LoadBalancer`: Integrates with cloud providers to provision external load balancers.
-   `Managed by`: Control plane and nodes.

### Ingress

-   `Role`: Manages external HTTP/S access to services within the cluster. Ingress defines routing rules to expose services to the outside world.
-   `Managed by`: Nodes and control plane.

### ConfigMap and Secret

-   `Role`:
    -   `ConfigMap`: Stores non-sensitive configuration data as key-value pairs for use by pods.
    -   `Secret`: Stores sensitive data like passwords or API keys, used by containers inside pods.
-   `Managed by`: Pods and containers.

---

## Storage Hierarchy

### PersistentVolume (PV)

-   `Role`: Represents a piece of storage in the cluster that has been provisioned by an administrator or dynamically by a storage class.
-   `Managed by`: Control plane.

### PersistentVolumeClaim (PVC)

-   `Role`: A request for storage by a pod. The PVC binds to an available PersistentVolume (PV) and allows the pod to use that storage.
-   `Managed by`: Pods and StatefulSets.

---

## Kubernetes Object Hierarchy Summary

`Cluster` : Manages: Control plane and nodes.

`Control Plane` : Manages: Nodes, pods, services, networking, and overall cluster state.

`Node` : Manages: Pods and containers via kubelet and container runtime.

`Namespace`: Manages: Resources within a logical scope.

`Deployment / StatefulSet / DaemonSet / Job / CronJob` : Manages: ReplicaSets and Pods.

`ReplicaSet` : Manages: Pods.

`Pod` : Manages: Containers.

`Container` : Runs the application inside the pod.

`Service` : Manages: Networking and external/internal communication for pods.

`Ingress` : Manages: External HTTP/S access to services.

`PersistentVolume (PV)` : Provides: Storage resources.

`PersistentVolumeClaim (PVC)` : Claims: Storage resources for pods.

---

## Visual Representation of the Complete Hierarchy:

```bash
Cluster
   └── Control Plane
         ├── Nodes
         │    ├── Pods
         │    │    └── Containers
         │    └── Kubelet, Kube-proxy, Container Runtime
         ├── Namespace
         │    └── Workload ( Deployment, StatefulSet, DaemonSet, CronJob, Job )
         │         └── ReplicaSet
         │              └── Pod
         │                   └── Container
         ├── Service
         ├── Ingress
         ├── PersistentVolume ( PV )
         └── PersistentVolumeClaim ( PVC )
```

---

### Explanation of the Hierarchy:

-   The `Cluster` is the highest abstraction, encapsulating everything.
-   The `Control Plane` manages all the worker nodes and cluster resources.
-   `Nodes` host the `pods`, which contain the actual `containers`.
-   `Namespaces` logically divide the cluster, and within each namespace, we define `workloads` like deployments and jobs.
-   `Pods` are the smallest units running containers, managed by `ReplicaSets` and other controllers.
-   Networking components like `Services` and `Ingress` manage communication between pods and the external world.
-   Storage is managed by `PersistentVolumes (PV)` and `PersistentVolumeClaims (PVC)`, providing persistent storage to pods.
