
# Day 17: Logging with ELK Stack

Welcome to Day 17 of the Kubernetes 20-Day Learning Challenge! Today, we will explore how to set up the **ELK Stack** (Elasticsearch, Logstash, and Kibana) on Kubernetes to centralize, store, and analyze logs from your cluster.

---

## **What is the ELK Stack?**

The **ELK Stack** is a popular open-source logging and observability platform that consists of:
1. **Elasticsearch**: A distributed search and analytics engine for storing and querying logs.
2. **Logstash**: A data processing pipeline for collecting, transforming, and forwarding logs.
3. **Kibana**: A visualization tool for exploring and analyzing log data stored in Elasticsearch.

### **Why Use the ELK Stack in Kubernetes?**
- Centralizes logs from all Pods, nodes, and applications in one place.
- Enables full-text search and filtering of logs for debugging.
- Provides powerful visualizations and dashboards to monitor application performance.
- Helps identify trends and anomalies in your cluster.

---

## **Setting Up the ELK Stack on Kubernetes**

We will use Helm to deploy Elasticsearch, Logstash, and Kibana in a Kubernetes cluster.

### Prerequisites:
- A running Kubernetes cluster with `kubectl` configured.
- Helm installed on your local machine.

---

### Step 1: Add the Elastic Helm Repository
Run the following commands to add the Elastic Helm repository:

```
helm repo add elastic https://helm.elastic.co
helm repo update
```

---

### Step 2: Create a Namespace for the ELK Stack
Create a dedicated namespace for the ELK stack:

```
kubectl create namespace elk
```

---

### Step 3: Deploy Elasticsearch
Install Elasticsearch using Helm:

```
helm install elasticsearch elastic/elasticsearch --namespace elk
```

Verify that Elasticsearch is running:

```
kubectl get pods -n elk
```

---

### Step 4: Deploy Logstash
Install Logstash using Helm:

```
helm install logstash elastic/logstash --namespace elk
```

Verify that Logstash is running:

```
kubectl get pods -n elk
```

---

### Step 5: Deploy Kibana
Install Kibana using Helm:

```
helm install kibana elastic/kibana --namespace elk
```

Verify that Kibana is running:

```
kubectl get pods -n elk
```

---

### Step 6: Access Kibana Dashboard
By default, Kibana is not exposed outside the cluster. Use port forwarding to access it locally:

```
kubectl port-forward -n elk svc/kibana-kibana 5601:5601
```

Open your browser and navigate to `http://localhost:5601`.

---

## **Sending Logs to the ELK Stack**

To collect logs from your Kubernetes cluster, you can use **Filebeat** or **Fluentd** as log collectors.

### Option 1: Configure Filebeat (Recommended)
Filebeat collects logs from Pods and nodes and forwards them to Elasticsearch.

1. Install Filebeat using Helm:

```
helm install filebeat elastic/filebeat --namespace elk
```

2. Verify that Filebeat is running:

```
kubectl get pods -n elk
```

3. Filebeat will automatically collect logs from `/var/log/containers` on each node and send them to Elasticsearch.

---

### Option 2: Configure Fluentd (Alternative)
Fluentd can also be used as a log collector. It runs as a DaemonSet on each node in your cluster.

1. Deploy Fluentd as a DaemonSet:

```
kubectl apply -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml
```

2. Verify that Fluentd is running on all nodes:

```
kubectl get pods -n kube-system -l k8s-app=fluentd-logging
```

3. Fluentd will forward logs to Elasticsearch for indexing.

---

## **Visualizing Logs in Kibana**

Once logs are flowing into Elasticsearch, you can create index patterns in Kibana to visualize them.

1. Open Kibana at `http://localhost:5601`.
2. Navigate to **Management > Index Patterns**.
3. Create an index pattern (e.g., `logstash-*`) to match your log indices.
4. Explore your logs using Kibana's Discover tab or create dashboards for visualization.

---

## **Activities**

### Activity #1: Deploy the ELK Stack
1. Use Helm to deploy Elasticsearch, Logstash, and Kibana in your cluster.
2. Verify that all components are running using `kubectl get pods -n elk`.

### Activity #2: Collect Logs with Filebeat or Fluentd
1. Install either Filebeat or Fluentd as a log collector.
2. Verify that logs are being forwarded to Elasticsearch by checking the indices in Kibana.

### Activity #3: Create Dashboards in Kibana (Optional)
1. Create an index pattern in Kibana for your log data.
2. Build visualizations such as bar charts or line graphs to analyze trends in your logs.

---

## **Best Practices**

- Use namespaces (`elk`, `logging`) to isolate logging components from other workloads.
- Limit resource usage by setting CPU/memory requests and limits for Elasticsearch and Logstash Pods.
- Enable role-based access control (RBAC) to restrict access to sensitive log data.
- Regularly monitor disk usage in Elasticsearch indices and set up retention policies to delete old logs.

---

## **Additional References**

- Official Documentation on [ELK Stack](https://www.elastic.co/what-is/elk-stack)
- Blog Post on *"Deploying ELK Stack on Kubernetes"* [Link](https://itsyndicate.org/blog/how-to-deploy-elk-stack-on-kubernetes-comprehensive-guide/)
- Video Tutorial on *"Kubernetes Logging with ELK"* [YouTube Link](https://www.youtube.com/watch?v=G7EIAgfkhmg)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
