# Lab 09: Docker Security

## 🎯 Learning Objectives
- Understand Docker security best practices
- Implement non-root user containers
- Scan images for vulnerabilities
- Configure security policies
- Learn about Docker security features

## ⏱️ Estimated Time: 60 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Linux security concepts
- Knowledge of Docker images and containers

## 🚀 Lab Exercises

### Exercise 1: Non-Root User Containers

1. **Create a Dockerfile with non-root user:**
   ```bash
   cat > Dockerfile.security << EOF
   FROM ubuntu:20.04
   
   # Create a non-root user
   RUN groupadd -r appuser && useradd -r -g appuser appuser
   
   # Create application directory
   RUN mkdir -p /app && chown appuser:appuser /app
   
   # Switch to non-root user
   USER appuser
   
   # Set working directory
   WORKDIR /app
   
   # Copy application files
   COPY --chown=appuser:appuser app.py .
   
   # Expose port
   EXPOSE 8080
   
   # Run application
   CMD ["python3", "app.py"]
   EOF
   ```

2. **Create a simple Python application:**
   ```bash
   cat > app.py << EOF
   import os
   import sys
   
   print(f"Running as user: {os.getuid()}")
   print(f"User name: {os.getenv('USER', 'unknown')}")
   print(f"Current directory: {os.getcwd()}")
   print(f"Files in directory: {os.listdir('.')}")
   
   # Try to write to a file
   try:
       with open('test.txt', 'w') as f:
           f.write('Hello from container!')
       print("Successfully wrote to file")
   except PermissionError:
       print("Permission denied - cannot write to file")
   
   # Keep container running
   import time
   while True:
       time.sleep(10)
   EOF
   ```

3. **Build and run the secure container:**
   ```bash
   docker build -f Dockerfile.security -t secure-app .
   docker run -d --name secure-demo secure-app
   ```

4. **Check the container logs:**
   ```bash
   docker logs secure-demo
   ```

### Exercise 2: Vulnerability Scanning

1. **Install Trivy (vulnerability scanner):**
   ```bash
   # For macOS
   brew install trivy
   
   # For Linux
   wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
   echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
   sudo apt-get update
   sudo apt-get install trivy
   ```

2. **Scan an image for vulnerabilities:**
   ```bash
   trivy image nginx:alpine
   ```

3. **Scan with detailed output:**
   ```bash
   trivy image --severity HIGH,CRITICAL nginx:alpine
   ```

4. **Scan your custom image:**
   ```bash
   trivy image secure-app
   ```

5. **Generate vulnerability report:**
   ```bash
   trivy image --format json --output nginx-vulnerabilities.json nginx:alpine
   ```

### Exercise 3: Security Policies

1. **Create a security policy file:**
   ```bash
   cat > security-policy.yaml << EOF
   apiVersion: v1
   kind: Pod
   metadata:
     name: security-policy-demo
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
       - name: tmp
         mountPath: /tmp
       - name: varlog
         mountPath: /var/log
     volumes:
     - name: tmp
       emptyDir: {}
     - name: varlog
       emptyDir: {}
   EOF
   ```

2. **Create a Docker Compose file with security policies:**
   ```bash
   cat > docker-compose.security.yml << EOF
   version: '3.8'
   
   services:
     secure-web:
       image: nginx:alpine
       security_opt:
         - no-new-privileges:true
       read_only: true
       tmpfs:
         - /tmp
         - /var/cache/nginx
         - /var/run
       user: "1000:1000"
       ports:
         - "8080:80"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf:ro
   
     secure-app:
       build:
         context: .
         dockerfile: Dockerfile.security
       security_opt:
         - no-new-privileges:true
       read_only: true
       tmpfs:
         - /tmp
       user: "appuser"
       ports:
         - "8081:8080"
   EOF
   ```

3. **Create nginx configuration:**
   ```bash
   cat > nginx.conf << EOF
   events {
       worker_connections 1024;
   }
   
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
       
       log_format main '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                       '\$status \$body_bytes_sent "\$http_referer" '
                       '"\$http_user_agent" "\$http_x_forwarded_for"';
       
       access_log /var/log/nginx/access.log main;
       error_log /var/log/nginx/error.log;
       
       server {
           listen 80;
           server_name localhost;
           
           location / {
               root   /usr/share/nginx/html;
               index  index.html index.htm;
           }
       }
   }
   EOF
   ```

