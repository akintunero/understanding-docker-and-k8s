# Lab 07: StatefulSets and DaemonSets

## 🎯 Learning Objectives
- Understand StatefulSets for stateful applications
- Configure DaemonSets for node-specific workloads
- Manage persistent storage with StatefulSets
- Implement ordered deployment and scaling
- Learn best practices for stateful workloads

## ⏱️ Estimated Time: 80 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of deployments and persistent volumes

## 🚀 Lab Exercises

### Exercise 1: Understanding StatefulSets

1. **Check existing StatefulSets:**
   ```bash
   kubectl get statefulsets --all-namespaces
   kubectl get statefulsets
   ```

2. **Create a namespace for StatefulSet testing:**
   ```bash
   kubectl create namespace statefulset-demo
   kubectl config set-context --current --namespace=statefulset-demo
   ```

### Exercise 2: Creating a Basic StatefulSet

1. **Create a simple StatefulSet:**
   ```bash
   cat << EOF > basic-statefulset.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: web-statefulset
   spec:
     serviceName: web-service
     replicas: 3
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web
       spec:
         containers:
         - name: web
           image: nginx:alpine
           ports:
           - containerPort: 80
           env:
           - name: POD_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
           - name: POD_IP
             valueFrom:
               fieldRef:
                 fieldPath: status.podIP
   EOF
   
   kubectl apply -f basic-statefulset.yaml
   ```

2. **Create a headless service for the StatefulSet:**
   ```bash
   cat << EOF > web-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-service
   spec:
     clusterIP: None
     selector:
       app: web
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f web-service.yaml
   ```

3. **Check StatefulSet status:**
   ```bash
   kubectl get statefulset web-statefulset
   kubectl get pods -l app=web
   kubectl describe statefulset web-statefulset
   ```

### Exercise 3: StatefulSet with Persistent Storage

1. **Create a StatefulSet with persistent volumes:**
   ```bash
   cat << EOF > statefulset-with-pv.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: db-statefulset
   spec:
     serviceName: db-service
     replicas: 3
     selector:
       matchLabels:
         app: db
     template:
       metadata:
         labels:
           app: db
       spec:
         containers:
         - name: db
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Database $(hostname) starting..." && echo "Data from $(hostname)" > /data/db-data.txt && sleep 3600']
           ports:
           - containerPort: 8080
           volumeMounts:
           - name: data-volume
             mountPath: /data
     volumeClaimTemplates:
     - metadata:
         name: data-volume
       spec:
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 1Gi
   EOF
   
   kubectl apply -f statefulset-with-pv.yaml
   ```

2. **Create a headless service for the database StatefulSet:**
   ```bash
   cat << EOF > db-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: db-service
   spec:
     clusterIP: None
     selector:
       app: db
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
   EOF
   
   kubectl apply -f db-service.yaml
   ```

3. **Test the StatefulSet:**
   ```bash
   # Wait for StatefulSet to be ready
   kubectl rollout status statefulset db-statefulset
   
   # Check pods
   kubectl get pods -l app=db
   
   # Check persistent volume claims
   kubectl get pvc
   
   # Test data persistence
   kubectl exec db-statefulset-0 -- cat /data/db-data.txt
   kubectl exec db-statefulset-1 -- cat /data/db-data.txt
   kubectl exec db-statefulset-2 -- cat /data/db-data.txt
   ```

### Exercise 4: StatefulSet Scaling and Updates

1. **Scale the StatefulSet:**
   ```bash
   # Scale up
   kubectl scale statefulset db-statefulset --replicas=5
   
   # Check the scaling process
   kubectl get pods -l app=db -w
   
   # Scale down
   kubectl scale statefulset db-statefulset --replicas=2
   ```

2. **Update the StatefulSet:**
   ```bash
   # Update the image
   kubectl patch statefulset db-statefulset -p '{"spec":{"template":{"spec":{"containers":[{"name":"db","image":"busybox:1.36"}]}}}}'
   
   # Check the update process
   kubectl rollout status statefulset db-statefulset
   ```

