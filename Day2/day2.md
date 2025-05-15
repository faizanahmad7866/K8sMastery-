📘 Kubernetes Day 2 — Easy and Clear Notes

🧠 Goal: Learn Kubernetes setup, components, and basic tools in a beginner-friendly way with real-life examples.


⚙️ Cluster Setup — Local vs Production
🧪 Local Setup (For Practice and Learning)
1. Kind (Kubernetes IN Docker)

Think of Kind as your personal Kubernetes lab.
It runs Kubernetes inside Docker containers, requiring minimal system resources.
Ideal for learning, testing, and CI/CD pipelines.
⚠️ Not suitable for production as it’s designed for local environments.

kind create cluster --name my-cluster

2. Kubeadm

Used to manually create Kubernetes clusters.
Great for learning how clusters are built step by step.
Offers full control but requires more setup time compared to Kind.


☁️ Cloud Production Setup
In production, you typically don’t create clusters manually. Instead, use managed Kubernetes services from cloud providers, which handle updates, security patches, and scaling:



Cloud Provider
Kubernetes Service



AWS
EKS


Google Cloud
GKE


Azure
AKS



🏠 On-Premise Production Setup

OpenShift by RedHat is popular for companies running their own servers.
Includes extras like dashboards and enhanced security features.

💡 Tip: Start with Kind or Kubeadm for practice. Move to EKS, GKE, or AKS for cloud-based production environments.

🧠 Kubernetes Core Concepts
🔧 Kubernetes Components (Behind the Scenes)

kube-apiserver: The receptionist — handles all API requests.
etcd: The database — stores cluster state and configuration.
kube-scheduler: Assigns pods to nodes based on resource needs.
kubelet: Manages containers on each node.
kube-proxy: Manages networking for pods and services.

📦 Kubernetes Objects (You Create These)

Pod: Runs your application.
Service: Exposes your app to other pods or external users.
ConfigMap: Stores configuration settings.
Ingress: Routes external traffic like a web server.

kubectl api-resources

📌 Learn more: Kubernetes API Reference

🔁 Kubernetes Versioning

Use a fixed version (e.g., 1.29.2), not latest.
The control plane (master) must be the same or newer than worker nodes.

kubectl version --short

⚠️ Using latest risks breaking apps due to unexpected changes.

🐳 Container & Image Basics
📦 What’s Inside a Container?
A container is like a zip file containing:

App code (e.g., app.py, index.html)
Dependencies (e.g., Express, Flask)
A minimal OS (if needed)
The runtime (e.g., Python, Node.js)

🏷️ Use Tags for Versions
nginx:1.25.0   ✅ Good
nginx:latest   ❌ Avoid in production

🗂️ Where Images Are Stored

Docker Hub
GitHub Container Registry
AWS ECR, Google Artifact Registry


📦 Base Images (Build Smarter)



Base Image
Use Case



scratch
Empty image, minimal size


alpine
Tiny Linux (~5MB), general use


slim
Lightweight official image


distroless
No shell, enhanced security


🛠️ Multi-Stage Builds
Why use them?

Smaller images: Copy only the final app, discarding build tools.
Faster builds: Optimize layers for caching.
More secure: Exclude unnecessary tools (e.g., compilers).

# Build stage
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main

# Final stage
FROM gcr.io/distroless/static
COPY --from=builder /app/main /
ENTRYPOINT ["/main"]

✅ distroless is ideal for production — no shell or extra tools reduces attack surfaces.
📌 Learn more: Docker Multi-Stage Builds

🔁 Stateless vs Stateful Apps
🟢 Stateless Apps

Don’t store data between restarts.
Easy to scale: Add more pods without worrying about data consistency.
Examples: Web servers (NGINX), APIs (Node.js, Flask).
Why easy? Each pod is independent, so replicas can be added or removed seamlessly.

🔵 Stateful Apps

Store data that persists across restarts.
Examples: Databases (MySQL, MongoDB).
Require careful handling (e.g., Persistent Volumes, StatefulSets).

🛑 Best Practice: Run databases outside Kubernetes (e.g., AWS RDS, Google Cloud SQL) for simpler management and reliability.

🧑‍💻 kubectl — Your Kubernetes Command Tool
Use kubectl to interact with your cluster:
kubectl get nodes
kubectl get pods
kubectl get services  # svc = services
kubectl describe pod <name>
kubectl get events  # Useful for troubleshooting

⚠️ Avoid aliases like k=kubectl in shared scripts or team projects to prevent confusion.
📌 Learn more: kubectl Cheat Sheet

👀 Use k9s for a Visual Terminal UI

A terminal-based UI for real-time monitoring.
Shows logs, pod status, and restarts at a glance.

k9s

📌 Get it here: k9s GitHub

🧬 Kubernetes Architecture (Simple View)
Cluster → Node → Pod → Container


Cluster: The entire Kubernetes system.
Node: A single machine (physical or virtual).
Pod: The smallest deployable unit, usually containing one container.
Container: Runs your application.


🧪 Sample Pod YAML (Beginner Example)
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25.0
      ports:
        - containerPort: 80

✍️ YAML Explanation



Field
Description



apiVersion
Kubernetes API version (e.g., v1)


kind
Object type (e.g., Pod, Service)


metadata.name
Unique name for the pod


spec
Defines the pod’s contents


containers
List of containers to run


image
Container image (e.g., nginx:1.25.0)


ports
Ports exposed by the container


📌 Tip: Pods are ephemeral. Use a Deployment to manage pod lifecycles and ensure restarts.

📋 Quick Recap



Topic
Best Practice



Local Cluster
Use Kind or Kubeadm for practice


Production Setup
Use EKS, GKE, or AKS for cloud


Image Tags
Use fixed versions, avoid latest


Base Image
Prefer alpine or distroless


kubectl
Use full commands, avoid aliases in teams


UI Tools
Use k9s for easier debugging


Pod YAML
Start simple, transition to Deployments



🎯 Coming Up

Deployments & ReplicaSets
Service Types: ClusterIP, NodePort, LoadBalancer
Ingress Controllers
Volumes, ConfigMaps, Secrets


