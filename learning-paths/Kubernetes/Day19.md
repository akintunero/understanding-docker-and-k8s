
# Day 19: Helm Package Manager

Welcome to Day 19 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Helm**, the Kubernetes package manager, and learn how to use it to simplify application deployment and management.

---

## **What is Helm?**

**Helm** is a Kubernetes package manager that helps you define, install, and manage applications in your cluster. It uses **Helm Charts**, which are pre-configured application templates, to deploy complex applications with minimal effort.

### **Key Features of Helm**
1. **Simplifies Application Deployment**: Automates the process of deploying and managing Kubernetes resources.
2. **Reusable Templates**: Use Helm Charts to deploy common applications (e.g., NGINX, Prometheus) without writing YAML files from scratch.
3. **Version Control**: Roll back to previous versions of an application easily.
4. **Customizable Deployments**: Override default configurations using values files or command-line arguments.

---

## **Helm Components**

1. **Helm CLI**:
    - The command-line tool used to interact with Helm.
    - Example commands: `helm install`, `helm upgrade`, `helm rollback`.

2. **Helm Chart**:
    - A collection of YAML files that define a Kubernetes application.
    - Includes templates for resources like Deployments, Services, ConfigMaps, etc.

3. **Chart Repository**:
    - A centralized location where Helm Charts are stored and shared.
    - Example repositories: [Artifact Hub](https://artifacthub.io/), Bitnami.

---

## **Installing Helm**

### Step 1: Install Helm CLI
Follow the official installation guide for your operating system:
- [Helm Installation Guide](https://helm.sh/docs/intro/install/)

Verify the installation:

```
helm version
```

---

## **Using Helm**

### Step 1: Add a Chart Repository
Add the official Bitnami repository as an example:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Step 2: Search for a Chart
Search for a specific chart (e.g., NGINX):

```
helm search repo nginx
```

### Step 3: Install a Chart
Install the NGINX chart from the Bitnami repository:

```
helm install my-nginx bitnami/nginx
```

### Step 4: Verify the Deployment
Check that the resources have been created:

```
kubectl get all
```

### Step 5: Uninstall a Release
To remove the deployed application:

```
helm uninstall my-nginx
```

---

## **Customizing Deployments**

You can customize Helm Charts by overriding default values using a `values.yaml` file or command-line arguments.

### Example: Override Values Using CLI

```
helm install my-nginx bitnami/nginx --set service.type=NodePort --set replicaCount=3
```

### Example: Override Values Using a File
Create a `custom-values.yaml` file:

```
service:
  type: NodePort
replicaCount: 3
```

Install the chart with custom values:

```
helm install my-nginx bitnami/nginx -f custom-values.yaml
```

---

## **Managing Releases**

1. **List Installed Releases**:

```
helm list
```

2. **Upgrade a Release**:

```
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

3. **Rollback to a Previous Version**:

```
helm rollback my-nginx <revision-number>
```

4. **View Release History**:

```
helm history my-nginx
```

---

## **Creating Your Own Helm Chart**

You can create your own Helm Chart for custom applications.

### Step 1: Create a New Chart

```
helm create my-chart
```

This generates a directory structure with default templates.

### Step 2: Customize Templates
Modify files in the `templates/` directory to define your application’s resources.

### Step 3: Install Your Chart
Deploy your custom chart to the cluster:

```
helm install my-app ./my-chart
```

---

## **Activities**

### Activity #1: Deploy an Application Using a Pre-Built Chart
1. Add the Bitnami repository and search for charts.
2. Install an application (e.g., WordPress or NGINX) using `helm install`.
3. Verify that the application is running in your cluster.

### Activity #2: Customize a Deployment (Optional)
1. Override default values using `--set` or a custom `values.yaml` file.
2. Scale the application by increasing replicas or changing resource limits.

### Activity #3: Create and Deploy a Custom Chart (Optional)
1. Use `helm create` to scaffold a new chart.
2. Define resources for a simple application (e.g., NGINX).
3. Deploy your custom chart using `helm install`.

---

## **Best Practices**

- Use versioned charts to ensure consistency across environments (e.g., dev, staging, production).
- Store custom charts in private repositories for internal use.
- Regularly update charts to include security patches and improvements.
- Use CI/CD pipelines to automate Helm deployments and upgrades.

---

## **Additional References**

- Official Documentation on [Helm](https://helm.sh/docs/)
- Blog Post on *"Getting Started with Helm"* [Link](https://www.cncf.io/blog/getting-started-with-helm/)
- Video Tutorial on *"Helm Package Manager Explained"* [YouTube Link](https://www.youtube.com/watch?v=ZzwqPO6l0PQ)
- Explore Pre-Built Charts on [Artifact Hub](https://artifacthub.io/)

---