4. **Run the secure services:**
   ```bash
   docker-compose -f docker-compose.security.yml up -d
   ```

### Exercise 4: Image Signing and Verification

1. **Enable Docker Content Trust:**
   ```bash
   export DOCKER_CONTENT_TRUST=1
   ```

2. **Pull a signed image:**
   ```bash
   docker pull nginx:alpine
   ```

3. **Disable content trust for testing:**
   ```bash
   export DOCKER_CONTENT_TRUST=0
   ```

### Exercise 5: Resource Limits and Security

1. **Create a container with resource limits:**
   ```bash
   docker run -d \
     --name resource-limited \
     --memory=512m \
     --cpus=1.0 \
     --pids-limit=100 \
     --security-opt=no-new-privileges \
     --cap-drop=ALL \
     nginx:alpine
   ```

2. **Check resource usage:**
   ```bash
   docker stats resource-limited
   ```

3. **Test resource limits:**
   ```bash
   # Try to use more memory than allocated
   docker exec resource-limited sh -c "python3 -c 'import time; [time.sleep(1) for _ in range(1000000)]'"
   ```

### Exercise 6: Security Auditing

1. **Audit container configuration:**
   ```bash
   docker inspect resource-limited | jq '.[0].Config'
   ```

2. **Check for security issues:**
   ```bash
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
     aquasec/trivy:latest config /var/run/docker.sock
   ```

3. **Scan Dockerfile for best practices:**
   ```bash
   cat > Dockerfile.audit << EOF
   FROM ubuntu:20.04
   
   # Install packages
   RUN apt-get update && apt-get install -y python3
   
   # Create user
   RUN useradd -m appuser
   
   # Copy application
   COPY app.py /app/
   RUN chown appuser:appuser /app/app.py
   
   USER appuser
   WORKDIR /app
   
   CMD ["python3", "app.py"]
   EOF
   
   trivy config .
   ```

### Exercise 7: Network Security

1. **Create a custom network:**
   ```bash
   docker network create --driver bridge secure-network
   ```

2. **Run containers on secure network:**
   ```bash
   docker run -d \
     --name secure-web \
     --network secure-network \
     --security-opt=no-new-privileges \
     nginx:alpine
   
   docker run -d \
     --name secure-db \
     --network secure-network \
     --security-opt=no-new-privileges \
     postgres:13-alpine
   ```

3. **Test network isolation:**
   ```bash
   # Test connectivity between containers
   docker exec secure-web ping -c 3 secure-db
   
   # Test external connectivity
   docker exec secure-web ping -c 3 google.com
   ```

## 🧹 Cleanup

1. **Stop and remove containers:**
   ```bash
   docker stop secure-demo resource-limited secure-web secure-db
   docker rm secure-demo resource-limited secure-web secure-db
   ```

2. **Stop Docker Compose services:**
   ```bash
   docker-compose -f docker-compose.security.yml down
   ```

3. **Remove networks:**
   ```bash
   docker network rm secure-network
   ```

4. **Clean up files:**
   ```bash
   rm -f Dockerfile.security app.py security-policy.yaml \
        docker-compose.security.yml nginx.conf Dockerfile.audit \
        nginx-vulnerabilities.json
   ```

## 📚 Key Takeaways

- **Non-Root Users**: Always run containers as non-root users
- **Vulnerability Scanning**: Regularly scan images for security vulnerabilities
- **Resource Limits**: Set appropriate resource limits for containers
- **Security Policies**: Implement security contexts and policies
- **Network Security**: Use custom networks for isolation
- **Image Signing**: Use Docker Content Trust for image verification

## 🔍 Troubleshooting

### Common Issues:
1. **Permission denied errors**: Check user permissions in containers
2. **Vulnerability scanner not found**: Install Trivy or alternative scanner
3. **Security policy violations**: Review and adjust security contexts
4. **Resource limit exceeded**: Monitor and adjust resource allocations

### Debug Commands:
```bash
# Check container security context
docker inspect <container> | jq '.[0].Config.User'

# Verify vulnerability scan results
trivy image --severity HIGH <image>

# Check resource usage
docker stats <container>

# Audit container configuration
docker inspect <container> | jq '.[0].Config.SecurityOpt'
```

## 🎯 Next Steps

- Implement security scanning in CI/CD pipelines
- Learn about Docker Bench Security
- Explore container runtime security tools
- Study Kubernetes security policies
- Practice with security-focused container images 