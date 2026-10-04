

# Kubernetes Interview Question Repository — Part 1

## 1. Kubernetes Fundamentals

### 1. What is Kubernetes?

**Answer:**  
Kubernetes is a container orchestration platform. It helps us deploy, manage, scale, and troubleshoot containerized applications.

---

### 2. Why do we use Kubernetes?

**Answer:**  
We use Kubernetes to manage containers automatically. It provides features like scaling, service discovery, load balancing, rolling updates, self-healing, and configuration management.

---

### 3. What is a Kubernetes cluster?

**Answer:**  
A Kubernetes cluster is a group of machines that run Kubernetes workloads. It consists mainly of a control plane and worker nodes.

---

### 4. What is a Node?

**Answer:**  
A node is a machine that runs applications inside the Kubernetes cluster. It can be a virtual machine or a physical machine.

---

### 5. What is the difference between the control plane and worker node?

**Answer:**  
The control plane manages the Kubernetes cluster and makes decisions about workloads. Worker nodes run the actual application pods.

---

### 6. What is a Pod?

**Answer:**  
A Pod is the smallest deployable unit in Kubernetes. It usually contains one container, but it can also contain multiple closely related containers that share the same network and storage.

---

### 7. Can a Pod have multiple containers?

**Answer:**  
Yes. Multiple containers can run inside one Pod. They share the same network namespace and can communicate with each other using `localhost`.

---

### 8. Why don't we normally create Pods directly?

**Answer:**  
Pods are temporary. If a Pod fails, Kubernetes does not automatically create a replacement when we create it directly. Controllers like Deployments manage Pods and make sure the desired number of Pods are running.

---

### 9. What is a Namespace?

**Answer:**  
A Namespace is used to logically separate resources inside a Kubernetes cluster. For example, we can have separate namespaces for development, staging, and production.

---

### 10. Why do we use namespaces?

**Answer:**  
Namespaces help us organize resources and provide isolation. We can also apply different permissions, resource limits, and policies to different namespaces.

---

# 2. Deployment and Replica Management

### 11. What is a Deployment?

**Answer:**  
A Deployment manages the lifecycle of application Pods. It maintains the desired number of replicas and supports features like rolling updates and rollbacks.

---

### 12. What is a ReplicaSet?

**Answer:**  
A ReplicaSet makes sure that the required number of Pod replicas are running. If a Pod goes down, the ReplicaSet creates a new one.

---

### 13. What is the difference between Deployment and ReplicaSet?

**Answer:**  
A ReplicaSet maintains the number of Pods, while a Deployment manages the ReplicaSet and provides features like rolling updates and rollbacks.

**Simple way to remember:**

`Deployment → ReplicaSet → Pods`

---

### 14. What happens when a Pod managed by a Deployment crashes?

**Answer:**  
The Deployment manages a ReplicaSet, and the ReplicaSet makes sure the desired number of Pods are running. So Kubernetes creates a replacement Pod.

---

### 15. What is a replica?

**Answer:**  
A replica is another running copy of the same Pod. For example, if we configure `replicas: 3`, Kubernetes tries to keep three Pods running.

---

### 16. What is scaling in Kubernetes?

**Answer:**  
Scaling means increasing or decreasing the number of application Pods based on requirements.

For example:

```bash
kubectl scale deployment myapp --replicas=5
```

---

### 17. What is the difference between scaling up and scaling down?

**Answer:**  
Scaling up means increasing the number of Pods. Scaling down means decreasing the number of Pods.

---

# 3. Deployment vs Pod — Very Common

### 18. What is the difference between a Pod and Deployment?

**Answer:**

| Pod | Deployment |
|---|---|
| Runs containers | Manages Pods |
| Smallest deployable unit | Higher-level controller |
| Can be temporary | Maintains desired state |
| Does not provide rolling updates by itself | Supports rolling updates and rollback |

**Interview answer:**

> A Pod runs the application container, while a Deployment manages Pods and makes sure the desired number of replicas are running.

---

# 4. StatefulSet

### 19. What is a StatefulSet?

**Answer:**  
A StatefulSet is used for applications that need stable identity, stable storage, or ordered deployment. It is commonly used for databases and other stateful applications.

---

### 20. What is the difference between Deployment and StatefulSet?

