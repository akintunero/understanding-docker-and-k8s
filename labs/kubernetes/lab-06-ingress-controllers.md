# Lab 06: Ingress Controllers

## 🎯 Learning Objectives
- Understand Ingress controllers and their purpose
- Configure Ingress resources for external access
- Set up SSL/TLS termination
- Implement path-based routing
- Learn Ingress best practices

## ⏱️ Estimated Time: 75 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of services and deployments

## 🚀 Lab Exercises

### Exercise 1: Installing and Configuring Ingress Controller

1. **Check if Ingress controller is installed:**
   ```bash
   kubectl get pods -n ingress-nginx
   kubectl get pods -n kube-system | grep ingress
   ```

2. **Install NGINX Ingress Controller (if not present):**
   ```bash
   # For Minikube
   minikube addons enable ingress
   
   # For kind or other clusters
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
   ```

3. **Wait for Ingress controller to be ready:**
   ```bash
   kubectl wait --namespace ingress-nginx \
     --for=condition=ready pod \
     --selector=app.kubernetes.io/component=controller \
     --timeout=120s
   ```

4. **Check Ingress controller status:**
   ```bash
   kubectl get pods -n ingress-nginx
   kubectl describe pod -n ingress-nginx -l app.kubernetes.io/component=controller
   ```

### Exercise 2: Creating Basic Ingress Resources

1. **Create sample applications:**
   ```bash
   # Create namespace for ingress testing
   kubectl create namespace ingress-test
   kubectl config set-context --current --namespace=ingress-test
   
   # Create deployment for app1
   cat << EOF > app1-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: app1
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: app1
     template:
       metadata:
         labels:
           app: app1
       spec:
         containers:
         - name: app1
           image: nginx:alpine
           ports:
           - containerPort: 80
           env:
           - name: POD_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
   EOF
   
   # Create deployment for app2
   cat << EOF > app2-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: app2
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: app2
     template:
       metadata:
         labels:
           app: app2
       spec:
         containers:
         - name: app2
           image: httpd:alpine
           ports:
           - containerPort: 80
           env:
           - name: POD_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
   EOF
   
   kubectl apply -f app1-deployment.yaml
   kubectl apply -f app2-deployment.yaml
   ```

2. **Create services for the applications:**
   ```bash
   cat << EOF > app1-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: app1-service
   spec:
     selector:
       app: app1
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     type: ClusterIP
   EOF
   
   cat << EOF > app2-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: app2-service
   spec:
     selector:
       app: app2
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     type: ClusterIP
   EOF
   
   kubectl apply -f app1-service.yaml
   kubectl apply -f app2-service.yaml
   ```

3. **Create a basic Ingress resource:**
   ```bash
   cat << EOF > basic-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: basic-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: app1.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
     - host: app2.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app2-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f basic-ingress.yaml
   ```

4. **Test the Ingress:**
   ```bash
   # Get the Ingress controller external IP
   kubectl get service -n ingress-nginx
   
   # For Minikube, get the IP
   minikube ip
   
   # Add entries to /etc/hosts (replace with actual IP)
   # echo "$(minikube ip) app1.local app2.local" | sudo tee -a /etc/hosts
   
   # Test the applications
   curl -H "Host: app1.local" http://$(minikube ip)
   curl -H "Host: app2.local" http://$(minikube ip)
   ```

### Exercise 3: Path-Based Routing

1. **Create an Ingress with path-based routing:**
   ```bash
   cat << EOF > path-based-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: path-based-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /$2
   spec:
     ingressClassName: nginx
     rules:
     - host: example.local
       http:
         paths:
         - path: /app1(/|$)(.*)
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
         - path: /app2(/|$)(.*)
           pathType: Prefix
           backend:
             service:
               name: app2-service
               port:
                 number: 80
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f path-based-ingress.yaml
   ```

2. **Test path-based routing:**
   ```bash
   # Test different paths
   curl -H "Host: example.local" http://$(minikube ip)/
   curl -H "Host: example.local" http://$(minikube ip)/app1/
   curl -H "Host: example.local" http://$(minikube ip)/app2/
   ```

### Exercise 4: SSL/TLS Configuration

1. **Generate SSL certificates:**
   ```bash
   # Generate private key
   openssl genrsa -out tls.key 2048
   
   # Generate certificate signing request
   openssl req -new -key tls.key -out tls.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=example.local"
   
   # Generate self-signed certificate
   openssl x509 -req -days 365 -in tls.csr -signkey tls.key -out tls.crt
   ```

2. **Create TLS Secret:**
   ```bash
   kubectl create secret tls example-tls \
     --key=tls.key \
     --cert=tls.crt
   ```

3. **Create Ingress with TLS:**
   ```bash
   cat << EOF > tls-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: tls-ingress
     annotations:
       nginx.ingress.kubernetes.io/ssl-redirect: "true"
   spec:
     ingressClassName: nginx
     tls:
     - hosts:
       - example.local
       secretName: example-tls
     rules:
     - host: example.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f tls-ingress.yaml
   ```

4. **Test TLS configuration:**
   ```bash
   # Test HTTPS (ignore certificate warnings)
   curl -k -H "Host: example.local" https://$(minikube ip)/
   
   # Test HTTP to HTTPS redirect
   curl -I -H "Host: example.local" http://$(minikube ip)/
   ```

### Exercise 5: Advanced Ingress Features

1. **Create Ingress with rate limiting:**
   ```bash
   cat << EOF > rate-limit-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: rate-limit-ingress
     annotations:
       nginx.ingress.kubernetes.io/rate-limit: "100"
       nginx.ingress.kubernetes.io/rate-limit-window: "1m"
   spec:
     ingressClassName: nginx
     rules:
     - host: ratelimit.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f rate-limit-ingress.yaml
   ```

