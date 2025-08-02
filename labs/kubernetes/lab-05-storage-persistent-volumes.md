# Lab 05: Storage and Persistent Volumes

## 🎯 Learning Objectives
- Understand Kubernetes storage concepts
- Create and manage Persistent Volumes (PV)
- Use Persistent Volume Claims (PVC)
- Configure Storage Classes
- Mount volumes in pods and deployments

## ⏱️ Estimated Time: 65 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of pods and deployments

## 🚀 Lab Exercises

### Exercise 1: Understanding Storage Types

1. **Check available storage classes:**
   ```bash
   kubectl get storageclass
   kubectl describe storageclass
   ```

2. **List existing persistent volumes:**
   ```bash
   kubectl get pv
   kubectl get pvc
   ```

3. **Check volume information:**
   ```bash
   kubectl get pv -o wide
   kubectl describe pv
   ```

### Exercise 2: Creating Persistent Volume Claims

1. **Create a basic PVC:**
   ```bash
   cat << EOF > basic-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: basic-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   EOF
   
   kubectl apply -f basic-pvc.yaml
   ```

2. **Create a PVC with specific storage class:**
   ```bash
   cat << EOF > storage-class-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: storage-class-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     storageClassName: standard
     resources:
       requests:
         storage: 2Gi
   EOF
   
   kubectl apply -f storage-class-pvc.yaml
   ```

3. **Create a PVC with multiple access modes:**
   ```bash
   cat << EOF > multi-access-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: multi-access-pvc
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 5Gi
   EOF
   
   kubectl apply -f multi-access-pvc.yaml
   ```

4. **Check PVC status:**
   ```bash
   kubectl get pvc
   kubectl describe pvc basic-pvc
   kubectl describe pvc storage-class-pvc
   kubectl describe pvc multi-access-pvc
   ```

### Exercise 3: Using PVCs in Pods

1. **Create a pod with PVC:**
   ```bash
   cat << EOF > pvc-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: pvc-pod
   spec:
     containers:
     - name: app
       image: nginx:alpine
       ports:
       - containerPort: 80
       volumeMounts:
       - name: data-volume
         mountPath: /data
     volumes:
     - name: data-volume
       persistentVolumeClaim:
         claimName: basic-pvc
   EOF
   
   kubectl apply -f pvc-pod.yaml
   ```

2. **Create a pod with multiple volumes:**
   ```bash
   cat << EOF > multi-volume-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: multi-volume-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'echo "Hello from $(hostname)" > /data1/hello.txt && echo "Data from $(hostname)" > /data2/data.txt && sleep 3600']
       volumeMounts:
       - name: data-volume-1
         mountPath: /data1
       - name: data-volume-2
         mountPath: /data2
     volumes:
     - name: data-volume-1
       persistentVolumeClaim:
         claimName: basic-pvc
     - name: data-volume-2
       persistentVolumeClaim:
         claimName: storage-class-pvc
   EOF
   
   kubectl apply -f multi-volume-pod.yaml
   ```

3. **Test data persistence:**
   ```bash
   # Wait for pod to be ready
   kubectl wait --for=condition=ready pod/pvc-pod
   kubectl wait --for=condition=ready pod/multi-volume-pod
   
   # Write data to the volume
   kubectl exec pvc-pod -- sh -c 'echo "Hello from $(hostname) at $(date)" > /data/persistent-data.txt'
   
   # Read the data
   kubectl exec pvc-pod -- cat /data/persistent-data.txt
   
   # Check data in multi-volume pod
   kubectl exec multi-volume-pod -- ls -la /data1/
   kubectl exec multi-volume-pod -- ls -la /data2/
   ```

### Exercise 4: Deployment with Persistent Storage

1. **Create a deployment with PVC:**
   ```bash
   cat << EOF > pvc-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: pvc-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: pvc-app
     template:
       metadata:
         labels:
           app: pvc-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
           volumeMounts:
           - name: data-volume
             mountPath: /usr/share/nginx/html
         volumes:
         - name: data-volume
           persistentVolumeClaim:
             claimName: multi-access-pvc
   EOF
   
   kubectl apply -f pvc-deployment.yaml
   ```

2. **Create a service for the deployment:**
   ```bash
   cat << EOF > pvc-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: pvc-service
   spec:
     selector:
       app: pvc-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     type: ClusterIP
   EOF
   
   kubectl apply -f pvc-service.yaml
   ```

3. **Test the deployment:**
   ```bash
   # Wait for deployment to be ready
   kubectl rollout status deployment pvc-deployment
   
   # Check pods
   kubectl get pods -l app=pvc-app
   
   # Test data persistence across pods
   kubectl exec -it $(kubectl get pods -l app=pvc-app -o jsonpath='{.items[0].metadata.name}') -- sh -c 'echo "Data from pod $(hostname)" > /usr/share/nginx/html/shared-data.txt'
   
   # Check data from another pod
   kubectl exec -it $(kubectl get pods -l app=pvc-app -o jsonpath='{.items[1].metadata.name}') -- cat /usr/share/nginx/html/shared-data.txt
   ```

### Exercise 5: Storage Classes and Dynamic Provisioning

1. **Create a custom storage class:**
   ```bash
   cat << EOF > custom-storage-class.yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: custom-storage
   provisioner: k8s.io/minikube-hostpath
   parameters:
     type: pd-ssd
   reclaimPolicy: Retain
   allowVolumeExpansion: true
   volumeBindingMode: Immediate
   EOF
   
   kubectl apply -f custom-storage-class.yaml
   ```