**Answer:**  
Deployment is generally used for stateless applications. StatefulSet is used when Pods need stable names, stable storage, or a specific order.

Example:

Deployment:

```text
myapp-7d8f9c-x1
myapp-7d8f9c-x2
```

StatefulSet:

```text
postgres-0
postgres-1
postgres-2
```

The StatefulSet Pod names remain predictable.

---

### 21. Why would you use StatefulSet for PostgreSQL?

**Answer:**  
PostgreSQL is a stateful application because it needs persistent data. StatefulSet can provide stable Pod identity and persistent storage through PersistentVolumeClaims.

---

# 5. DaemonSet

### 22. What is a DaemonSet?

**Answer:**  
A DaemonSet makes sure that a Pod runs on every eligible node, or on selected nodes.

It is commonly used for things like:

- Log collectors
- Monitoring agents
- Node-level agents

---

### 23. What is the difference between Deployment and DaemonSet?

**Answer:**  
A Deployment runs a desired number of replicas, while a DaemonSet normally runs one Pod on each eligible node.

---

# 6. Job and CronJob

### 24. What is a Job?

**Answer:**  
A Job runs a task until it successfully completes. Unlike a Deployment, it is designed for tasks that have an end.

Example:

```text
Database migration
Backup
Batch processing
```

---

### 25. What is a CronJob?

**Answer:**  
A CronJob runs a Job according to a schedule.

For example, we can run a backup every night.

---

### 26. Difference between Job and CronJob?

**Answer:**  
A Job runs a task once until it completes. A CronJob creates Jobs according to a schedule.

---

# 7. Service

### 27. What is a Kubernetes Service?

**Answer:**  
A Service provides a stable network endpoint for accessing Pods. Since Pod IPs can change, applications normally communicate through a Service instead of directly using Pod IPs.

---

### 28. Why do we need a Service if Pods already have IP addresses?

**Answer:**  
Pod IP addresses are temporary and can change when Pods are recreated. A Service provides a stable endpoint and forwards traffic to the appropriate Pods.

---

### 29. What are the main types of Kubernetes Services?

**Answer:**

The common types are:

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

---

### 30. What is ClusterIP?

**Answer:**  
ClusterIP exposes the Service inside the Kubernetes cluster. It is the default Service type.

---

### 31. What is NodePort?

**Answer:**  
NodePort exposes a Service through a port on each node. Traffic coming to that port is forwarded to the Service.

---

### 32. What is LoadBalancer?

**Answer:**  
LoadBalancer exposes a Service externally using a cloud provider's load balancer.

---

### 33. What is the difference between ClusterIP and NodePort?

**Answer:**  
ClusterIP is normally accessible only inside the cluster, while NodePort exposes the Service through a port on the Kubernetes nodes.

---

### 34. How does a Service know which Pods to send traffic to?

**Answer:**  
A Service uses labels and selectors. The selector matches Pods with specific labels and sends traffic to those Pods.

---

# 8. Labels and Selectors

### 35. What are labels in Kubernetes?

**Answer:**  
Labels are key-value pairs attached to Kubernetes resources. They help us identify and organize resources.

Example:

```yaml
labels:
  app: nginx
  environment: production
```

---

### 36. What is a selector?

**Answer:**  
A selector is used to find Kubernetes resources based on their labels.

For example, a Service can select all Pods with:

```yaml
app: nginx
```

---

### 37. What happens if Service selector doesn't match Pod labels?

**Answer:**  
The Service will not have the correct backend Pods, so traffic will not reach the application.

This is a **very common troubleshooting question**.

---

# 9. ConfigMap

### 38. What is a ConfigMap?

**Answer:**  
A ConfigMap stores non-sensitive configuration data separately from the application container.

For example:

```text
APP_ENV=production
APP_PORT=8080
LOG_LEVEL=info
```

---

### 39. How can we use a ConfigMap?

**Answer:**  
We can use a ConfigMap as environment variables or mount it as files inside a Pod.

---

### 40. Why shouldn't we store passwords in ConfigMaps?

**Answer:**  
ConfigMaps are intended for non-sensitive configuration. Passwords and other sensitive information should be stored in Secrets or an external secret manager.

---

# 10. Secret

### 41. What is a Secret?

**Answer:**  
A Secret is used to store sensitive information such as passwords, tokens, API keys, and certificates.

---

### 42. What is the difference between ConfigMap and Secret?

