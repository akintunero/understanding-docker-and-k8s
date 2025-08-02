# Lab 03: Services and Networking

## 🎯 Learning Objectives
- Understand Kubernetes service types
- Configure service discovery
- Implement load balancing
- Set up network policies
- Learn service communication patterns

## ⏱️ Estimated Time: 70 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of pods and deployments

## 🚀 Lab Exercises

### Exercise 1: Understanding Service Types

1. **Create a deployment for testing:**
   ```bash
   cat << EOF > web-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web-app
     labels:
       app: web-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: web-app
     template:
       metadata:
         labels:
           app: web-app
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
   
   kubectl apply -f web-deployment.yaml
   ```

2. **Create different service types:**
   ```bash
   # ClusterIP Service (default)
   cat << EOF > clusterip-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-clusterip
   spec:
     type: ClusterIP
     selector:
       app: web-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   # NodePort Service
   cat << EOF > nodeport-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-nodeport
   spec:
     type: NodePort
     selector:
       app: web-app
     ports:
     - port: 80
       targetPort: 80
       nodePort: 30080
       protocol: TCP
   EOF
   
   # LoadBalancer Service
   cat << EOF > loadbalancer-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-loadbalancer
   spec:
     type: LoadBalancer
     selector:
       app: web-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f clusterip-service.yaml
   kubectl apply -f nodeport-service.yaml
   kubectl apply -f loadbalancer-service.yaml
   ```

3. **Inspect the services:**
   ```bash
   kubectl get services
   kubectl describe service web-clusterip
   kubectl describe service web-nodeport
   kubectl describe service web-loadbalancer
   ```

### Exercise 2: Service Discovery and Communication

1. **Create a client pod to test service discovery:**
   ```bash
   cat << EOF > client-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: client-pod
   spec:
     containers:
     - name: client
       image: busybox:1.35
       command: ['sh', '-c', 'while true; do sleep 30; done']
   EOF
   
   kubectl apply -f client-pod.yaml
   ```

2. **Test service discovery:**
   ```bash
   # Test DNS resolution
   kubectl exec client-pod -- nslookup web-clusterip
   kubectl exec client-pod -- nslookup web-clusterip.default.svc.cluster.local
   
   # Test HTTP connectivity
   kubectl exec client-pod -- wget -qO- http://web-clusterip
   kubectl exec client-pod -- wget -qO- http://web-clusterip.default.svc.cluster.local
   ```

3. **Test load balancing:**
   ```bash
   # Make multiple requests to see load balancing
   for i in {1..10}; do
     kubectl exec client-pod -- wget -qO- http://web-clusterip
     echo "Request $i completed"
   done
   ```

### Exercise 3: Advanced Service Configuration

1. **Create a service with multiple ports:**
   ```bash
   cat << EOF > multi-port-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-multi-port
   spec:
     selector:
       app: web-app
     ports:
     - name: http
       port: 80
       targetPort: 80
       protocol: TCP
     - name: https
       port: 443
       targetPort: 443
       protocol: TCP
   EOF
   
   kubectl apply -f multi-port-service.yaml
   ```

2. **Create a service without selectors (ExternalName):**
   ```bash
   cat << EOF > external-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: external-api
   spec:
     type: ExternalName
     externalName: api.github.com
     ports:
     - port: 443
       targetPort: 443
       protocol: TCP
   EOF
   
   kubectl apply -f external-service.yaml
   
   # Test external service
   kubectl exec client-pod -- nslookup external-api
   ```

3. **Create a service with session affinity:**
   ```bash
   cat << EOF > session-affinity-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-session-affinity
   spec:
     selector:
       app: web-app
     sessionAffinity: ClientIP
     sessionAffinityConfig:
       clientIP:
         timeoutSeconds: 10800
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f session-affinity-service.yaml
   ```

### Exercise 4: Network Policies

1. **Create a namespace for network policy testing:**
   ```bash
   kubectl create namespace network-test
   kubectl config set-context --current --namespace=network-test
   ```

2. **Create pods in the namespace:**
   ```bash
   cat << EOF > network-test-pods.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: frontend-pod
     labels:
       app: frontend
   spec:
     containers:
     - name: frontend
       image: nginx:alpine
       ports:
       - containerPort: 80
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: backend-pod
     labels:
       app: backend
   spec:
     containers:
     - name: backend
       image: nginx:alpine
       ports:
       - containerPort: 80
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: database-pod
     labels:
       app: database
   spec:
     containers:
     - name: database
       image: nginx:alpine
       ports:
       - containerPort: 80
   EOF
   
   kubectl apply -f network-test-pods.yaml
   ```

3. **Create network policies:**
   ```bash
   # Allow frontend to backend communication
   cat << EOF > frontend-to-backend-policy.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: frontend-to-backend
   spec:
     podSelector:
       matchLabels:
         app: frontend
     policyTypes:
     - Egress
     egress:
     - to:
       - podSelector:
           matchLabels:
             app: backend
       ports:
       - protocol: TCP
         port: 80
   EOF
   
   # Allow backend to database communication
   cat << EOF > backend-to-database-policy.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: backend-to-database
   spec:
     podSelector:
       matchLabels:
         app: backend
     policyTypes:
     - Egress
     egress:
     - to:
       - podSelector:
           matchLabels:
             app: database
       ports:
       - protocol: TCP
         port: 80
   ---
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: database-ingress
   spec:
     podSelector:
       matchLabels:
         app: database
     policyTypes:
     - Ingress
     ingress:
     - from:
       - podSelector:
           matchLabels:
             app: backend
       ports:
       - protocol: TCP
         port: 80
   EOF
   
   kubectl apply -f frontend-to-backend-policy.yaml
   kubectl apply -f backend-to-database-policy.yaml
   ```

