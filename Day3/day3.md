Kubernetes Security & Core Concepts Guide
A comprehensive guide to Kubernetes security tools (Trivy, Kyverno, KubeLinter), static pods, pod lifecycle, sidecar containers, init containers, and best practices for production-grade clusters.

🔐 Trivy - Container Image Vulnerability Scanner
Trivy is a lightweight, open-source vulnerability scanner designed for container images, Kubernetes manifests, and infrastructure as code. It helps identify security issues early in the development lifecycle.
🔍 Key Features

Scans Multiple Targets: Docker/OCI images, Kubernetes manifests, and filesystems.
Layer-by-Layer Analysis: Inspects every layer of a container image for vulnerabilities.
Built-in Database: Uses an up-to-date vulnerability database, no external dependencies required.
CI/CD Integration: Seamlessly integrates with GitHub Actions, Jenkins, GitLab CI, and more.
Customizable Output: Supports JSON, table, or SARIF formats for reporting.

🔧 Usage Examples
# Scan an image for all vulnerabilities
trivy image nginx:latest

# Filter for critical vulnerabilities only
trivy image nginx:latest --severity CRITICAL,HIGH

# Scan a specific image tag and output to JSON
trivy image --format json --output report.json nginx:1.21

# Scan a private image (requires authentication)
trivy image --username <user> --password <pass> my-private-repo/nginx


⚠️ Best Practice: Always scan images before deployment. Avoid using images with critical or high-severity vulnerabilities in production. Automate scans in CI/CD pipelines to catch issues early.

✅ Supported Ecosystems

Public/private container registries (Docker Hub, AWS ECR, GCR, etc.).
Languages: Python, Node.js, Ruby, Java, and more.
Operating Systems: Debian, Ubuntu, Alpine, RHEL, etc.

📚 Real-World Example
To integrate Trivy into a GitHub Actions workflow:
name: Container Image Scan
on: [push]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'nginx:latest'
          format: 'table'
          exit-code: '1'  # Fail on critical vulnerabilities
          severity: 'CRITICAL,HIGH'


🛡️ Kyverno - Kubernetes Native Policy Engine
Kyverno is a Kubernetes-native policy engine for validating, mutating, and generating resources. It simplifies policy management with YAML-based configurations and enforces security best practices.
📦 Core Components

Custom Resource Definitions (CRDs): Define policies as Kubernetes resources.
Admission Controller: Intercepts and processes API requests (validate/mutate).
Policy Reports Controller: Generates detailed compliance reports.
Cleanup Controller: Manages resource lifecycle by deleting outdated objects.
Background Controller: Applies policies to existing resources retroactively.

🔧 Installation
# Add Kyverno Helm repository
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update

# Install Kyverno in its own namespace
helm install kyverno kyverno/kyverno -n kyverno --create-namespace

📝 Example Policy: Restrict Container Runtime
This policy ensures all pods use the secure kata runtime:
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-runtime
spec:
  validationFailureAction: Enforce
  rules:
    - name: require-kata-runtime
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "Pods must use kata runtime."
        pattern:
          spec:
            runtimeClassName: "kata"

📌 Applying and Monitoring Policies
# Apply the policy
kubectl apply -f restrict-runtime.yaml

# Check policy compliance
kubectl get policyreport -n kyverno

🧠 Key Concepts

Validation Policies: Enforce rules (e.g., reject non-compliant pods). Ideal for production.
Mutating Policies: Modify resources on creation (e.g., add labels or resource limits). Use cautiously.
Generate Policies: Create new resources based on triggers (e.g., auto-generate ConfigMaps).

✅ Best Practices

Use Enforce mode in production to block non-compliant resources.
Test policies in Audit mode first to avoid disruptions.
Regularly review PolicyReport CRDs to ensure compliance.


🔍 KubeLinter - Static Analysis for Kubernetes YAML
KubeLinter is a static analysis tool that validates Kubernetes YAML files and Helm charts for misconfigurations and security issues.
✅ What It Checks

Missing CPU/memory resource limits and requests.
Containers running as root or with excessive privileges.
Missing liveness/readiness probes.
Deprecated API versions.
Insecure port configurations.

🔧 Installation & Usage
# Install KubeLinter (via Homebrew for macOS/Linux)
brew install kube-linter

# Lint a single YAML file
kube-linter lint pod.yaml

# Lint an entire directory
kube-linter lint ./manifests/

📚 Example Output
For a pod without resource limits:
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest

Running kube-linter lint pod.yaml might output:
pod.yaml: (object: default/insecure-pod) container "nginx" does not have resource limits set (check: resource-limits)


⚠️ Best Practice: Use KubeLinter in CI/CD pipelines to catch misconfigurations before deployment.


📦 Static Pods vs. Normal Pods
Static pods are created and managed directly by the kubelet on a specific node, while normal pods are managed by the Kubernetes control plane.



Feature
Normal Pod
Static Pod



Created By
kube-apiserver
kubelet


Managed By
Controllers (Deployment, ReplicaSet)
None


Scheduling
kube-scheduler
Fixed to a specific node