**Answer:**

| ConfigMap | Secret |
|---|---|
| Non-sensitive configuration | Sensitive information |
| Environment variables/config files | Passwords, tokens, keys |
| Example: `APP_ENV=prod` | Example: database password |

**Interview answer:**

> ConfigMap is used for normal application configuration, while Secret is used for sensitive information such as passwords and tokens.

---

### 43. Are Kubernetes Secrets encrypted?

**Good interview question.**

**Answer:**

> Kubernetes Secrets are not automatically encrypted just because they are stored as Secrets. By default, their values are base64 encoded. Encryption at rest can be configured for stronger protection.

This answer will impress more than simply saying **"Secrets are encrypted."**

---

### 44. What is the difference between encoding and encryption?

**Answer:**  
Encoding is used to represent data in another format and can easily be reversed. Encryption is used to protect data and requires a key or proper method to decrypt it.

---

# 11. kubeconfig

### 45. What is a kubeconfig file?

**Answer:**  
The kubeconfig file contains the information required by `kubectl` to connect to a Kubernetes cluster.

It normally contains:

- Cluster information
- User credentials
- Contexts
- Current context

---

### 46. What is a Kubernetes context?

**Answer:**  
A context tells `kubectl` which cluster, user, and namespace to use.

---

### 47. How do you check the current context?

```bash
kubectl config current-context
```

---

### 48. How do you list available contexts?

```bash
kubectl config get-contexts
```

---

### 49. How do you switch between Kubernetes clusters?

```bash
kubectl config use-context <context-name>
```

---

# 12. Kubernetes Architecture

### 50. What are the main components of the Kubernetes control plane?

**Answer:**

The main components are:

- API Server
- etcd
- Scheduler
- Controller Manager

---

### 51. What does kube-apiserver do?

**Answer:**  
The API Server is the main entry point to the Kubernetes cluster. Tools like `kubectl` communicate with Kubernetes through the API Server.

---

### 52. What is etcd?

**Answer:**  
etcd is a distributed key-value store that stores the Kubernetes cluster state and configuration.

---

### 53. What does the Scheduler do?

**Answer:**  
The Scheduler decides which worker node should run a newly created Pod based on available resources and scheduling rules.

---

### 54. What does Controller Manager do?

**Answer:**  
The Controller Manager runs different controllers that continuously check the cluster and try to bring the actual state closer to the desired state.

---

# 13. Worker Node Components

### 55. What is kubelet?

**Answer:**  
kubelet runs on each worker node. It communicates with the API Server and makes sure the containers described in the Pod specification are running.

---

### 56. What is kube-proxy?

**Answer:**  
kube-proxy helps implement Kubernetes Service networking and routes network traffic to the appropriate Pods.

---

### 57. What is a Container Runtime?

**Answer:**  
The container runtime is responsible for running containers. Kubernetes commonly uses containerd as the runtime.

---

# 14. Networking

### 58. How do Pods communicate with each other?

**Answer:**  
Pods receive IP addresses from the Kubernetes network. The Kubernetes networking system allows Pods to communicate with each other across nodes.

---

### 59. Can two Pods on different nodes communicate?

**Answer:**  
Yes. Kubernetes networking is designed so that Pods can communicate across nodes without needing manual routing between every Pod.

---

### 60. What is DNS in Kubernetes?

**Answer:**  
Kubernetes provides internal DNS so that Services can be accessed using DNS names instead of IP addresses.

For example:

```text
my-service.my-namespace.svc.cluster.local
```

---

### 61. How does one application communicate with another application inside Kubernetes?

**Answer:**  
Usually through a Kubernetes Service. The application can use the Service DNS name instead of directly using the Pod IP.

---

# 15. Ingress

### 62. What is Ingress?

**Answer:**  
Ingress manages external HTTP and HTTPS access to services inside a Kubernetes cluster. It can route traffic based on hostname or URL path.

---

### 63. What is an Ingress Controller?

**Answer:**  
An Ingress resource only defines the routing rules. The Ingress Controller actually implements those rules and handles the incoming traffic.

---

### 64. What is the difference between Service and Ingress?

**Answer:**  
A Service provides access to Pods, mainly inside the cluster. Ingress provides HTTP/HTTPS routing from outside the cluster to Services.

---

### 65. Can Ingress route traffic based on hostname?

