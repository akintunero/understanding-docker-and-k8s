# Lab 01: Your First Kubernetes Pod

## 🎯 Learning Objectives
- Create your first Kubernetes pod
- Understand pod lifecycle
- Practice basic kubectl commands
- Learn pod inspection and management

## ⏱️ Estimated Time: 45 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of containers

## 🚀 Lab Exercises

### Exercise 1: Create Your First Pod

1. **Check cluster status:**
   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

2. **Create a simple pod using YAML:**
   ```bash
   # Create a pod definition file
   cat << EOF > first-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-pod
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:alpine
       ports:
       - containerPort: 80
   EOF
   ```

3. **Apply the pod definition:**
   ```bash
   kubectl apply -f first-pod.yaml
   ```

4. **Check pod status:**
   ```bash
   kubectl get pods
   kubectl get pods -o wide
   ```

### Exercise 2: Pod Lifecycle Management

1. **Describe the pod:**
   ```bash
   kubectl describe pod nginx-pod
   ```

2. **Get pod logs:**
   ```bash
   kubectl logs nginx-pod
   ```

3. **Execute commands in the pod:**
   ```bash
   # Get a shell in the pod
   kubectl exec -it nginx-pod -- /bin/sh
   
   # Inside the pod, try these commands:
   # hostname
   # whoami
   # ps aux
   # exit
   ```

4. **Copy files to/from pod:**
   ```bash
   # Create a test file locally
   echo "Hello from host" > test.txt
   
   # Copy to pod
   kubectl cp test.txt nginx-pod:/tmp/
   
   # Copy from pod
   kubectl cp nginx-pod:/etc/nginx/nginx.conf ./nginx.conf
   ```

### Exercise 3: Pod Networking

1. **Port forward to access the pod:**
   ```bash
   kubectl port-forward nginx-pod 8080:80
   ```

2. **Test the connection:**
   ```bash
   # In another terminal
   curl http://localhost:8080
   ```

3. **Stop port forwarding:**
   ```bash
   # Press Ctrl+C in the port-forward terminal
   ```

### Exercise 4: Pod Resource Management

1. **Create a pod with resource limits:**
   ```bash
   cat << EOF > resource-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: resource-pod
   spec:
     containers:
     - name: stress
       image: busybox
       command: ["sh", "-c", "while true; do echo 'Hello World'; sleep 10; done"]
       resources:
         requests:
           memory: "64Mi"
           cpu: "250m"
         limits:
           memory: "128Mi"
           cpu: "500m"
   EOF
   ```

2. **Apply the resource pod:**
   ```bash
   kubectl apply -f resource-pod.yaml
   ```

3. **Monitor resource usage:**
   ```bash
   kubectl top pods
   kubectl describe pod resource-pod
   ```

### Exercise 5: Multi-Container Pod

1. **Create a pod with multiple containers:**
   ```bash
   cat << EOF > multi-container-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: multi-container-pod
   spec:
     containers:
     - name: nginx
       image: nginx:alpine
       ports:
       - containerPort: 80
     - name: sidecar
       image: busybox
       command: ["sh", "-c", "while true; do echo 'Sidecar running'; sleep 30; done"]
   EOF
   ```

2. **Apply the multi-container pod:**
   ```bash
   kubectl apply -f multi-container-pod.yaml
   ```

3. **Interact with specific containers:**
   ```bash
   # Get logs from nginx container
   kubectl logs multi-container-pod -c nginx
   
   # Get logs from sidecar container
   kubectl logs multi-container-pod -c sidecar
   
   # Execute command in nginx container
   kubectl exec -it multi-container-pod -c nginx -- /bin/sh
   
   # Execute command in sidecar container
   kubectl exec -it multi-container-pod -c sidecar -- /bin/sh
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Pod Health Checks

1. **Create a pod with health checks:**
   ```bash
   cat << EOF > health-check-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: health-check-pod
   spec:
     containers:
     - name: nginx
       image: nginx:alpine
       ports:
       - containerPort: 80
       livenessProbe:
         httpGet:
           path: /
           port: 80
         initialDelaySeconds: 30
         periodSeconds: 10
       readinessProbe:
         httpGet:
           path: /
           port: 80
         initialDelaySeconds: 5
         periodSeconds: 5
   EOF
   ```

2. **Apply the health check pod:**
   ```bash
   kubectl apply -f health-check-pod.yaml
   ```

3. **Monitor the pod:**
   ```bash
   kubectl get pods health-check-pod -w
   kubectl describe pod health-check-pod
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create a basic pod?
- [ ] Do you understand pod lifecycle?
- [ ] Can you manage pods (create, delete, describe)?
- [ ] Can you access pod logs and execute commands?
- [ ] Do you understand pod networking?
- [ ] Can you configure resource limits?

### Skills Demonstrated:
- ✅ Basic pod operations
- ✅ Pod lifecycle management
- ✅ Pod inspection and debugging
- ✅ Resource monitoring
- ✅ Multi-container pods
- ✅ Health checks

## 🔍 Troubleshooting

### Common Issues:

1. **Pod stuck in Pending:**
   ```bash
   # Check pod events
   kubectl describe pod <pod-name>
   
   # Check node resources
   kubectl describe nodes
   ```

2. **Pod stuck in CrashLoopBackOff:**
   ```bash
   # Check pod logs
   kubectl logs <pod-name>
   
   # Check previous container logs
   kubectl logs <pod-name> --previous
   ```

3. **Cannot connect to pod:**
   ```bash
   # Check pod status
   kubectl get pods
   
   # Check pod IP
   kubectl get pods -o wide
   
   # Test connectivity
   kubectl exec <pod-name> -- ping -c 3 google.com
   ```

## 📚 Additional Resources

- [Kubernetes Pods Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)
- [kubectl Command Reference](https://kubernetes.io/docs/reference/kubectl/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

## 🎉 Lab Completion

Congratulations! You've completed your first Kubernetes lab. You now understand:
- How to create and manage pods
- Pod lifecycle and states
- Basic kubectl commands
- Pod networking and resource management

**Next Lab**: [Lab 02: Deployments](../kubernetes/lab-02-deployments.md)

---

**Happy Learning! ☸️** 