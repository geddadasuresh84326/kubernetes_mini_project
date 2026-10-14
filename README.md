# 🚀 Cloud-Native Python Microservice Template


An end-to-end cloud-native implementation demonstrating how to containerize a standard **Python Application** and orchestrate it using **Kubernetes (Minikube)**. This project highlights best practices for container build pipelines, local single-node cluster networking, and zero-downtime application horizontal scaling.

---

## 🏗️ Core Architecture & High Availability

```text
       [ External Client Request ]
                    │
                    ▼
     [ Minikube Cluster Service ] (Port: 5000)
                    │
        ┌───────────┴───────────┐  (Load Balanced Traffic)
        ▼                       ▼
┌───────────────┐       ┌───────────────┐
│   Pod #1      │       │   Pod #2      │  <-- Managed by K8s Deployment
│ App Container │       │ App Container │  <-- Image: local-app:latest
└───────────────┘       └───────────────┘
        └───────────┬───────────┘
                    ▼
  [ 1-Node Minikube Control-Plane ]
```

---

## 🛠️ Key Engineering Highlights
* **High Availability Configuration:** Scaled to a declarative **2-pod architecture** on a single-node control plane to prevent single-point-of-failure (SPOF) inside the cluster namespace.
* **Local Image Sideloading:** Avoided external registry overhead by caching and loading the application container image directly into the internal Minikube runtime node.
* **Strict Security Posture:** Stripped all infrastructure parameters and cloud provider credentials out of the application code to strictly follow standard 12-Factor App design patterns.

---

## 💻 Tech Stack
* **Language:** Python 3.10.12
* **Containerization:** Docker (Engine & CLI)
* **Orchestration Tool:** Kubernetes 
* **Local Cluster Provider:** Minikube (1-Node Control Plane)

---

## 🏃‍♂️ Quick Start & Deployment Guide

Follow these sequential terminal steps to replicate the complete build and infrastructure setup locally.

### 1. Prerequisites
Ensure you have the following environments configured on your machine:
* [Docker Desktop / Engine](https://docker.com)
* [Minikube](https://k8s.io)
* [kubectl CLI](https://kubernetes.iodocs/tasks/tools/)

### 2. Containerize the Python Application
Build the application container image with a lightweight footprint:
```bash
docker build -t local-app:latest .
```
Verify the build inside your local environment:
```bash
docker images | grep local-app
```

### 3. Spin Up the Local Kubernetes Infrastructure
Initialize your single-node control-plane cluster:
```bash
minikube start
```

Because Minikube operates inside its own isolated hypervisor or runtime environment, sideload your locally-built image straight into the cluster’s node image cache:
```bash
minikube image load local-app:latest
```

### 4. Deploy Manifests to the Cluster
Apply your declarative Kubernetes configuration files to boot up the **2 active replicas (Pods)**:
```bash
kubectl apply -f deployment.yaml
```

Track the initialization, healthy deployment states, and pod cluster placements:
```bash
kubectl get deployments
kubectl get pods -o wide
```

### 5. Expose and Access the Service
Instruct Minikube to bridge network traffic and tunnel a routable endpoint to your application:
```bash
minikube service local-app-service
```

---

## 📝 Declarative Architecture Template (`deployment.yaml`)
The project utilizes the following production-grade configuration to handle lifecycle status management, container port forwarding, and node pod placement policies:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app-deployment
  labels:
    app: python-service
spec:
  replicas: 2 # Configured for zero-downtime rolling updates
  selector:
    matchLabels:
      app: python-service
  template:
    metadata:
      labels:
        app: python-service
    spec:
      containers:
      - name: app-container
        image: local-app:latest
        imagePullPolicy: Never # Forces K8s to utilize the preloaded minikube node cache
        ports:
        - containerPort: 5000
```

---

## 📊 Next-Level Engineering Roadmap
- [ ] **Multi-Stage Builds:** Convert the `Dockerfile` into a multi-stage compilation to keep the distribution layer free of development binaries.
- [ ] **Dynamic Environment Injection:** Transition environment-specific values from plain-text definitions into Kubernetes `ConfigMaps` and `Secrets`.
- [ ] **Health Probes:** Define precise `livenessProbe` and `readinessProbe` lifecycle endpoints to manage traffic routing intelligently.

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for details.
