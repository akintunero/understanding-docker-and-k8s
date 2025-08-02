# Day 2: Kubernetes Architecture

Welcome to Day 2 of the Kubernetes 20-Day Learning Challenge! Today, we will explore the architecture of Kubernetes and understand how its components work together to manage containerized applications.

---

## **Kubernetes Architecture Overview**

Kubernetes follows a client-server architecture, consisting of a **Control Plane** and **Worker Nodes**. Below is a high-level overview of its key components:

### **1. Control Plane (Master Node)**
The control plane manages the Kubernetes cluster and ensures the desired state of the system.

- **API Server**:
    - Acts as the front-end for the Kubernetes control plane.
    - All communication with the cluster happens through the API server.
    - Example command: `kubectl` interacts with this component.

- **etcd**:
    - A distributed key-value store that holds all cluster data.
    - Stores information about nodes, Pods, ConfigMaps, Secrets, and more.

- **Controller Manager**:
    - Runs controllers that handle routine tasks (e.g., ensuring desired replicas, managing endpoints).
    - Examples: Deployment Controller, Node Controller.

- **Scheduler**:
    - Assigns Pods to nodes based on resource availability and constraints.
    - Ensures optimal placement of workloads.

### **2. Worker Nodes**
Worker nodes run application workloads in Pods. Each node has the following components:

- **kubelet**:
    - An agent that runs on each worker node.
    - Ensures containers are running in Pods as specified by the control plane.

- **kube-proxy**:
    - Manages networking for Pods on each node.
    - Handles routing and load balancing within the cluster.

- **Container Runtime**:
    - Responsible for running containers (e.g., Docker, containerd, CRI-O).

---

## **How Kubernetes Works**

1. The user interacts with Kubernetes through `kubectl` or other tools by sending requests to the API Server.
2. The API Server validates and processes these requests, storing information in `etcd`.
3. The Scheduler assigns workloads (Pods) to nodes based on available resources.
4. The kubelet on each worker node ensures that containers are running as defined in the Pod specification.
5. kube-proxy handles network traffic to ensure Pods can communicate with each other and external clients.

---

## **Activities**

### 1. **Visualize Kubernetes Architecture**
- Create a diagram of Kubernetes architecture showing Control Plane components and Worker Node components.
- Use tools like [draw.io](https://app.diagrams.net/) or draw it on paper for better understanding.

### 2. **Explore Your Cluster**
- If you set up Minikube or kind on Day 1:
    - Run `kubectl cluster-info` to view your cluster details.
    - Use `kubectl get nodes` to list all nodes in your cluster.
- Inspect node details using:
  ```
  kubectl describe node <node-name>
  ```

### 3. **Learn About Components**
- Read about each component in detail from the official documentation:
    - [Control Plane Components](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components)
    - [Node Components](https://kubernetes.io/docs/concepts/overview/components/#node-components)

---

## **Additional References**

- Official Documentation: [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- Blog Post: *"Kubernetes Architecture Explained"* by CNCF
- Video Tutorial: *"Deep Dive into Kubernetes Architecture"* by Google Cloud [YouTube Link](https://www.youtube.com/watch?v=pnP6eYh6LkA)
- Interactive Lab: Explore architecture interactively using Katacoda: [Katacoda Kubernetes Labs](https://www.katacoda.com/courses/kubernetes)

---