2. **Create PVC with custom storage class:**
   ```bash
   cat << EOF > custom-storage-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: custom-storage-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     storageClassName: custom-storage
     resources:
       requests:
         storage: 3Gi
   EOF
   
   kubectl apply -f custom-storage-pvc.yaml
   ```

3. **Check dynamic provisioning:**
   ```bash
   kubectl get pvc custom-storage-pvc
   kubectl get pv
   kubectl describe pvc custom-storage-pvc
   ```

### Exercise 6: Volume Expansion

1. **Create a PVC for expansion testing:**
   ```bash
   cat << EOF > expandable-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: expandable-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   EOF
   
   kubectl apply -f expandable-pvc.yaml
   ```

2. **Expand the PVC:**
   ```bash
   # Patch the PVC to increase storage
   kubectl patch pvc expandable-pvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
   
   # Check the PVC status
   kubectl get pvc expandable-pvc
   kubectl describe pvc expandable-pvc
   ```

3. **Create a pod to test the expanded volume:**
   ```bash
   cat << EOF > expandable-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: expandable-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'df -h /data && sleep 3600']
       volumeMounts:
       - name: data-volume
         mountPath: /data
     volumes:
     - name: data-volume
       persistentVolumeClaim:
         claimName: expandable-pvc
   EOF
   
   kubectl apply -f expandable-pod.yaml
   ```

### Exercise 7: StatefulSet with Persistent Storage

1. **Create a StatefulSet with persistent storage:**
   ```bash
   cat << EOF > statefulset-storage.yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: storage-statefulset
   spec:
     serviceName: storage-service
     replicas: 3
     selector:
       matchLabels:
         app: storage-app
     template:
       metadata:
         labels:
           app: storage-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
           volumeMounts:
           - name: data-volume
             mountPath: /usr/share/nginx/html
     volumeClaimTemplates:
     - metadata:
         name: data-volume
       spec:
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 1Gi
   EOF
   
   kubectl apply -f statefulset-storage.yaml
   ```

2. **Create a headless service for the StatefulSet:**
   ```bash
   cat << EOF > storage-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: storage-service
   spec:
     clusterIP: None
     selector:
       app: storage-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f storage-service.yaml
   ```

3. **Test the StatefulSet:**
   ```bash
   # Wait for StatefulSet to be ready
   kubectl rollout status statefulset storage-statefulset
   
   # Check StatefulSet pods
   kubectl get pods -l app=storage-app
   
   # Write data to each pod
   for i in {0..2}; do
     kubectl exec storage-statefulset-$i -- sh -c "echo 'Data from pod $i' > /usr/share/nginx/html/pod-$i-data.txt"
   done
   
   # Verify data persistence
   kubectl exec storage-statefulset-0 -- ls -la /usr/share/nginx/html/
   ```

### Exercise 8: Storage Monitoring and Troubleshooting

1. **Check storage usage:**
   ```bash
   # Get PVC details
   kubectl get pvc -o wide
   
   # Get PV details
   kubectl get pv -o wide
   
   # Check storage class
   kubectl get storageclass -o wide
   ```

2. **Monitor storage events:**
   ```bash
   # Check events related to storage
   kubectl get events --sort-by='.lastTimestamp' | grep -i volume
   kubectl get events --sort-by='.lastTimestamp' | grep -i pvc
   ```

3. **Troubleshoot storage issues:**
   ```bash
   # Check pod events
   kubectl describe pod pvc-pod
   
   # Check PVC status
   kubectl describe pvc basic-pvc
   
   # Check PV status
   kubectl describe pv
   ```

## 🧹 Cleanup

1. **Delete all resources:**
   ```bash
   kubectl delete -f pvc-pod.yaml
   kubectl delete -f multi-volume-pod.yaml
   kubectl delete -f pvc-deployment.yaml
   kubectl delete -f pvc-service.yaml
   kubectl delete -f expandable-pod.yaml
   kubectl delete -f statefulset-storage.yaml
   kubectl delete -f storage-service.yaml
   ```

2. **Delete PVCs:**
   ```bash
   kubectl delete pvc basic-pvc
   kubectl delete pvc storage-class-pvc
   kubectl delete pvc multi-access-pvc
   kubectl delete pvc custom-storage-pvc
   kubectl delete pvc expandable-pvc
   ```

3. **Delete Storage Class:**
   ```bash
   kubectl delete storageclass custom-storage
   ```

4. **Clean up files:**
   ```bash
   rm -f *.yaml
   ```

## 📚 Key Takeaways

- **Persistent Volumes**: Provide persistent storage for pods
- **Persistent Volume Claims**: Request storage resources
- **Storage Classes**: Define storage provisioners and parameters
- **Dynamic Provisioning**: Automatically create PVs based on PVCs
- **Volume Expansion**: Increase storage capacity without downtime
- **StatefulSets**: Use persistent storage with ordered deployment

## 🔍 Troubleshooting

### Common Issues:
1. **PVC pending**: Check storage class and provisioner
2. **Volume mount failed**: Verify PVC exists and is bound
3. **Storage expansion failed**: Check if storage class supports expansion
4. **Access denied**: Check access modes and permissions

### Debug Commands:
```bash
# Check PVC status
kubectl get pvc -o wide

# Check PV status
kubectl get pv -o wide

# Check storage class
kubectl get storageclass

# Check pod volume mounts
kubectl describe pod <pod-name>

# Check storage events
kubectl get events | grep -i volume
```

## 🎯 Next Steps

- Learn about Ingress controllers for external access
- Explore advanced storage features (snapshots, cloning)
- Study storage security and encryption
- Practice with cloud storage providers 