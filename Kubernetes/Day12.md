
# Day 12: DaemonSets

Welcome to Day 12 of the Kubernetes 20-Day Learning Challenge! Today, we will explore **DaemonSets**, a Kubernetes resource used to ensure that specific Pods run on all (or selected) nodes in a cluster.

---

## **What is a DaemonSet?**

A **DaemonSet** is a Kubernetes workload resource that ensures a copy of a Pod is running on every node in the cluster. It is typically used for cluster-wide services that need to run on all nodes, such as:
- Logging agents (e.g., Fluentd).
- Monitoring agents (e.g., Prometheus Node Exporter).
- Network plugins or storage daemons.

### **Key Features of DaemonSets**
1. Ensures Pods are automatically added to new nodes when they join the cluster.
2. Removes Pods from nodes when they are removed from the cluster.
3. Can target specific nodes using node selectors or taints and tolerations.

---

## **When to Use DaemonSets**

DaemonSets are ideal for workloads that:
- Need to run on every node in the cluster (e.g., monitoring, logging).
- Require access to node-specific resources (e.g., host networking, storage).
- Need to be deployed only on specific nodes (e.g., GPU-enabled nodes).

For applications that don’t require node-wide deployment, use Deployments or StatefulSets instead.

---

## **Creating a DaemonSet**

Here’s an example YAML file for creating a DaemonSet with Fluentd as a logging agent:

### Example: Fluentd DaemonSet

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
  labels:
    app: fluentd
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
          image: fluent/fluentd:v1.14
          resources:
            limits:
              memory: "200Mi"
              cpu: "100m"
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
```

### Key Components of the YAML File:
1. **hostPath Volumes**: Mounts directories from the host node into the container.
2. **Pod Template**: Defines the Pod specification for each node.
3. **Selector**: Ensures Pods are managed by this DaemonSet.

---

## **Managing DaemonSets**

### Create the DaemonSet:

```
kubectl apply -f daemonset.yaml
```

### Check the DaemonSet and its Pods:

```
kubectl get daemonsets
kubectl get pods -l app=fluentd -o wide
```

### Delete the DaemonSet (removes all Pods):

```
kubectl delete daemonset fluentd-daemonset
```

---

## **Targeting Specific Nodes**

You can configure DaemonSets to run only on specific nodes using **node selectors** or **taints and tolerations**.

### Example: Using Node Selectors
Add the following to your Pod spec in the DaemonSet YAML file:

```
nodeSelector:
  disktype: ssd
```

This ensures that Pods are scheduled only on nodes labeled with `disktype=ssd`.

### Example: Using Taints and Tolerations
If certain nodes are tainted, you can add tolerations to your Pod spec to allow scheduling on those nodes:

```
tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

---

## **Activities**

### Activity #1: Create a Simple DaemonSet
1. Use the provided Fluentd YAML file to create a DaemonSet.
2. Verify that a Pod is running on each node using `kubectl get pods -o wide`.

### Activity #2: Target Specific Nodes (Optional)
1. Label one of your nodes with `disktype=ssd` using:

```
kubectl label node <node-name> disktype=ssd
```

2. Modify your DaemonSet YAML file to include a `nodeSelector` targeting `disktype=ssd`.
3. Verify that Pods are only scheduled on labeled nodes.

### Activity #3 (Optional): Test Scaling Nodes
1. Add a new node to your cluster (if possible).
2. Verify that the DaemonSet automatically schedules a Pod on the new node.

---

## **Best Practices**

- Use tolerations and node selectors to control where DaemonSet Pods are scheduled.
- Monitor resource usage of DaemonSet Pods to avoid overloading nodes.
- Use lightweight images for agents running as DaemonSets, especially in large clusters.
- Regularly review and clean up unused or outdated DaemonSets.

---

## **Additional References**

- Official Documentation on [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- Blog Post on *"Understanding Kubernetes DaemonSets"* [Link](https://www.cncf.io/blog/kubernetes-daemonsets/)
- Video Tutorial on *"Kubernetes DaemonSets Explained"* [YouTube Link](https://www.youtube.com/watch?v=4kPvHt0QK6s)
- Interactive Lab on [Katacoda Labs](https://www.katacoda.com/courses/kubernetes)

---
