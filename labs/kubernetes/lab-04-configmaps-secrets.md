# Lab 04: ConfigMaps and Secrets

## 🎯 Learning Objectives
- Create and manage ConfigMaps
- Handle sensitive data with Secrets
- Mount configurations in pods
- Use environment variables and volumes
- Implement secure configuration practices

## ⏱️ Estimated Time: 55 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of pods and deployments

## 🚀 Lab Exercises

### Exercise 1: Creating and Using ConfigMaps

1. **Create a ConfigMap from literal values:**
   ```bash
   kubectl create configmap app-config \
     --from-literal=APP_NAME=my-application \
     --from-literal=APP_VERSION=1.0.0 \
     --from-literal=ENVIRONMENT=development \
     --from-literal=LOG_LEVEL=info
   ```

2. **Create a ConfigMap from a file:**
   ```bash
   # Create a configuration file
   cat << EOF > app.properties
   database.host=postgres-service
   database.port=5432
   database.name=myapp
   redis.host=redis-service
   redis.port=6379
   api.timeout=30
   api.retries=3
   EOF
   
   kubectl create configmap app-properties --from-file=app.properties
   ```

3. **Create a ConfigMap from a directory:**
   ```bash
   # Create a directory with configuration files
   mkdir -p config
   
   cat << EOF > config/database.yaml
   host: postgres-service
   port: 5432
   name: myapp
   user: appuser
   EOF
   
   cat << EOF > config/redis.yaml
   host: redis-service
   port: 6379
   password: ""
   EOF
   
   cat << EOF > config/api.yaml
   timeout: 30
   retries: 3
   rate_limit: 100
   EOF
   
   kubectl create configmap app-config-files --from-file=config/
   ```

4. **Inspect the ConfigMaps:**
   ```bash
   kubectl get configmaps
   kubectl describe configmap app-config
   kubectl describe configmap app-properties
   kubectl describe configmap app-config-files
   
   # Get ConfigMap data
   kubectl get configmap app-config -o yaml
   kubectl get configmap app-properties -o yaml
   ```

### Exercise 2: Using ConfigMaps in Pods

1. **Create a pod that uses ConfigMap as environment variables:**
   ```bash
   cat << EOF > configmap-env-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: configmap-env-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'echo "App: $APP_NAME, Version: $APP_VERSION, Env: $ENVIRONMENT, Log: $LOG_LEVEL" && sleep 3600']
       env:
       - name: APP_NAME
         valueFrom:
           configMapKeyRef:
             name: app-config
             key: APP_NAME
       - name: APP_VERSION
         valueFrom:
           configMapKeyRef:
             name: app-config
             key: APP_VERSION
       - name: ENVIRONMENT
         valueFrom:
           configMapKeyRef:
             name: app-config
             key: ENVIRONMENT
       - name: LOG_LEVEL
         valueFrom:
           configMapKeyRef:
             name: app-config
             key: LOG_LEVEL
   EOF
   
   kubectl apply -f configmap-env-pod.yaml
   ```

2. **Create a pod that mounts ConfigMap as a volume:**
   ```bash
   cat << EOF > configmap-volume-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: configmap-volume-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'ls -la /config && cat /config/app.properties && sleep 3600']
       volumeMounts:
       - name: config-volume
         mountPath: /config
     volumes:
     - name: config-volume
       configMap:
         name: app-properties
   EOF
   
   kubectl apply -f configmap-volume-pod.yaml
   ```

3. **Create a pod that mounts specific ConfigMap keys:**
   ```bash
   cat << EOF > configmap-specific-keys-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: configmap-specific-keys-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'ls -la /config && cat /config/database.yaml && sleep 3600']
       volumeMounts:
       - name: config-volume
         mountPath: /config
     volumes:
     - name: config-volume
       configMap:
         name: app-config-files
         items:
         - key: database.yaml
           path: database.yaml
         - key: redis.yaml
           path: redis.yaml
   EOF
   
   kubectl apply -f configmap-specific-keys-pod.yaml
   ```

4. **Check the pods:**
   ```bash
   kubectl get pods
   kubectl logs configmap-env-pod
   kubectl logs configmap-volume-pod
   kubectl logs configmap-specific-keys-pod
   ```

### Exercise 3: Creating and Using Secrets

1. **Create a Secret from literal values:**
   ```bash
   kubectl create secret generic app-secrets \
     --from-literal=DB_PASSWORD=secret123 \
     --from-literal=API_KEY=api-key-12345 \
     --from-literal=JWT_SECRET=jwt-secret-key
   ```