3. **Test ordered deployment:**
   ```bash
   # Delete a pod to test ordered recreation
   kubectl delete pod db-statefulset-1
   
   # Watch the recreation process
   kubectl get pods -l app=db -w
   ```

### Exercise 5: Understanding DaemonSets

1. **Check existing DaemonSets:**
   ```bash
   kubectl get daemonsets --all-namespaces
   kubectl get daemonsets
   ```

2. **Create a namespace for DaemonSet testing:**
   ```bash
   kubectl create namespace daemonset-demo
   kubectl config set-context --current --namespace=daemonset-demo
   ```

### Exercise 6: Creating a Basic DaemonSet

1. **Create a simple DaemonSet:**
   ```bash
   cat << EOF > basic-daemonset.yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: monitoring-daemonset
   spec:
     selector:
       matchLabels:
         app: monitoring
     template:
       metadata:
         labels:
           app: monitoring
       spec:
         containers:
         - name: monitoring
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Monitoring agent on $(hostname)" && sleep 3600']
           env:
           - name: NODE_NAME
             valueFrom:
               fieldRef:
                 fieldPath: spec.nodeName
   EOF
   
   kubectl apply -f basic-daemonset.yaml
   ```

2. **Check DaemonSet status:**
   ```bash
   kubectl get daemonset monitoring-daemonset
   kubectl get pods -l app=monitoring
   kubectl describe daemonset monitoring-daemonset
   ```

### Exercise 7: DaemonSet with Node Selectors

1. **Create a DaemonSet with node selectors:**
   ```bash
   cat << EOF > node-selector-daemonset.yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: storage-daemonset
   spec:
     selector:
       matchLabels:
         app: storage
     template:
       metadata:
         labels:
           app: storage
       spec:
         nodeSelector:
           storage: ssd
         containers:
         - name: storage
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Storage agent on $(hostname)" && sleep 3600']
           volumeMounts:
           - name: host-storage
             mountPath: /host-storage
         volumes:
         - name: host-storage
           hostPath:
             path: /var/lib/storage
   EOF
   
   kubectl apply -f node-selector-daemonset.yaml
   ```

2. **Label nodes for the DaemonSet:**
   ```bash
   # Get node names
   kubectl get nodes
   
   # Label a node (replace with actual node name)
   kubectl label nodes $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') storage=ssd
   
   # Check the DaemonSet
   kubectl get pods -l app=storage
   ```

### Exercise 8: Advanced StatefulSet Features

1. **Create a StatefulSet with init containers:**
   ```bash
   cat << EOF > init-statefulset.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: app-statefulset
   spec:
     serviceName: app-service
     replicas: 3
     selector:
       matchLabels:
         app: app
     template:
       metadata:
         labels:
           app: app
       spec:
         initContainers:
         - name: init-db
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Initializing database for $(hostname)" && sleep 5']
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
           volumeMounts:
           - name: config-volume
             mountPath: /etc/app/config
         volumes:
         - name: config-volume
           configMap:
             name: app-config
   EOF
   
   # Create ConfigMap
   cat << EOF > app-config.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     app.conf: |
       server {
           listen 80;
           location / {
               return 200 "Hello from StatefulSet!";
           }
       }
   EOF
   
   kubectl apply -f app-config.yaml
   kubectl apply -f init-statefulset.yaml
   ```

2. **Create a service for the app StatefulSet:**
   ```bash
   cat << EOF > app-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: app-service
   spec:
     clusterIP: None
     selector:
       app: app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f app-service.yaml
   ```

### Exercise 9: DaemonSet with Host Networking

1. **Create a DaemonSet with host networking:**
   ```bash
   cat << EOF > host-network-daemonset.yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: network-daemonset
   spec:
     selector:
       matchLabels:
         app: network
     template:
       metadata:
         labels:
           app: network
       spec:
         hostNetwork: true
         containers:
         - name: network
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Network agent on $(hostname)" && netstat -tuln && sleep 3600']
           securityContext:
             privileged: true
   EOF
   
   kubectl apply -f host-network-daemonset.yaml
   ```