**Answer:**  
Yes.

For example:

```text
api.example.com → api-service
web.example.com → web-service
```

---

### 66. Can Ingress route traffic based on path?

**Answer:**  
Yes.

Example:

```text
example.com/api → api-service
example.com/web → web-service
```

---

# 16. Storage

### 67. What is a PersistentVolume (PV)?

**Answer:**  
A PersistentVolume is storage available to the Kubernetes cluster for storing persistent data.

---

### 68. What is a PersistentVolumeClaim (PVC)?

**Answer:**  
A PVC is a request for storage made by an application. Kubernetes uses the PVC to provide the required persistent storage.

---

### 69. Difference between PV and PVC?

**Answer:**

> PV is the actual storage resource, while PVC is a request for that storage by an application.

---

### 70. Why do we need PVCs?

**Answer:**  
Pods are temporary, so data stored only inside a Pod can be lost when the Pod is recreated. PVCs allow applications to keep data on persistent storage.

---

### 71. What is StorageClass?

**Answer:**  
A StorageClass defines how Kubernetes should provision storage. It can automatically create a PersistentVolume when a PVC requests storage.

---

# 17. Helm

Since your resume specifically mentions Helm and says you deployed 50+ microservices using Helm, **expect several Helm questions.** Punit_Saini_DevOps_Resume

### 72. What is Helm?

**Answer:**  
Helm is a package manager for Kubernetes. It helps us package, configure, and deploy Kubernetes applications using reusable charts.

---

### 73. What is a Helm Chart?

**Answer:**  
A Helm Chart is a collection of Kubernetes YAML templates and configuration files used to deploy an application.

---

### 74. What is values.yaml?

**Answer:**  
`values.yaml` contains configuration values that are used by Helm templates.

