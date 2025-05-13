# 🏛️ Kubernetes Architecture Overview

Kubernetes (K8s) is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. Its architecture is divided into control plane (master nodes) and worker nodes, which work together to maintain the desired state of your applications.

---

## 🌐 Key Components

* **Control Plane**: Manages the cluster (e.g., `kube-apiserver`, `etcd`, `scheduler`, `controllers`).
* **Worker Nodes**: Run application workloads (e.g., `kubelet`, `kube-proxy`, container runtime).
* **Add-ons**: Extend functionality—networking (CNI), storage (CSI), monitoring, and logging.

---

## 🧩 Kube-API Server: The Gateway to Kubernetes

The `kube-apiserver` is the front-end of the Kubernetes control plane. It serves as the central access point for all interactions with the cluster.

### 📌 Key Responsibilities

* **Authentication**

  * Verifies user/service identity via certificates, tokens, or other auth mechanisms.
  * *Example:* When a user runs `kubectl get pods`, the API server checks their token/certificate.

* **Authorization**

  * Determines whether the authenticated identity can perform the requested action.
  * *Example:* A user with read-only access cannot run `kubectl delete pod <pod-name>`.

* **Admission Control**

  * Validates or mutates requests before they are persisted to `etcd`.
  * Two main types:

    * **Mutating Admission Controller**: Adds defaults like memory limits.
    * **Validating Admission Controller**: Rejects requests that violate rules (e.g., missing labels).

### 💡 Real-World Scenario: Enforcing Resource Limits

* **Policy**: All pods must have a memory limit of `500Mi`.
* **Request**: A developer submits a pod with `1Gi` memory.

**Admission Workflow:**

1. **Mutating Webhook**: Rewrites `1Gi` to `500Mi`.
2. **Validating Webhook**: Ensures the updated pod meets policies.
3. **Persistence**: Approved spec is stored in `etcd`.

**Why It Matters**:

* Prevents resource exhaustion.
* Enforces org-wide policies.
* Promotes stability and cost-efficiency.

---

## 🗄️ ETCD: The Heart of Kubernetes

`etcd` is a distributed key-value store that stores the entire state of the Kubernetes cluster.

### 🔑 Key Features

* **Distributed**: Runs across multiple nodes.
* **NoSQL**: Stores data as key-value pairs.
* **Unencrypted by Default**: Encryption must be explicitly configured.
* **Write-Ahead Logging (WAL)**: Ensures data durability.
* **Raft Consensus**: Guarantees consistency through leader election.

### 🛠️ How It Works

* **Write Flow**:

  1. Request is logged to WAL.
  2. Log is replicated to follower nodes.
  3. Once a quorum agrees, the change is committed.
* **Read Flow**:

  1. Client requests are handled by the current leader node.
  2. Data is read and sent back via the API server.

### 📌 Real-World Example

If you deploy a web app with 3 replicas and one node fails, the deployment's desired state is preserved in etcd. The controller uses this state to reschedule missing pods on healthy nodes.

---

## 🔄 Controllers: Maintaining Desired State

Controllers constantly monitor the state of Kubernetes objects and ensure the actual state matches the desired state.

### 🛠️ Types of Controllers

* **Node Controller**: Detects node failures.
* **Service Controller**: Updates endpoints.
* **ReplicaSet Controller**: Ensures the correct number of pod replicas.
* **CronJob Controller**: Schedules recurring jobs.
* **Custom Controller**: Tailored logic using Operators.

### 🚀 Detailed Example: ReplicaSet Controller

1. You define a deployment with 5 replicas of a pod.
2. Due to node crash, only 2 pods are running.
3. The ReplicaSet controller detects the discrepancy.
4. It immediately creates 3 new pods to match the desired replica count.

This automation is critical for self-healing.

---

## ⚖️ Scheduler: Placing Pods on Nodes

The `kube-scheduler` assigns unscheduled pods to appropriate nodes.

### 🔄 Scheduling Flow

1. **Queue**: New pods enter the scheduling queue.
2. **Filtering**: Evaluates which nodes meet pod requirements.
3. **Scoring**: Ranks eligible nodes.
4. **Binding**: Assigns the pod to the highest-scoring node.

### 🌍 Detailed Example

* Pod needs: 2 vCPUs + GPU
* Cluster: 10 nodes, only 2 have GPUs
* Filter Phase: 8 nodes excluded (no GPU)
* Score Phase: Picks least-loaded GPU node
* Bind Phase: Pod assigned

You can influence this behavior using taints, tolerations, and affinity rules.

---

## 🧱 Kubelet: The Node Agent

Runs on each node and ensures containers in pods are healthy and running.

### Responsibilities

* Syncs with `kube-apiserver`
* Manages container lifecycle via the container runtime
* Automatically runs static pods defined in `/etc/kubernetes/manifests`

### 🧪 Real-World Example

If a pod container crashes due to memory pressure, the kubelet restarts it per the `restartPolicy` defined in the pod spec.

---

## 🌐 Kube-Proxy: Networking Maestro

Responsible for maintaining network rules on nodes.

### Core Duties

* Manages iptables/IPVS rules
* Forwards traffic to appropriate pod endpoints

### 📌 Real-World Example

A frontend pod calls a backend service via its ClusterIP. Kube-proxy routes the request to a backend pod IP, enabling service discovery and load balancing.

---

## 🐋 Container & Platform Interfaces

Kubernetes uses standard interfaces for container runtime, networking, and storage.

| Interface | Full Form                   | Role                                                  |
| --------- | --------------------------- | ----------------------------------------------------- |
| 🐋 CRI    | Container Runtime Interface | Manages container lifecycle (e.g., containerd, CRI-O) |
| 💾 CSI    | Container Storage Interface | Manages storage volumes (e.g., AWS EBS, Ceph)         |
| 🌐 CNI    | Container Network Interface | Manages pod networking (e.g., Calico, Flannel)        |

### 🔍 CRI Deep Dive

* **Container Runtime** (e.g., `containerd`) talks to `runc` to start containers.
* **runc** applies Linux namespaces, cgroups, seccomp profiles.
* Example: When a pod is scheduled, the CRI directs containerd to create a container using runc with memory limits.

---

## 📡 Watch API: Real-Time Updates

Monitors etcd for changes and notifies relevant components.

### 🧠 How It Works

* Components (like controllers) watch resources
* Get notified on create/update/delete
* Act accordingly (e.g., re-create a deleted pod)

### 🌍 Real-World Example

A user manually deletes a pod. The ReplicaSet controller sees this event via Watch API and creates a new pod to maintain replica count.

---

## 🔗 Webhooks: Extending Kubernetes

Webhooks allow custom logic execution during admission control.

### Types

* **Mutating Admission Webhook**: Modifies the object (e.g., inject sidecar containers)
* **Validating Admission Webhook**: Enforces policy (e.g., reject pods missing labels)

---

## 📁 Critical Kubernetes Directories

```bash
/etc/kubernetes/manifests
```

* Stores static pod manifests (kube-apiserver, etcd, etc.)
* Kubelet monitors this directory to launch static pods

```bash
/var/log
```

* Stores logs for Kubernetes components and containers

---

## 🔍 Check Admission Plugins

To see which admission plugins are enabled:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

---

## 🌐 Kubernetes Communication

### Protocol:

* Uses **Protobuf** (Protocol Buffers) for efficient, serialized communication between components.

### Why Protobuf?

* Faster and lighter than JSON
* Helps reduce network and CPU overhead

### Example

Kube-apiserver communicates with etcd using Protobuf to rapidly write and read cluster state.
