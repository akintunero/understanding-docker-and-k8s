
# Day 16: Monitoring with Prometheus

Welcome to Day 16 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **Prometheus**, a powerful open-source monitoring and alerting toolkit, and learn how to set it up to monitor your Kubernetes cluster effectively.

---

## **What is Prometheus?**

Prometheus is an open-source monitoring system designed for collecting, storing, and querying metrics in a time-series database. It integrates seamlessly with Kubernetes to provide detailed insights into cluster performance and health.

### **Key Features of Prometheus**
1. **Pull-Based Metrics Collection**: Prometheus scrapes metrics from endpoints instead of relying on agents to push data.
2. **Service Discovery**: Automatically detects and monitors new services and Pods in Kubernetes.
3. **Time-Series Database**: Stores metrics in a highly efficient format for querying and analysis.
4. **Alerting**: Configures alert rules to notify you when certain conditions are met.
5. **Integration**: Works seamlessly with visualization tools like Grafana.

---

## **Why Use Prometheus for Kubernetes Monitoring?**

Prometheus allows you to:
- Monitor the health of your cluster components (e.g., nodes, Pods, Services).
- Track resource usage (CPU, memory, disk) over time.
- Set up alerts for critical events (e.g., high resource usage, Pod failures).
- Gain visibility into application performance.

---

## **Setting Up Prometheus**

### Prerequisites:
- A running Kubernetes cluster (e.g., Minikube, kind, or cloud-based cluster).
- `kubectl` installed and configured.
- Helm installed for easier deployment.

### Step 1: Install Prometheus Using Helm
1. Add the Prometheus Helm repository:
   ```
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```

2. Install Prometheus:
   ```
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
   ```

3. Verify that Prometheus is running:
   ```
   kubectl get pods -n monitoring
   kubectl get svc -n monitoring
   ```

### Step 2: Access the Prometheus Dashboard
1. Forward the Prometheus UI port to your local machine:
   ```
   kubectl port-forward -n monitoring svc/prometheus-server 9090:80
   ```

2. Open your browser and navigate to `http://localhost:9090`.

---

## **Visualizing Metrics with Grafana**

Grafana is a popular visualization tool that integrates with Prometheus to create dashboards for monitoring.

### Step 1: Install Grafana Using Helm
   
 ```
    helm install grafana grafana/grafana --namespace monitoring --set adminPassword=admin --create-namespace
 ```

### Step 2: Access the Grafana Dashboard
1. Forward the Grafana UI port to your local machine:
   ```
   kubectl port-forward -n monitoring svc/grafana 3000:80
   ```

2. Open your browser and navigate to `http://localhost:3000`. Log in using the default credentials (`admin`/`admin`).

3. Add Prometheus as a data source in Grafana and import pre-built Kubernetes dashboards from [Grafana Labs](https://grafana.com/grafana/dashboards/315-kubernetes-cluster-monitoring-via-prometheus/).

---

## **Monitoring Key Metrics**

Prometheus collects metrics from various Kubernetes components, such as:

1. **Node Metrics**:
    - CPU, memory, disk usage.
    - Example query:
      ```
      node_cpu_seconds_total{mode="idle"}
      ```

2. **Pod Metrics**:
    - Pod restarts, resource requests/limits.
    - Example query:
      ```
      rate(container_cpu_usage_seconds_total[5m])
      ```

3. **Cluster Health**:
    - Number of ready nodes, failed Pods.
    - Example query:
      ```
      kube_node_status_condition{condition="Ready", status="true"}
      ```

4. **Application Metrics**:
    - Custom application metrics exposed via `/metrics` endpoints.

---

## **Activities**

### Activity #1: Set Up Prometheus
1. Use Helm to install Prometheus in your cluster.
2. Verify that the Prometheus server is running by accessing its dashboard locally.

### Activity #2: Visualize Metrics with Grafana
1. Install Grafana using Helm and connect it to your Prometheus instance.
2. Import a pre-built Kubernetes dashboard from Grafana Labs.

### Activity #3 (Optional): Create Custom Alerts
1. Configure alerting rules in Prometheus using a YAML file.
2. Example alert rule (save as `alert-rules.yaml`):
   ```
   groups:
   - name: example-alerts
     rules:
     - alert: HighCPUUsage
       expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
       for: 5m
       labels:
         severity: warning
       annotations:
         summary: "High CPU usage detected"
         description: "CPU usage is above 80% for more than 5 minutes."
   ```
3. Apply the alert rules using `kubectl apply`.

---

## **Best Practices**

- Use `kube-state-metrics` to monitor Kubernetes-specific resources like Deployments and Pods.
- Set up Alertmanager with Prometheus for advanced alerting capabilities (e.g., email or Slack notifications).
- Regularly review and optimize scrape intervals and retention periods to balance performance and resource usage.
- Use dashboards in Grafana for real-time visualization of key metrics.

---

## **Additional References**

- [Kubernetes Monitoring with Prometheus](https://sysdig.com/blog/kubernetes-monitoring-prometheus/) by Sysdig.
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Dashboards for Kubernetes](https://grafana.com/grafana/dashboards/)
- Video Tutorial on *"Kubernetes Monitoring with Prometheus"* [YouTube Link](https://www.youtube.com/watch?v=rvgnR9ZzHWA)

---