For example:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25"
```

---

### 75. Why do we use Helm instead of writing YAML files separately?

**Answer:**  
Helm allows us to create reusable templates. We can use the same chart for different environments by changing values instead of maintaining completely separate YAML files.

---

### 76. What is `helm install`?

**Answer:**

```bash
helm install myapp ./mychart
```

It installs a Helm chart into Kubernetes.

---

### 77. What is `helm upgrade`?

**Answer:**

```bash
helm upgrade myapp ./mychart
```

It updates an existing Helm release using the new chart or values.

---

### 78. What is a Helm release?

**Answer:**  
A release is an installed instance of a Helm chart in a Kubernetes cluster.

---

### 79. How do you check Helm releases?

```bash
helm list -A
```

---

### 80. How do you see the values used by an existing release?

```bash
helm get values <release-name>
```

---

# 18. Probes

### 81. What is a liveness probe?

**Answer:**  
A liveness probe checks whether the application is still running correctly. If the check fails repeatedly, Kubernetes can restart the container.

---

### 82. What is a readiness probe?

**Answer:**  
A readiness probe checks whether the application is ready to receive traffic. If it fails, Kubernetes removes the Pod from Service endpoints.

---

### 83. Difference between liveness and readiness probe?

**Simple interview answer:**

> Liveness tells Kubernetes whether the application is alive. Readiness tells Kubernetes whether the application is ready to receive traffic.

---

### 84. What is a startup probe?

**Answer:**  
A startup probe is useful for applications that take a long time to start. Kubernetes waits for the startup probe to succeed before starting liveness and readiness checks.

---

# 19. Requests and Limits

### 85. What are resource requests?

**Answer:**  
Requests define the minimum CPU and memory resources required by a container. Kubernetes uses requests when deciding which node can run the Pod.

---

### 86. What are resource limits?

**Answer:**  
Limits define the maximum CPU and memory a container can use.

---

### 87. Difference between requests and limits?

**Answer:**

> Requests are used mainly for scheduling the Pod, while limits control the maximum resources the container can use.

---

### 88. What happens if a container exceeds its memory limit?

**Answer:**  
The container can be terminated by Kubernetes and may show an `OOMKilled` status.

---

# 20. Rolling Updates and Rollbacks

### 89. What is a rolling update?

**Answer:**  
A rolling update gradually replaces old Pods with new Pods without stopping the entire application at once.

---

### 90. What is rollback?

**Answer:**  
Rollback means going back to a previous working version of an application after a failed or problematic deployment.

---

### 91. How do you check Deployment rollout status?

```bash
kubectl rollout status deployment/myapp
```

---

### 92. How do you check rollout history?

```bash
kubectl rollout history deployment/myapp
```

---

### 93. How do you rollback a Deployment?

```bash
kubectl rollout undo deployment/myapp
```

---

# 21. Config and Environment

### 94. How can you pass environment variables to a Pod?

**Answer:**  
We can define them directly in the Pod specification or load them from a ConfigMap or Secret.

---

### 95. What is `envFrom`?

**Answer:**  
`envFrom` allows us to load multiple environment variables from a ConfigMap or Secret.

---

### 96. Can ConfigMap and Secret be mounted as files?

**Answer:**  
Yes. Both ConfigMaps and Secrets can be mounted into containers as files.

---

# 22. RBAC

### 97. What is RBAC?

**Answer:**  
RBAC stands for Role-Based Access Control. It controls what actions users or service accounts can perform on Kubernetes resources.

---

### 98. What is a Role?

**Answer:**  
A Role defines permissions within a specific namespace.

---

### 99. What is a ClusterRole?

**Answer:**  
A ClusterRole defines permissions at the cluster level and can also be used for namespace-level access.

---

### 100. Difference between Role and ClusterRole?

**Answer:**

> Role is limited to a namespace, while ClusterRole can provide permissions across the cluster.

---

### 101. What is RoleBinding?

**Answer:**  
RoleBinding connects a Role to a user, group, or ServiceAccount within a namespace.

---

### 102. What is ClusterRoleBinding?

**Answer:**  
ClusterRoleBinding connects a ClusterRole to a user, group, or ServiceAccount at the cluster level.

---

# 23. ServiceAccount

### 103. What is a ServiceAccount?

**Answer:**  
A ServiceAccount provides an identity for applications running inside Kubernetes Pods.

---

### 104. Why do Pods need ServiceAccounts?

**Answer:**  
A Pod can use a ServiceAccount to authenticate when it needs to communicate with the Kubernetes API or other systems that support its identity.

---

# 24. Taints, Tolerations and Affinity

### 105. What is a taint?

**Answer:**  
A taint prevents Pods from being scheduled on a node unless the Pod has a matching toleration.

---

### 106. What is a toleration?

**Answer:**  
A toleration allows a Pod to be scheduled on a node that has a matching taint.

---

### 107. What is nodeSelector?

**Answer:**  
`nodeSelector` is a simple way to tell Kubernetes to schedule a Pod only on nodes with specific labels.

---

### 108. What is node affinity?

**Answer:**  
Node affinity provides more flexible rules for deciding which nodes a Pod should run on.

---

### 109. Difference between taint/toleration and affinity?

**Simple answer:**

> Taints and tolerations control which Pods are allowed on a node. Affinity controls where a Pod prefers or requires to run.

---

# 25. kubectl

### 110. How do you see Pods?

```bash
kubectl get pods
```

---

### 111. How do you see Pods in all namespaces?

```bash
kubectl get pods -A
```

---

### 112. How do you get detailed information about a Pod?

```bash
kubectl describe pod <pod-name>
```

---

### 113. How do you check Pod logs?

```bash
kubectl logs <pod-name>
```

---

### 114. How do you enter a running container?

```bash
kubectl exec -it <pod-name> -- /bin/bash
```

If bash is unavailable:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

---

### 115. How do you check Kubernetes resources?

```bash
kubectl get all
```

---

### 116. How do you get YAML of an existing resource?

```bash
kubectl get deployment myapp -o yaml
```

---

### 117. How do you see events?

```bash
kubectl get events
```

For troubleshooting, this is extremely useful.

---

# 26. Very Important Conceptual Questions

### 118. What is desired state in Kubernetes?

**Answer:**  
Desired state is the state we want the cluster to have. For example, if we specify three replicas, the desired state is three running Pods.

---

### 119. What is the actual state?

**Answer:**  
Actual state is what is currently running in the cluster.

Kubernetes continuously tries to make:

```text
Actual State → Desired State
```

---

### 120. What happens when you run `kubectl apply -f deployment.yaml`?

**Good interview answer:**

> `kubectl` sends the configuration to the Kubernetes API Server. Kubernetes stores the desired state and the controllers work to create or update the required resources, such as the Deployment, ReplicaSet, and Pods.

---