2. **Check the DaemonSet:**
   ```bash
   kubectl get pods -l app=network
   kubectl logs -l app=network
   ```

### Exercise 10: StatefulSet and DaemonSet Monitoring

1. **Check StatefulSet events:**
   ```bash
   kubectl get events --sort-by='.lastTimestamp' | grep -i statefulset
   kubectl describe statefulset db-statefulset
   ```

2. **Check DaemonSet events:**
   ```bash
   kubectl get events --sort-by='.lastTimestamp' | grep -i daemonset
   kubectl describe daemonset monitoring-daemonset
   ```

3. **Monitor pod status:**
   ```bash
   # StatefulSet pods
   kubectl get pods -l app=db -o wide
   kubectl get pods -l app=web -o wide
   
   # DaemonSet pods
   kubectl get pods -l app=monitoring -o wide
   kubectl get pods -l app=network -o wide
   ```

### Exercise 11: Troubleshooting StatefulSets and DaemonSets

1. **Common StatefulSet issues:**
   ```bash
   # Check StatefulSet status
   kubectl get statefulset -o wide
   
   # Check pod status
   kubectl get pods -l app=db
   
   # Check PVC status
   kubectl get pvc
   
   # Check events
   kubectl get events --sort-by='.lastTimestamp'
   ```

2. **Common DaemonSet issues:**
   ```bash
   # Check DaemonSet status
   kubectl get daemonset -o wide
   
   # Check node labels
   kubectl get nodes --show-labels
   
   # Check pod status
   kubectl get pods -l app=monitoring
   ```

3. **Debug commands:**
   ```bash
   # Check StatefulSet configuration
   kubectl get statefulset db-statefulset -o yaml
   
   # Check DaemonSet configuration
   kubectl get daemonset monitoring-daemonset -o yaml
   
   # Check pod logs
   kubectl logs db-statefulset-0
   kubectl logs -l app=monitoring
   ```

## 🧹 Cleanup

1. **Delete StatefulSets and DaemonSets:**
   ```bash
   kubectl delete -f basic-statefulset.yaml
   kubectl delete -f statefulset-with-pv.yaml
   kubectl delete -f init-statefulset.yaml
   kubectl delete -f basic-daemonset.yaml
   kubectl delete -f node-selector-daemonset.yaml
   kubectl delete -f host-network-daemonset.yaml
   ```

2. **Delete services:**
   ```bash
   kubectl delete -f web-service.yaml
   kubectl delete -f db-service.yaml
   kubectl delete -f app-service.yaml
   ```

3. **Delete ConfigMap:**
   ```bash
   kubectl delete -f app-config.yaml
   ```

4. **Delete namespaces:**
   ```bash
   kubectl delete namespace statefulset-demo
   kubectl delete namespace daemonset-demo
   kubectl config set-context --current --namespace=default
   ```

5. **Clean up files:**
   ```bash
   rm -f *.yaml
   ```

## 📚 Key Takeaways

- **StatefulSets**: Provide ordered deployment and stable network identities
- **DaemonSets**: Ensure all nodes run a copy of a pod
- **Persistent Storage**: StatefulSets can use volume claim templates
- **Ordered Operations**: StatefulSets deploy and scale in order
- **Node Selectors**: DaemonSets can target specific nodes
- **Host Networking**: DaemonSets can access host network and resources

## 🔍 Troubleshooting

### Common Issues:
1. **StatefulSet pods not ready**: Check PVC binding and pod events
2. **DaemonSet not scheduling**: Check node selectors and tolerations
3. **Storage issues**: Verify storage class and PVC status
4. **Network issues**: Check service configuration and DNS

### Debug Commands:
```bash
# Check StatefulSet status
kubectl get statefulset -o wide

# Check DaemonSet status
kubectl get daemonset -o wide

# Check pod status
kubectl get pods -l app=<label>

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check logs
kubectl logs <pod-name>
```

## 🎯 Next Steps

- Learn about Jobs and CronJobs for batch processing
- Explore resource management and autoscaling
- Study security and RBAC
- Practice with production StatefulSet and DaemonSet patterns 