2. **Create a Secret from a file:**
   ```bash
   # Create a private key file
   cat << EOF > private.key
   -----BEGIN PRIVATE KEY-----
   MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7VJTUt9Us8cKj
   MzEfYyjiWA4R4/M2bS1GB4t7NXp98C3SC6dVMvDuictGeurT8jNbvJZHtCSuYEvu
   NMoSfm76oqFvAp8Gy0iz5sxjZmSnXyCdPEovGhLa0VzMaQ8s+CLOyS56YyCFGeJZ
   -----END PRIVATE KEY-----
   EOF
   
   kubectl create secret generic app-private-key --from-file=private.key
   ```

3. **Create a Secret from a TLS certificate:**
   ```bash
   # Generate a self-signed certificate
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout tls.key -out tls.crt \
     -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"
   
   kubectl create secret tls app-tls \
     --key=tls.key \
     --cert=tls.crt
   ```

4. **Inspect the Secrets:**
   ```bash
   kubectl get secrets
   kubectl describe secret app-secrets
   kubectl describe secret app-private-key
   kubectl describe secret app-tls
   
   # Get Secret data (base64 encoded)
   kubectl get secret app-secrets -o yaml
   ```

### Exercise 4: Using Secrets in Pods

1. **Create a pod that uses Secret as environment variables:**
   ```bash
   cat << EOF > secret-env-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: secret-env-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'echo "DB Password: $DB_PASSWORD, API Key: $API_KEY" && sleep 3600']
       env:
       - name: DB_PASSWORD
         valueFrom:
           secretKeyRef:
             name: app-secrets
             key: DB_PASSWORD
       - name: API_KEY
         valueFrom:
           secretKeyRef:
             name: app-secrets
             key: API_KEY
   EOF
   
   kubectl apply -f secret-env-pod.yaml
   ```

2. **Create a pod that mounts Secret as a volume:**
   ```bash
   cat << EOF > secret-volume-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: secret-volume-pod
   spec:
     containers:
     - name: app
       image: busybox:1.35
       command: ['sh', '-c', 'ls -la /secrets && cat /secrets/private.key && sleep 3600']
       volumeMounts:
       - name: secret-volume
         mountPath: /secrets
         readOnly: true
     volumes:
     - name: secret-volume
       secret:
         secretName: app-private-key
   EOF
   
   kubectl apply -f secret-volume-pod.yaml
   ```

3. **Check the pods:**
   ```bash
   kubectl get pods
   kubectl logs secret-env-pod
   kubectl logs secret-volume-pod
   ```

### Exercise 5: Advanced Configuration Management

1. **Create a deployment with ConfigMap and Secret:**
   ```bash
   cat << EOF > app-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: config-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: config-app
     template:
       metadata:
         labels:
           app: config-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           ports:
           - containerPort: 80
           env:
           - name: APP_NAME
             valueFrom:
               configMapKeyRef:
                 name: app-config
                 key: APP_NAME
           - name: APP_VERSION
             valueFrom:
               configMapKeyRef:
                 name: app-config
                 key: APP_VERSION
           - name: DB_PASSWORD
             valueFrom:
               secretKeyRef:
                 name: app-secrets
                 key: DB_PASSWORD
           - name: API_KEY
             valueFrom:
               secretKeyRef:
                 name: app-secrets
                 key: API_KEY
           volumeMounts:
           - name: config-volume
             mountPath: /etc/app/config
           - name: secret-volume
             mountPath: /etc/app/secrets
             readOnly: true
         volumes:
         - name: config-volume
           configMap:
             name: app-properties
         - name: secret-volume
           secret:
             secretName: app-private-key
   EOF
   
   kubectl apply -f app-deployment.yaml
   ```

2. **Create a service for the deployment:**
   ```bash
   cat << EOF > app-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: config-app-service
   spec:
     selector:
       app: config-app
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     type: ClusterIP
   EOF
   
   kubectl apply -f app-service.yaml
   ```

3. **Test the deployment:**
   ```bash
   kubectl get pods -l app=config-app
   kubectl exec -it $(kubectl get pods -l app=config-app -o jsonpath='{.items[0].metadata.name}') -- ls -la /etc/app/
   ```

### Exercise 6: ConfigMap and Secret Updates

1. **Update a ConfigMap:**
   ```bash
   # Update the ConfigMap
   kubectl patch configmap app-config --patch '{"data":{"LOG_LEVEL":"debug"}}'
   
   # Or edit the ConfigMap
   kubectl edit configmap app-config
   ```

2. **Update a Secret:**
   ```bash
   # Update the Secret
   kubectl patch secret app-secrets --patch '{"data":{"DB_PASSWORD":"bmV3LXBhc3N3b3Jk"}}'
   
   # Or edit the Secret
   kubectl edit secret app-secrets
   ```