4. **Test network policies:**
   ```bash
   # Test frontend to backend communication
   kubectl exec frontend-pod -- wget -qO- http://backend-pod
   
   # Test backend to database communication
   kubectl exec backend-pod -- wget -qO- http://database-pod
   
   # Test frontend to database communication (should fail)
   kubectl exec frontend-pod -- wget -qO- http://database-pod
   ```

### Exercise 5: Service with Headless Service

1. **Create a headless service:**
   ```bash
   cat << EOF > headless-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-headless
   spec:
     clusterIP: None
     selector:
       app: web-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f headless-service.yaml
   ```

2. **Test headless service:**
   ```bash
   # Get all pods directly
   kubectl exec client-pod -- nslookup web-headless
   
   # Test individual pod access
   kubectl exec client-pod -- wget -qO- http://web-app-0.web-headless
   ```

### Exercise 6: Service Endpoints

1. **Create a service without selectors:**
   ```bash
   cat << EOF > manual-endpoint-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: manual-endpoint-service
   spec:
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   ---
   apiVersion: v1
   kind: Endpoints
   metadata:
     name: manual-endpoint-service
   subsets:
   - addresses:
     - ip: 10.0.0.1
     ports:
     - port: 80
       protocol: TCP
   EOF
   
   kubectl apply -f manual-endpoint-service.yaml
   ```

2. **Inspect endpoints:**
   ```bash
   kubectl get endpoints
   kubectl describe endpoints manual-endpoint-service
   ```

### Exercise 7: Service Load Balancing Testing

1. **Create a test application that shows pod information:**
   ```bash
   cat << EOF > pod-info-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: pod-info
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: pod-info
     template:
       metadata:
         labels:
           app: pod-info
       spec:
         containers:
         - name: pod-info
           image: nginx:alpine
           command: ["/bin/sh", "-c"]
           args:
           - |
             echo "Pod: $POD_NAME, IP: $POD_IP" > /usr/share/nginx/html/index.html
             nginx -g "daemon off;"
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
   
   kubectl apply -f pod-info-deployment.yaml
   ```

2. **Create a service for the pod-info deployment:**
   ```bash
   cat << EOF > pod-info-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: pod-info-service
   spec:
     selector:
       app: pod-info
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
   EOF
   
   kubectl apply -f pod-info-service.yaml
   ```

3. **Test load balancing:**
   ```bash
   # Make multiple requests to see load balancing
   for i in {1..10}; do
     kubectl exec client-pod -- wget -qO- http://pod-info-service
     echo "Request $i completed"
   done
   ```

## 🧹 Cleanup

1. **Delete all resources:**
   ```bash
   kubectl delete -f web-deployment.yaml
   kubectl delete -f clusterip-service.yaml
   kubectl delete -f nodeport-service.yaml
   kubectl delete -f loadbalancer-service.yaml
   kubectl delete -f client-pod.yaml
   kubectl delete -f multi-port-service.yaml
   kubectl delete -f external-service.yaml
   kubectl delete -f session-affinity-service.yaml
   kubectl delete -f network-test-pods.yaml
   kubectl delete -f frontend-to-backend-policy.yaml
   kubectl delete -f backend-to-database-policy.yaml
   kubectl delete -f headless-service.yaml
   kubectl delete -f manual-endpoint-service.yaml
   kubectl delete -f pod-info-deployment.yaml
   kubectl delete -f pod-info-service.yaml
   ```

2. **Delete namespace:**
   ```bash
   kubectl delete namespace network-test
   kubectl config set-context --current --namespace=default
   ```

3. **Clean up files:**
   ```bash
   rm -f *.yaml
   ```

## 📚 Key Takeaways

- **Service Types**: ClusterIP (internal), NodePort (external access), LoadBalancer (cloud)
- **Service Discovery**: DNS-based service discovery within the cluster
- **Load Balancing**: Automatic load balancing across pod endpoints
- **Network Policies**: Control pod-to-pod communication
- **Headless Services**: Direct pod access without load balancing
- **Session Affinity**: Sticky sessions for stateful applications

## 🔍 Troubleshooting

### Common Issues:
1. **Service not accessible**: Check selector labels and pod status
2. **DNS resolution fails**: Verify service and endpoint configuration
3. **Load balancing not working**: Check pod readiness and service endpoints
4. **Network policies blocking traffic**: Review policy rules and pod labels

### Debug Commands:
```bash
# Check service endpoints
kubectl get endpoints

# Check service DNS
kubectl exec <pod> -- nslookup <service>

# Check network policies
kubectl get networkpolicies

# Check service configuration
kubectl describe service <service-name>
```

## 🎯 Next Steps

- Learn about Ingress controllers for external access
- Explore service mesh (Istio/Linkerd) for advanced networking
- Study network policies for security
- Practice with multi-cluster service discovery 