2. **Create Ingress with CORS:**
   ```bash
   cat << EOF > cors-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: cors-ingress
     annotations:
       nginx.ingress.kubernetes.io/cors-allow-origin: "*"
       nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, OPTIONS"
       nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range"
   spec:
     ingressClassName: nginx
     rules:
     - host: cors.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f cors-ingress.yaml
   ```

3. **Create Ingress with custom headers:**
   ```bash
   cat << EOF > custom-headers-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: custom-headers-ingress
     annotations:
       nginx.ingress.kubernetes.io/configuration-snippet: |
         add_header X-Custom-Header "CustomValue" always;
         add_header X-Server "Kubernetes" always;
   spec:
     ingressClassName: nginx
     rules:
     - host: headers.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f custom-headers-ingress.yaml
   ```

### Exercise 6: Ingress with Multiple Backends

1. **Create a third application:**
   ```bash
   cat << EOF > app3-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: app3
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: app3
     template:
       metadata:
         labels:
           app: app3
       spec:
         containers:
         - name: app3
           image: nginx:alpine
           ports:
           - containerPort: 80
   EOF
   
   cat << EOF > app3-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: app3-service
   spec:
     selector:
       app: app3
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     type: ClusterIP
   EOF
   
   kubectl apply -f app3-deployment.yaml
   kubectl apply -f app3-service.yaml
   ```

2. **Create Ingress with multiple backends:**
   ```bash
   cat << EOF > multi-backend-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: multi-backend-ingress
     annotations:
       nginx.ingress.kubernetes.io/upstream-hash-by: "ip_hash"
   spec:
     ingressClassName: nginx
     rules:
     - host: multi.local
       http:
         paths:
         - path: /app1
           pathType: Prefix
           backend:
             service:
               name: app1-service
               port:
                 number: 80
         - path: /app2
           pathType: Prefix
           backend:
             service:
               name: app2-service
               port:
                 number: 80
         - path: /app3
           pathType: Prefix
           backend:
             service:
               name: app3-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f multi-backend-ingress.yaml
   ```

3. **Test multiple backends:**
   ```bash
   # Test different paths
   curl -H "Host: multi.local" http://$(minikube ip)/app1/
   curl -H "Host: multi.local" http://$(minikube ip)/app2/
   curl -H "Host: multi.local" http://$(minikube ip)/app3/
   ```

### Exercise 7: Ingress Monitoring and Troubleshooting

1. **Check Ingress status:**
   ```bash
   kubectl get ingress
   kubectl describe ingress basic-ingress
   kubectl describe ingress path-based-ingress
   kubectl describe ingress tls-ingress
   ```

2. **Check Ingress controller logs:**
   ```bash
   kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
   ```

3. **Test Ingress connectivity:**
   ```bash
   # Check if Ingress controller is accessible
   kubectl get service -n ingress-nginx
   
   # Test Ingress endpoints
   kubectl get endpoints -n ingress-nginx
   ```

4. **Monitor Ingress events:**
   ```bash
   kubectl get events --sort-by='.lastTimestamp' | grep -i ingress
   ```

## 🧹 Cleanup

1. **Delete all Ingress resources:**
   ```bash
   kubectl delete -f basic-ingress.yaml
   kubectl delete -f path-based-ingress.yaml
   kubectl delete -f tls-ingress.yaml
   kubectl delete -f rate-limit-ingress.yaml
   kubectl delete -f cors-ingress.yaml
   kubectl delete -f custom-headers-ingress.yaml
   kubectl delete -f multi-backend-ingress.yaml
   ```

2. **Delete deployments and services:**
   ```bash
   kubectl delete -f app1-deployment.yaml
   kubectl delete -f app2-deployment.yaml
   kubectl delete -f app3-deployment.yaml
   kubectl delete -f app1-service.yaml
   kubectl delete -f app2-service.yaml
   kubectl delete -f app3-service.yaml
   ```

3. **Delete TLS Secret:**
   ```bash
   kubectl delete secret example-tls
   ```

4. **Delete namespace:**
   ```bash
   kubectl delete namespace ingress-test
   kubectl config set-context --current --namespace=default
   ```

5. **Clean up files:**
   ```bash
   rm -f *.yaml tls.key tls.csr tls.crt
   ```

## 📚 Key Takeaways

- **Ingress Controllers**: Provide external access to cluster services
- **Path-Based Routing**: Route traffic based on URL paths
- **SSL/TLS Termination**: Handle HTTPS traffic at the Ingress level
- **Annotations**: Configure Ingress behavior and features
- **Multiple Backends**: Route to different services based on rules
- **Monitoring**: Monitor Ingress controller logs and events

## 🔍 Troubleshooting

### Common Issues:
1. **Ingress not accessible**: Check Ingress controller installation
2. **SSL certificate issues**: Verify TLS secret configuration
3. **Path routing not working**: Check path configuration and annotations
4. **Service not found**: Verify service names and selectors

### Debug Commands:
```bash
# Check Ingress status
kubectl get ingress -o wide

# Check Ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Check service endpoints
kubectl get endpoints

# Test Ingress connectivity
curl -H "Host: <hostname>" http://<ingress-ip>
```

## 🎯 Next Steps

- Learn about StatefulSets and DaemonSets
- Explore advanced Ingress features (canary deployments)
- Study service mesh (Istio) for advanced routing
- Practice with production Ingress configurations 