3. **Check if pods are updated:**
   ```bash
   # The pods won't automatically restart, so we need to restart them
   kubectl rollout restart deployment config-app
   
   # Check the rollout status
   kubectl rollout status deployment config-app
   ```

### Exercise 7: Best Practices and Security

1. **Create a ConfigMap with proper labels:**
   ```bash
   cat << EOF > labeled-configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: production-config
     labels:
       app: production-app
       environment: production
       version: v1.0.0
   data:
     database.host: prod-db.example.com
     database.port: "5432"
     redis.host: prod-redis.example.com
     redis.port: "6379"
     api.timeout: "30"
     api.retries: "3"
   EOF
   
   kubectl apply -f labeled-configmap.yaml
   ```

2. **Create a Secret with proper labels:**
   ```bash
   cat << EOF > labeled-secret.yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: production-secrets
     labels:
       app: production-app
       environment: production
       version: v1.0.0
   type: Opaque
   data:
     DB_PASSWORD: cHJvZC1wYXNzd29yZA==
     API_KEY: cHJvZC1hcGkta2V5
     JWT_SECRET: cHJvZC1qd3Qtc2VjcmV0
   EOF
   
   kubectl apply -f labeled-secret.yaml
   ```

3. **Create a pod with security context:**
   ```bash
   cat << EOF > secure-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: secure-pod
   spec:
     securityContext:
       runAsNonRoot: true
       runAsUser: 1000
       runAsGroup: 1000
       fsGroup: 1000
     containers:
     - name: app
       image: nginx:alpine
       securityContext:
         allowPrivilegeEscalation: false
         readOnlyRootFilesystem: true
         capabilities:
           drop:
           - ALL
       volumeMounts:
       - name: config-volume
         mountPath: /etc/app/config
         readOnly: true
       - name: secret-volume
         mountPath: /etc/app/secrets
         readOnly: true
       - name: tmp-volume
         mountPath: /tmp
     volumes:
     - name: config-volume
       configMap:
         name: production-config
     - name: secret-volume
       secret:
         secretName: production-secrets
     - name: tmp-volume
       emptyDir: {}
   EOF
   
   kubectl apply -f secure-pod.yaml
   ```

## 🧹 Cleanup

1. **Delete all resources:**
   ```bash
   kubectl delete -f configmap-env-pod.yaml
   kubectl delete -f configmap-volume-pod.yaml
   kubectl delete -f configmap-specific-keys-pod.yaml
   kubectl delete -f secret-env-pod.yaml
   kubectl delete -f secret-volume-pod.yaml
   kubectl delete -f app-deployment.yaml
   kubectl delete -f app-service.yaml
   kubectl delete -f labeled-configmap.yaml
   kubectl delete -f labeled-secret.yaml
   kubectl delete -f secure-pod.yaml
   ```

2. **Delete ConfigMaps and Secrets:**
   ```bash
   kubectl delete configmap app-config
   kubectl delete configmap app-properties
   kubectl delete configmap app-config-files
   kubectl delete secret app-secrets
   kubectl delete secret app-private-key
   kubectl delete secret app-tls
   ```

3. **Clean up files:**
   ```bash
   rm -f app.properties private.key tls.key tls.crt *.yaml
   rm -rf config/
   ```

## 📚 Key Takeaways

- **ConfigMaps**: Store non-sensitive configuration data
- **Secrets**: Store sensitive data (automatically base64 encoded)
- **Environment Variables**: Inject ConfigMap/Secret data as env vars
- **Volume Mounts**: Mount ConfigMap/Secret data as files
- **Security**: Use security contexts and read-only mounts
- **Updates**: ConfigMap/Secret updates don't automatically restart pods

## 🔍 Troubleshooting

### Common Issues:
1. **ConfigMap not found**: Check ConfigMap name and namespace
2. **Secret not found**: Check Secret name and namespace
3. **Permission denied**: Check security contexts and file permissions
4. **Pods not updated**: Manually restart pods after ConfigMap/Secret changes

### Debug Commands:
```bash
# Check ConfigMap/Secret data
kubectl get configmap <name> -o yaml
kubectl get secret <name> -o yaml

# Check pod environment variables
kubectl exec <pod> -- env | grep <variable>

# Check mounted files
kubectl exec <pod> -- ls -la /path/to/mount

# Check pod logs
kubectl logs <pod>
```

## 🎯 Next Steps

- Learn about Persistent Volumes for data storage
- Explore external secret management (Vault, AWS Secrets Manager)
- Study configuration management tools (Helm, Kustomize)
- Practice with production configuration patterns 