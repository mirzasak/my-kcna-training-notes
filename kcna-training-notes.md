# KCNA (Kubernetes and Cloud Native Associate) Study Notes

## Table of Contents
- [Cloud Native Fundamentals](#cloud-native-fundamentals)
- [Containers](#containers)
- [Kubernetes Overview](#kubernetes-overview)
- [Kubernetes Architecture](#kubernetes-architecture)
- [Kubernetes Resources](#kubernetes-resources)
- [Networking](#networking)
- [Storage](#storage)
- [Cloud Native Architecture](#cloud-native-architecture)
- [Autoscaling](#autoscaling)
- [Observability](#observability)
- [GitOps](#gitops)
- [CI/CD](#cicd)
- [Tips and Best Practices](#tips-and-best-practices)

---

## Cloud Native Fundamentals

**Cloud Native** leverages the cloud to achieve our goals with applications through:

- **Resiliency**: Using the cloud to protect against downtime or loss of service in the event of failures. This includes designing applications that can recover from failures automatically, using redundancy and failover mechanisms.
- **Manageability**: Using the cloud to ensure that applications and infrastructure are easy to manage. This involves automated deployment, scaling, and configuration management.
- **Observability**: Using the cloud to provide visibility into the status and performance of applications and infrastructure. This includes monitoring, logging, and tracing capabilities.

Cloud native design means building applications and processes from the ground up to take full advantage of cloud technologies, achieving manageability, observability, and leveraging everything the cloud offers. This approach enables organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds.

---

## Containers

### What is a Container?
A **container** is a self-contained package containing code and all of its dependencies. It can run with specialized isolation from other processes on a host. Containers provide a consistent environment for applications to run, regardless of where they are deployed. They are lightweight, portable, and can start up quickly compared to traditional virtual machines.

### Dockerfile
A **Dockerfile** is a text file containing the instructions/commands used to build a container image. It defines the base image, dependencies, environment variables, and commands needed to create a reproducible container image. Each instruction in a Dockerfile creates a new layer in the image.

### Containers vs Pods

**Containers:**
- Single process or application
- Isolated from other processes on the host using namespaces and control groups
- Packages code with all the dependencies it needs
- Provides consistency across different environments

**Pods:**
- Kubernetes object that wraps one or more containers
- Defines and tracks state of containers
- Can have one or more containers per pod that share resources
- Containers in a pod share the same network namespace and can communicate via localhost
- Represents the smallest deployable unit in Kubernetes

---

## Kubernetes Overview

**Kubernetes (k8s)** is a container orchestration tool and open-source system for automating deployment, scaling, and management of containerized applications. Originally developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF).

- k8s is a cloud-native platform for containerized applications
- Enables you to manage containerized applications with cloud native features
- Provides declarative configuration and automation
- Works across various environments (on-premises, cloud, hybrid)

### Key Tools for k8s Cluster Setup

- **kubeadm**: Tool that simplifies the process of setting up a k8s cluster. It handles the bootstrapping of control plane components and helps join worker nodes to the cluster.
- **containerd**: Container runtime used for the k8s cluster. It manages the complete container lifecycle including image transfer, container execution, and storage.
- **kubelet**: Kubernetes agent running on each node. Primary node agent that ensures containers are running in a pod.
- **kubectl**: Command-line tool for interacting with the cluster. Used to deploy applications, inspect cluster resources, and view logs.
- **calico**: Network plugin for Kubernetes that provides networking and network policy functionality. Implements the Container Network Interface (CNI).

---

## Kubernetes Architecture

A Kubernetes cluster consists of worker nodes and a control plane. This distributed architecture allows for high availability, scalability, and resilience.

### Worker Nodes
Worker nodes are the machines (physical or virtual) that host pods and run container workloads. They provide the runtime environment for your applications. Each worker node contains:
- **Kubelet**: Ensures containers are running and healthy
- **Container Runtime**: Runs the containers (e.g., containerd, CRI-O)
- **Kube-proxy**: Maintains network rules for pod communication

Worker nodes are managed by the control plane and can be added or removed dynamically. They report their status and resource availability to the control plane, which uses this information for scheduling decisions.

### Control Plane
The control plane is a set of components that manage worker nodes and pods. It makes global decisions about the cluster (scheduling, responding to events) and maintains the desired state of the cluster.

#### Control Plane Components

- **API Server**: The center of the control plane and the front-end for the Kubernetes control plane. It exposes the Kubernetes API, which is used by all other components and external users. All operations and communications between components go through the API server. It validates and processes REST requests, then updates the corresponding objects in etcd.

- **etcd**: Consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data. It reliably stores the complete state of the cluster including configuration data, state data, and metadata. The distributed nature of etcd provides fault tolerance - if one etcd node fails, others can continue serving requests.

- **Scheduler**: Watches for newly created pods that have not yet been assigned to a worker node. It selects an appropriate node to run each new pod based on factors like:
  - Resource requirements (CPU, memory)
  - Hardware/software/policy constraints
  - Affinity and anti-affinity specifications
  - Data locality
  - Workload interference
  
  The scheduler doesn't actually start pods; it just assigns them to nodes. The kubelet on each node is responsible for starting the pods.

- **Controller Manager**: Combines multiple controllers into a single process. Controllers are control loops that watch the state of your cluster through the API server and make changes to move the current state toward the desired state. Examples include:
  - Node Controller: Monitors node health
  - Replication Controller: Maintains the correct number of pods
  - Endpoints Controller: Populates endpoint objects
  - Service Account Controller: Creates default service accounts

- **Kube-proxy**: Network proxy that runs on each node, maintaining network rules that allow network communication to pods from inside or outside the cluster. It implements part of the Kubernetes Service concept by maintaining network rules and performing connection forwarding. It can use the operating system packet filtering layer or forward traffic itself.

- **Kubelet**: The primary node agent running on each node. It ensures that containers described in Pod specifications are running and healthy. The kubelet:
  - Takes PodSpecs (pod definitions) from the API server
  - Works with the container runtime to manage containers
  - Reports the status of the node and pods back to the API server
  - Mounts volumes as required by the pod
  - Downloads pod secrets from the API server

- **Container Runtime**: Software responsible for running containers. While not part of Kubernetes itself, it's essential for Kubernetes to function. Common container runtimes include:
  - **containerd**: Industry-standard container runtime, graduated CNCF project
  - **CRI-O**: Lightweight alternative to containerd, specifically designed for Kubernetes
  - **Docker**: Was commonly used but Kubernetes deprecated direct Docker support in favor of CRI-compatible runtimes

---

## Kubernetes Resources

### Pods
The most basic and smallest deployable unit in Kubernetes. A pod represents a group of one or more containers with shared storage and network resources. Pods are designed to be ephemeral - they can be created, destroyed, and recreated as needed. Each pod gets its own IP address. If you want to run containers in Kubernetes, you create a pod that defines those containers.

### ReplicaSet
Ensures a specified number of replica pods are running at any given time. It continuously monitors the running pods and creates or deletes pods to match the desired count. ReplicaSets use a pod template to create new pods when needed. They provide self-healing capabilities - if a pod fails, the ReplicaSet automatically creates a replacement. However, ReplicaSets are typically not used directly; instead, Deployments are used which manage ReplicaSets automatically.

### Deployment
Provides declarative updates for pods and ReplicaSets. Deployments are ideal for stateless applications and offer several advantages:
- **Rolling Updates**: Updates pods gradually without downtime
- **Rollback**: Can revert to previous versions if issues occur
- **Scaling**: Easily scale the number of replicas up or down
- **Versioning**: Maintains revision history

The default RollingUpdate strategy ensures zero-downtime deployments by gradually replacing old pods with new ones.

### StatefulSet
Similar to Deployment but designed for stateful applications that require:
- **Stable, unique network identifiers**: Each pod gets a persistent hostname
- **Stable, persistent storage**: Pods are associated with PersistentVolumeClaims
- **Ordered deployment and scaling**: Pods are created sequentially
- **Ordered, automated rolling updates**: Updates happen in reverse ordinal order

Pods in a StatefulSet maintain a sticky identity even if they are rescheduled to different nodes. This is crucial for databases and other stateful applications.

### DaemonSet
Ensures that a copy of a pod runs on all (or some) nodes in the cluster. As nodes are added to the cluster, pods are automatically added to them. As nodes are removed, those pods are garbage collected. DaemonSets are typically used for:
- Cluster storage daemons
- Log collection daemons
- Node monitoring daemons

You can use node selectors or taints and tolerations to run DaemonSet pods on specific nodes only.

### Job
Creates one or more pods and ensures that a specified number of them successfully complete. Jobs track successful completions and retry failed pods until the specified number succeed. They're useful for batch processing, data processing tasks, or any work that needs to run to completion. Once a Job completes, the pods remain for log inspection but are not restarted.

### CronJob
Runs Jobs on a time-based schedule, similar to Unix cron jobs. CronJobs are useful for periodic tasks like:
- Backups
- Report generation
- Sending emails
- Cleanup tasks

The schedule is defined using cron syntax (e.g., "0 2 * * *" for daily at 2 AM).

### Static Pods
Static pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. The kubelet watches each static pod and restarts it if it crashes. Static pods are typically used for control plane components on each master node. The kubelet automatically creates mirror pods in the Kubernetes API server, allowing you to see them when you run `kubectl get pods`, but you cannot manage them through the API server.

### Liveness Probes
Let you customize how kubelet determines whether a container is healthy and running. If a liveness probe fails, kubelet kills the container and the container is subjected to its restart policy. Liveness probes help detect and recover from situations where an application is running but not functioning correctly (e.g., deadlock). Common probe types include:
- **HTTP probes**: Performs an HTTP GET request
- **TCP probes**: Attempts to open a TCP connection
- **Exec probes**: Executes a command inside the container

---

## Networking

### Services
Provide a stable way to expose an application running on a set of replica pods as a network service. Since pods are ephemeral and can be replaced at any time with new IP addresses, Services provide:
- **Stable endpoint**: A single, stable IP address or DNS name
- **Load balancing**: Traffic is distributed across all healthy pod replicas
- **Service discovery**: Pods can find services via DNS or environment variables

Services use selectors to determine which pods they route traffic to. The Service monitors matching pods and automatically updates its endpoints as pods are created or destroyed.

#### Service Types

- **ClusterIP** (default): Exposes the service on an internal IP address accessible only within the cluster. This is useful for internal microservice communication. The service is reachable via `<service-name>.<namespace>.svc.cluster.local`.

- **NodePort**: Exposes the service on each node's IP at a static port (30000-32767 by default). External traffic can reach the service by accessing `<NodeIP>:<NodePort>`. This automatically creates a ClusterIP service, which the NodePort service routes to.

- **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer. This creates a NodePort and ClusterIP service automatically, which the external load balancer routes to. The cloud provider provisions an external load balancer with a public IP address.

- **ExternalName**: Maps a service to a DNS name by returning a CNAME record with its value. This makes it easy for pods to access external services using a consistent internal name. No proxying or load balancing is performed.

### Headless Services
Services with `clusterIP: None` specified, resulting in no cluster IP address being allocated. Instead of load balancing, DNS returns the IP addresses of all matching pods, allowing clients to connect directly to pods. This is useful for:
- Stateful applications that need direct pod communication
- Service discovery without kube-proxy load balancing
- Custom load balancing implementations

### Ingress
An API object that manages external HTTP(S) access to services in a cluster. Ingress provides:
- **URL-based routing**: Route traffic based on the request URL path
- **Virtual hosting**: Route traffic based on the hostname
- **SSL/TLS termination**: Handle HTTPS certificates
- **Load balancing**: Distribute traffic across services

Ingress requires an Ingress Controller (like NGINX, Traefik, or HAProxy) to be installed in the cluster to actually handle the routing.
**It's gonna be retired in *March 2026.***

### Service Mesh
A dedicated infrastructure layer for managing service-to-service communication. It provides advanced networking features without requiring changes to application code:
- **Traffic management**: Advanced routing, retries, failover
- **Security**: Mutual TLS, authorization policies
- **Observability**: Metrics, logs, traces for all service communication
- **Resilience**: Circuit breakers, timeouts, rate limiting

Popular service meshes include Istio, Linkerd, and Consul.

**Two main components:**
1. **Service Proxy / Data Plane**: Lightweight proxy containers (sidecars) deployed alongside each application container. All network traffic flows through these proxies, enabling the service mesh to observe and control communication.

2. **Control Plane**: Centralized component that manages and configures the data plane proxies. It provides the API for configuration and collects telemetry data from all proxies.

### Cluster Network
- Kubernetes uses a virtual network to allow pods to communicate within the cluster
- Each pod receives its own unique IP address within the cluster's network
- Pods can communicate directly with each other without NAT, even across different nodes
- This is achieved through a Container Network Interface (CNI) plugin like Calico, Flannel, or Weave

The Kubernetes network model requires:
- All pods can communicate with all other pods without NAT
- All nodes can communicate with all pods without NAT
- The IP a pod sees itself as is the same IP others see it as

### Network Policy
Network policies are specifications of how groups of pods are allowed to communicate with each other and other network endpoints. They function like firewall rules for pods:

- Each network policy defines which pods it applies to using label selectors
- The policy provides rules that define what incoming (ingress) and/or outgoing (egress) traffic is allowed
- By default, pods are non-isolated and accept traffic from any source
- Once a network policy selects a pod, the pod becomes isolated and only traffic explicitly allowed by network policies is permitted
- Ingress and egress rules are defined independently
- Network policies are additive - multiple policies can apply to the same pod

Example use cases:
- Restricting database pods to only accept traffic from application pods
- Preventing pods in one namespace from accessing pods in another namespace
- Blocking egress traffic to external networks

---

## Storage

### Volumes
Provide external storage to containers, solving the problem of ephemeral container storage. When a container restarts, data stored in the container filesystem is lost. Volumes persist data beyond the container lifecycle and can be shared between containers in a pod. Kubernetes supports many volume types:
- **emptyDir**: Temporary directory that exists for the lifetime of the pod
- **hostPath**: Mounts a file or directory from the host node's filesystem
- **configMap/secret**: Special volumes for configuration data
- **Cloud storage**: AWS EBS, Azure Disk, GCE PD, etc.

### Persistent Volumes (PV)
A PersistentVolume is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs are cluster resources that exist independently of any pod. They abstract storage details from users and provide a way to "claim" storage resources.

### PersistentVolumeClaim (PVC)
A PersistentVolumeClaim is a request for storage by a user. PVCs consume PV resources similar to how pods consume node resources. A PVC specifies:
- Access mode (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
- Storage size
- Storage class (optional)

When a PVC is created, Kubernetes finds an available PV that matches the requirements and binds them together. The pod then mounts the PVC as a volume.

**Dynamic Provisioning**: Storage Classes enable dynamic provisioning - when a PVC is created, the storage is automatically provisioned from the cloud provider without manual intervention.

### Reclaim Policies
Define what happens to a PersistentVolume when its PersistentVolumeClaim is deleted:

- **Retain**: The PV is not deleted. Storage asset remains and must be reclaimed manually. This prevents accidental data loss and allows administrators to recover data.
- **Recycle** (deprecated): Performs a basic scrub (rm -rf) on the volume and makes it available for new claims. Use dynamic provisioning instead.
- **Delete**: Both the PV and the underlying storage asset (cloud disk) are automatically deleted. This is the default for dynamically provisioned volumes.

### Rook
A 3rd party storage orchestrator tool that integrates deeply with Kubernetes. Rook automates storage management and turns distributed storage systems into self-managing, self-scaling, and self-healing storage services. It supports:
- **Ceph**: Distributed storage system for block, file, and object storage
- **Cassandra**: NoSQL database
- **NFS**: Network File System

Rook deploys storage providers as native Kubernetes applications, using operators to manage the lifecycle.

### ConfigMaps and Secrets
Both are Kubernetes objects used to store configuration data, but they serve different purposes:

**ConfigMaps**:
- Store non-sensitive configuration data
- Can contain key-value pairs or entire configuration files
- Can be consumed by pods as environment variables, command-line arguments, or mounted as files

**Secrets**:
- Store sensitive data like passwords, OAuth tokens, and SSH keys
- Data is base64 encoded (not encrypted by default)
- Can be encrypted at rest by enabling encryption in the API server
- Should be used with RBAC to restrict access

**Best Practices**:
- Use `immutable: true` to prevent changes to ConfigMaps and Secrets after creation
- This improves performance and prevents accidental updates
- Immutable ConfigMaps/Secrets can be cached aggressively by the kubelet

---

## Cloud Native Architecture

Cloud Native Architecture seeks to design systems that support the goals of cloud native technology. This architectural approach is important because it removes roadblocks to innovation through:

- **Software Agility**: Enables rapid development, testing, and deployment cycles. Teams can release features quickly and respond to market changes faster. Embraces microservices, containers, and automation.

- **Automation**: Reduces manual intervention and human error. Automates deployment, scaling, recovery, and infrastructure management. Infrastructure as Code (IaC) allows teams to manage infrastructure through version-controlled configuration files.

- **Robust/Reliable Systems**: Designs systems that can withstand failures through:
  - Redundancy and replication
  - Health checking and auto-recovery
  - Graceful degradation
  - Chaos engineering practices

Cloud native architecture emphasizes:
- **Microservices**: Breaking applications into small, independent services
- **Containers**: Packaging applications with their dependencies
- **Dynamic orchestration**: Using platforms like Kubernetes
- **DevOps practices**: Collaboration between development and operations
- **Continuous delivery**: Automated deployment pipelines

---

## Autoscaling

**Autoscaling** automatically adjusts compute resources allocated to an application or system in response to real-time demand. This ensures:
- **Cost efficiency**: Pay only for resources you need
- **Performance**: Handle traffic spikes without manual intervention
- **Reliability**: Maintain service levels during varying load conditions

### Vertical Autoscaling
Vertical scaling involves increasing the compute power of existing resources:
- Adding more CPU or memory to cluster nodes
- Increasing the resource limits of containers
- Resizing virtual machines

**Vertical Pod Autoscaler (VPA)**: Automatically adjusts CPU and memory requests/limits for containers based on usage. It can:
- Update resource requirements for running pods
- Evict and recreate pods with new resource settings
- Provide resource recommendations without actually updating pods

**When to use**: Applications that cannot be horizontally scaled (e.g., legacy databases), or when you want to optimize resource allocation.

### Horizontal Autoscaling
Horizontal scaling involves adding more instances of an application:
- Adding more pod replicas
- Adding new nodes to the cluster
- Distributing load across multiple instances

**Horizontal Pod Autoscaler (HPA)**: Monitors resource usage (CPU, memory) or custom metrics and automatically adjusts the number of pod replicas. It:
- Queries metrics every 15 seconds (configurable)
- Calculates desired replica count based on target utilization
- Updates the Deployment/ReplicaSet replica count
- Supports scale-up and scale-down with configurable stabilization windows

**Cluster Autoscaler**: Automatically adjusts the number of nodes in a cluster based on:
- Pending pods that cannot be scheduled due to insufficient resources
- Node utilization - removes underutilized nodes to save costs

Works with cloud providers to add/remove nodes from node pools.

**When to use**: Stateless applications, web services, API servers - anything that can run multiple identical instances.

---

## Observability

### Telemetry
The collection of measurements and data from distributed systems. Telemetry data includes:
- **Metrics**: Numerical measurements (CPU usage, request latency, error rates)
- **Logs**: Text records of events that occurred
- **Traces**: Records of requests flowing through distributed systems

### Observability
The ability to understand and measure the internal state of a system based on the data it generates. Observability goes beyond monitoring:
- **Monitoring**: Asks known questions about known problems
- **Observability**: Enables asking new questions about unknown problems

Good observability allows teams to:
- Debug production issues quickly
- Understand system behavior under load
- Identify performance bottlenecks
- Detect anomalies and predict failures

### Container Logs in k8s
- Kubernetes automatically collects logs from each container's stdout and stderr streams
- Logs are stored on the node where the container runs
- Access logs using `kubectl logs <pod-name>`
- Logs are rotated to prevent filling up disk space
- For production systems, use a centralized logging solution (ELK, Fluentd, Loki)

**Best Practices**:
- Write logs to stdout/stderr (not log files)
- Use structured logging (JSON format)
- Include correlation IDs for tracing requests
- Aggregate logs from all nodes in a central location

### Distributed System Tracing
Distributed tracing tracks requests as they flow through multiple microservices:

- Each incoming request receives a unique trace ID
- As the request propagates through services, each service adds span information
- Traces help identify:
  - Which service is causing slowness
  - Dependencies between services
  - Error propagation paths
  - Performance bottlenecks

**Key Concepts:**
- **Trace**: The complete journey of a request through all services, represented as a collection of spans
- **Span**: A single operation within a trace, representing work done by one service. Contains:
  - Operation name
  - Start time and duration
  - Tags (metadata)
  - Logs (events during the span)
  - Parent span ID

Popular distributed tracing tools: Jaeger, Zipkin, AWS X-Ray

### Prometheus
Open-source monitoring and alerting toolkit focused on gathering time-series metric data. Prometheus:
- **Pulls** metrics from instrumented applications via HTTP endpoints
- Stores metrics in a time-series database
- Provides a powerful query language (PromQL)
- Evaluates alerting rules and sends notifications
- Works well with Kubernetes (can auto-discover services)

**Architecture**:
- Prometheus server: Scrapes and stores metrics
- Client libraries: Instrument application code
- Exporters: Expose metrics from third-party systems
- Alertmanager: Handles alerts (grouping, routing, silencing)

#### Prometheus Metric Types

- **Counter**: A cumulative metric that only increases or resets to zero (e.g., total number of requests, errors). Used for rates and calculating growth.

- **Gauge**: A metric that can go up and down (e.g., current memory usage, number of active connections, temperature). Represents a snapshot of a current value.

- **Histogram**: Samples observations (e.g., request durations, response sizes) and counts them in configurable buckets. Also provides a sum of all observed values. Useful for calculating percentiles and distributions.

- **Summary**: Similar to histogram but calculates quantiles (e.g., 95th, 99th percentile) on the client side over a sliding time window. Provides more accurate quantiles but higher computational cost.

### Grafana
Visualization platform that creates dashboards and graphs from various data sources, especially Prometheus. Grafana enables:
- Building interactive, customizable dashboards
- Creating alerts based on metrics
- Sharing dashboards across teams
- Supporting multiple data sources simultaneously

Use Grafana to build useful visualizations of Prometheus data, making metrics more accessible to teams.

### FinOps
**Financial Operations (FinOps)** is a cultural practice that brings financial accountability to cloud spending. It combines:
- Observability tools to track resource usage
- Automation to optimize costs
- Data-driven decision making

**Key principles**:
- Teams collaborate on cloud spending
- Everyone takes ownership of cloud usage
- Centralized team drives FinOps practices
- Use real-time data for decisions
- Variable cost model of cloud requires new strategies

Common FinOps practices:
- Right-sizing resources based on actual usage
- Using spot instances for fault-tolerant workloads
- Implementing autoscaling to match demand
- Tagging resources for cost allocation
- Setting budgets and alerts

---

## GitOps

**GitOps** is a paradigm where Git is used as the single source of truth for declarative infrastructure and applications. The desired state of the system is versioned in Git, and automated processes ensure the actual state matches the desired state.

**Core Principles**:
1. **Declarative**: System state described declaratively (YAML manifests)
2. **Versioned**: All declarations stored in Git
3. **Pulled**: Changes automatically applied to the system
4. **Continuously reconciled**: Software agents ensure desired state matches actual state

**Benefits**:
- **Audit trail**: Full history of all changes in Git
- **Easy rollback**: Revert to any previous state
- **Improved collaboration**: Code review for infrastructure changes
- **Disaster recovery**: Git repo contains complete system state
- **Security**: GitOps operators don't need direct cluster access

### GitOps Tools

- **Flux**: A set of continuous and progressive delivery solutions for Kubernetes. Built on top of the GitOps Toolkit, Flux:
  - Monitors Git repositories for changes
  - Automatically applies changes to clusters
  - Supports multi-tenancy and multiple clusters
  - Provides image automation to update manifests when new container images are available

- **Argo CD**: Declarative, GitOps continuous delivery tool for Kubernetes. Features include:
  - Web-based UI for visualizing application state
  - Manual or automatic sync from Git repositories
  - Rollback to any application state in Git history
  - Health status analysis of deployed applications
  - SSO integration and RBAC for access control
  - Support for Helm, Kustomize, and plain YAML

---

## CI/CD

### Continuous Integration (CI)
A development practice where developers integrate code into a shared repository frequently (multiple times per day). Each integration is verified by an automated build and automated tests.

**Key aspects**:
- **Frequent commits**: Developers commit code changes regularly
- **Automated builds**: Code is compiled automatically when pushed
- **Automated testing**: Unit tests, integration tests run automatically
- **Fast feedback**: Developers know quickly if their changes broke something
- **Version control**: All code is in a version control system (Git)

**Benefits**:
- Detect integration problems early
- Reduce integration conflicts
- Maintain a continuously deployable codebase
- Improve code quality through automated testing

**Popular CI tools**: Jenkins, GitLab CI, GitHub Actions, CircleCI, Travis CI

### Continuous Delivery (CD)
A software development practice where code changes are automatically built, tested, and prepared for production release. Every change that passes automated tests can be deployed to production.

**Continuous Delivery vs Continuous Deployment**:
- **Continuous Delivery**: Changes are automatically prepared for production but require manual approval to deploy
- **Continuous Deployment**: Changes are automatically deployed to production without manual intervention

**Key aspects**:
- **Automated deployment pipelines**: Standardized, repeatable deployment process
- **Environment parity**: Development, staging, and production are similar
- **Blue-green deployments**: Run two identical production environments for zero-downtime deployments
- **Canary releases**: Gradually roll out changes to a subset of users
- **Feature flags**: Enable/disable features without deploying new code

**Benefits**:
- Deploy frequently with confidence
- Reduce deployment risk
- Get features to users faster
- Quick rollback if issues occur
- Improved developer productivity

**Popular CD tools**: Spinnaker, Argo CD, Flux, Jenkins X, GitLab

**CI/CD Pipeline Stages** (typical):
1. **Source**: Developer commits code to Git
2. **Build**: Application is compiled/built
3. **Test**: Automated tests run (unit, integration, security)
4. **Stage**: Deploy to staging environment
5. **Production**: Deploy to production (manual or automatic)
6. **Monitor**: Track application performance and errors

---

## Tips and Best Practices

1. Use `kubectl api-resources` command to list all of the available resource types in the cluster
2. Use `kubectl explain <resource>` command to get documentation about a resource (e.g., `kubectl explain pod.spec`)
3. You can define your own custom resource types using a CustomResourceDefinition (CRD), extending Kubernetes API
4. Use an init container to run a task before a pod's main container starts up (e.g., database migrations, configuration setup)
5. Scheduling occurs when a new pod is created and has not yet been assigned to a node
6. The scheduler takes into account multiple factors when selecting a node:
   - Resource requirements (CPU, memory)
   - Pod affinity/anti-affinity rules
   - Node affinity/anti-affinity
   - Taints and tolerations
   - Data locality
7. Control groups (cgroups) provide isolation, keeping container processes separate from everything else on the host
8. A headless service is a service with no cluster IP address (clusterIP: None), useful for direct pod-to-pod communication
9. A service without a selector requires endpoints to be manually created, allowing you to point to external services
10. A sidecar is an additional container running in a pod alongside the main container, often used for logging, monitoring, or proxy functionality
11. By default, Secret data is not encrypted, just base64 encoded. Enable encryption at rest for true security
12. Use `immutable: true` with ConfigMaps and Secrets to mark data as unchangeable, improving performance and preventing accidental modifications
13. Service mesh adds functionality around network communication by directing traffic through sidecar proxy containers
14. Span refers to the request's journey through a single component of the distributed system
15. Gauge is a Prometheus metric type that can both increase and decrease
16. Cloud native techniques and technologies support rapid innovation and reliability when delivering applications
17. Rapid innovation and reliability are two key goals of cloud native architecture
18. Use namespaces to organize resources and provide scope for names, useful for multi-tenancy
19. Resource quotas and limit ranges help prevent resource exhaustion and ensure fair resource allocation
20. Always define resource requests and limits for containers to ensure proper scheduling and resource management
21. Use labels and selectors extensively for organizing and selecting groups of resources
22. Implement health checks (liveness and readiness probes) for all production workloads
23. Use rolling updates for zero-downtime deployments
24. Store all configuration in ConfigMaps/Secrets, not in container images
25. Follow the principle of least privilege when setting up RBAC policies

---

## Additional Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [CNCF Cloud Native Glossary](https://glossary.cncf.io/)
- [KCNA Exam Curriculum](https://github.com/cncf/curriculum)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [GitOps Working Group](https://opengitops.dev/)

---

*These notes are based on KCNA (Kubernetes and Cloud Native Associate) certification training from Pluralsight. Good luck with your certification!*