Storage
etcd
Local manifest file


Use Case
Application workloads
Control plane components (e.g., kube-apiserver, etcd)


Rescheduling
Automatic across cluster
Node-specific, no rescheduling


📁 Static Pod Manifest Directory
Static pod manifests are stored in:
/etc/kubernetes/manifests/

Common files:

kube-apiserver.yaml
etcd.yaml
kube-scheduler.yaml
kube-controller-manager.yaml

🔄 How Static Pods Work

systemd starts the kubelet.
Kubelet reads manifests from the configured directory.
Kubelet creates containers for static pods and reports their status to the API server as "mirror pods."


✨ Use Case: Static pods are critical for bootstrapping control plane components in self-hosted clusters.


🌀 DaemonSets vs. Sidecar Containers

DaemonSet: Ensures one pod runs on every node in the cluster. Ideal for node-level tasks like logging (Fluentd) or monitoring (Prometheus node-exporter).
Sidecar Container: A helper container running alongside the main application container in the same pod. Used for tasks like logging, proxying, or configuration management.

📚 Example: DaemonSet for Fluentd
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
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
          image: fluent/fluentd:latest

📚 Example: Sidecar for Logging
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
    - name: main-app
      image: nginx:latest
    - name: logging-sidecar
      image: fluentd:latest
      env:
        - name: LOG_PATH
          value: "/var/log/nginx"


✨ Key Difference: DaemonSets operate at the node level, while sidecars operate at the pod level.


🚀 Init Containers
Init containers run to completion before the main containers in a pod start. They are used for setup tasks like fetching configurations, initializing databases, or waiting for dependencies.
📚 Example: Init Container for Dependency Check
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    - name: wait-for-db
      image: busybox:latest
      command: ['sh', '-c', 'until nslookup db-service; do echo waiting for db; sleep 2; done;']
  containers:
    - name: main-app
      image: nginx:latest
      ports:
        - containerPort: 80


✅ Best Practice: Use multiple init containers for complex setups, ensuring each completes sequentially.


🛠️ Pod Lifecycle
The lifecycle of a pod involves multiple phases and states, managed by the Kubernetes control plane.
🔄 Phases of a Pod

Pending: Pod is created but not yet running (e.g., scheduling or image pulling).
Running: At least one container is running or starting.
Succeeded: All containers have terminated successfully.
Failed: At least one container has terminated with an error.
Unknown: Kubelet cannot communicate with the API server.

🔄 Container States

Waiting: Container is preparing (e.g., pulling image).
Running: Container is executing.
Terminated: Container has stopped (success or failure).

📚 Example: Monitoring Pod Lifecycle
# Check pod status
kubectl get pods -o wide

# View detailed events
kubectl describe pod <pod-name>


🔁 Sidecar Containers (Kubernetes 1.29+)
Sidecar containers are tightly coupled with the main application container and share the same lifecycle (restarted together).
📚 Example: Sidecar with Restart Policy
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  restartPolicy: Always
  containers:
    - name: main-app
      image: nginx:latest
    - name: sidecar
      image: prom/prometheus:latest
      args:
        - --config.file=/etc/prometheus/prometheus.yml

🔧 Accessing a Sidecar
kubectl exec -it app-with-sidecar -c sidecar -- /bin/sh


✅ Best Practice: Explicitly define restartPolicy to ensure consistent behavior.


🧨 Pod Termination
When a pod is deleted, Kubernetes allows a grace period (default: 30 seconds) for containers to shut down gracefully:

SIGTERM: Sent to the main process in each container.
SIGKILL: Sent after the grace period if the process doesn’t exit.

🔧 Customizing Grace Period
apiVersion: v1
kind: Pod
metadata:
  name: graceful-shutdown
spec:
  terminationGracePeriodSeconds: 60
  containers:
    - name: nginx
      image: nginx:latest


📦 etcd Backup Importance
etcd is the distributed key-value store that holds the entire Kubernetes cluster state. Losing etcd without a backup can render the cluster unrecoverable.
🔧 Backup Command
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

✅ Best Practices

Schedule regular etcd backups (e.g., daily).
Store backups in a secure, off-cluster location.
Test restore procedures periodically.


☁️ Cloud Provider Considerations (EKS, GKE, AKS)
Managed Kubernetes services abstract control plane management:

Control Plane: Fully managed (kube-apiserver, etcd, etc.).
Static Pods: Used internally but not accessible to users.
High Availability: Built-in redundancy and scaling.
Security: Cloud providers enforce best practices (e.g., RBAC, network policies).


✨ Tip: Use cloud-native tools like AWS IAM roles for EKS or GKE Workload Identity for secure access.


✅ Summary Table



Tool/Component
Purpose



Trivy
Scans container images for vulnerabilities


Kyverno
Enforces, mutates, and generates policies


KubeLinter
Lints YAML files for misconfigurations


Static Pods
Runs control plane components on nodes


Init Containers
Performs setup tasks before main containers


Sidecar Containers
Runs helper containers in the same pod


DaemonSet
Ensures one pod per node for system tasks


