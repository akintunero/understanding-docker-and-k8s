# Lab 12: Docker Registry Management

## 🎯 Learning Objectives
- Set up and manage private Docker registries
- Understand image distribution workflows
- Implement access control and authentication
- Manage image versioning and tagging
- Learn registry best practices

## ⏱️ Estimated Time: 65 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of Docker images and containers
- Knowledge of networking concepts

## 🚀 Lab Exercises

### Exercise 1: Local Registry Setup

1. **Start a local Docker registry:**
   ```bash
   docker run -d \
     --name registry \
     -p 5000:5000 \
     -v registry-data:/var/lib/registry \
     registry:2
   ```

2. **Verify registry is running:**
   ```bash
   docker ps | grep registry
   curl http://localhost:5000/v2/
   ```

3. **Create a test image:**
   ```bash
   cat > Dockerfile.test << EOF
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   
   cat > index.html << EOF
   <html>
   <body>
     <h1>Hello from Private Registry!</h1>
     <p>This image was pushed to our local registry.</p>
   </body>
   </html>
   EOF
   ```

4. **Build and tag the image:**
   ```bash
   docker build -f Dockerfile.test -t localhost:5000/myapp:v1.0 .
   ```

5. **Push to local registry:**
   ```bash
   docker push localhost:5000/myapp:v1.0
   ```

6. **Pull from local registry:**
   ```bash
   docker pull localhost:5000/myapp:v1.0
   ```

### Exercise 2: Registry with Authentication

1. **Create authentication directory:**
   ```bash
   mkdir -p auth
   ```

2. **Create password file:**
   ```bash
   htpasswd -Bbn admin secret123 > auth/htpasswd
   ```

3. **Stop the previous registry:**
   ```bash
   docker stop registry
   docker rm registry
   ```

4. **Start registry with authentication:**
   ```bash
   docker run -d \
     --name registry-auth \
     -p 5000:5000 \
     -v registry-data:/var/lib/registry \
     -v $(pwd)/auth:/auth \
     -e REGISTRY_AUTH=htpasswd \
     -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
     -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
     registry:2
   ```

5. **Login to registry:**
   ```bash
   docker login localhost:5000
   # Username: admin
   # Password: secret123
   ```

6. **Push image with authentication:**
   ```bash
   docker push localhost:5000/myapp:v1.0
   ```

### Exercise 3: Registry with TLS

1. **Generate SSL certificates:**
   ```bash
   mkdir -p certs
   
   # Generate private key
   openssl genrsa -out certs/domain.key 2048
   
   # Generate certificate signing request
   openssl req -new -key certs/domain.key -out certs/domain.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"
   
   # Generate self-signed certificate
   openssl x509 -req -days 365 -in certs/domain.csr -signkey certs/domain.key -out certs/domain.crt
   ```

2. **Stop previous registry:**
   ```bash
   docker stop registry-auth
   docker rm registry-auth
   ```

3. **Start registry with TLS:**
   ```bash
   docker run -d \
     --name registry-tls \
     -p 5000:5000 \
     -v registry-data:/var/lib/registry \
     -v $(pwd)/auth:/auth \
     -v $(pwd)/certs:/certs \
     -e REGISTRY_AUTH=htpasswd \
     -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
     -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
     -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
     -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
     registry:2
   ```

4. **Test TLS connection:**
   ```bash
   curl -k https://localhost:5000/v2/
   ```

### Exercise 4: Image Versioning and Tagging

1. **Create multiple versions of an application:**
   ```bash
   # Version 1.0
   cat > app-v1.py << EOF
   from flask import Flask
   app = Flask(__name__)
   
   @app.route('/')
   def hello():
       return {'version': '1.0', 'message': 'Hello from v1.0!'}
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   
   # Version 2.0
   cat > app-v2.py << EOF
   from flask import Flask
   app = Flask(__name__)
   
   @app.route('/')
   def hello():
       return {'version': '2.0', 'message': 'Hello from v2.0!', 'features': ['api', 'health']}
   
   @app.route('/health')
   def health():
       return {'status': 'healthy'}
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   ```

2. **Create Dockerfiles for different versions:**
   ```bash
   cat > Dockerfile.v1 << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY app-v1.py app.py
   EXPOSE 5000
   CMD ["python", "app.py"]
   EOF
   
   cat > Dockerfile.v2 << EOF
   FROM python:3.9-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY app-v2.py app.py
   EXPOSE 5000
   CMD ["python", "app.py"]
   EOF
   
   cat > requirements.txt << EOF
   flask==2.0.1
   EOF
   ```

3. **Build and tag multiple versions:**
   ```bash
   # Build v1.0
   docker build -f Dockerfile.v1 -t localhost:5000/myapp:v1.0 .
   docker tag localhost:5000/myapp:v1.0 localhost:5000/myapp:latest
   
   # Build v2.0
   docker build -f Dockerfile.v2 -t localhost:5000/myapp:v2.0 .
   docker tag localhost:5000/myapp:v2.0 localhost:5000/myapp:latest
   
   # Add additional tags
   docker tag localhost:5000/myapp:v1.0 localhost:5000/myapp:stable
   docker tag localhost:5000/myapp:v2.0 localhost:5000/myapp:beta
   ```

4. **Push all versions to registry:**
   ```bash
   docker push localhost:5000/myapp:v1.0
   docker push localhost:5000/myapp:v2.0
   docker push localhost:5000/myapp:latest
   docker push localhost:5000/myapp:stable
   docker push localhost:5000/myapp:beta
   ```

5. **List images in registry:**
   ```bash
   curl -u admin:secret123 http://localhost:5000/v2/myapp/tags/list
   ```

### Exercise 5: Registry Management Tools

1. **Install registry management tools:**
   ```bash
   # Install crane (for registry operations)
   # For macOS
   brew install crane
   
   # For Linux
   wget https://github.com/google/go-containerregistry/releases/download/v0.11.1/go-containerregistry_Linux_x86_64.tar.gz
   tar -xzf go-containerregistry_Linux_x86_64.tar.gz
   sudo mv crane /usr/local/bin/
   ```

2. **Use crane to manage registry:**
   ```bash
   # List repositories
   crane catalog localhost:5000
   
   # List tags for a repository
   crane ls localhost:5000/myapp
   
   # Copy image between registries
   crane copy localhost:5000/myapp:v1.0 localhost:5000/myapp:backup
   
   # Delete image
   crane delete localhost:5000/myapp:backup
   ```

### Exercise 6: Registry with Storage Backend

1. **Create S3-compatible storage configuration:**
   ```bash
   cat > registry-config.yml << EOF
   version: 0.1
   log:
     level: debug
   storage:
     cache:
       blobdescriptor: inmemory
     filesystem:
       rootdirectory: /var/lib/registry
   auth:
     htpasswd:
       realm: basic-realm
       path: /auth/htpasswd
   http:
     addr: :5000
     headers:
       X-Content-Type-Options: [nosniff]
   health:
     storagedriver:
       enabled: true
       interval: 10s
       threshold: 3
   EOF
   ```

2. **Start registry with custom config:**
   ```bash
   docker stop registry-tls
   docker rm registry-tls
   
   docker run -d \
     --name registry-config \
     -p 5000:5000 \
     -v registry-data:/var/lib/registry \
     -v $(pwd)/auth:/auth \
     -v $(pwd)/registry-config.yml:/etc/docker/registry/config.yml \
     registry:2
   ```

3. **Test the configured registry:**
   ```bash
   curl http://localhost:5000/v2/
   ```

### Exercise 7: Registry Cleanup and Maintenance

1. **Create cleanup script:**
   ```bash
   cat > cleanup-registry.sh << EOF
   #!/bin/bash
   
   echo "=== Registry Cleanup Script ==="
   
   # List all images in registry
   echo "Current images in registry:"
   curl -u admin:secret123 http://localhost:5000/v2/myapp/tags/list
   
   # Remove old images (keep only latest 3 versions)
   echo "Cleaning up old images..."
   
   # Get all tags
   TAGS=\$(curl -s -u admin:secret123 http://localhost:5000/v2/myapp/tags/list | jq -r '.tags[]')
   
   # Keep only the latest 3 versions
   COUNT=0
   for tag in \$TAGS; do
       if [ \$COUNT -ge 3 ]; then
           echo "Removing old tag: \$tag"
           # Note: This requires registry garbage collection to be enabled
           # docker exec registry-config registry garbage-collect /etc/docker/registry/config.yml
       fi
       COUNT=\$((COUNT + 1))
   done
   
   echo "Cleanup completed!"
   EOF
   
   chmod +x cleanup-registry.sh
   ```

2. **Run cleanup:**
   ```bash
   ./cleanup-registry.sh
   ```

3. **Monitor registry storage:**
   ```bash
   # Check registry storage usage
   docker exec registry-config du -sh /var/lib/registry
   
   # List registry contents
   docker exec registry-config ls -la /var/lib/registry/docker/registry/v2/repositories/
   ```

## 🧹 Cleanup

1. **Stop and remove registry containers:**
   ```bash
   docker stop registry registry-auth registry-tls registry-config
   docker rm registry registry-auth registry-tls registry-config
   ```

2. **Remove registry volumes:**
   ```bash
   docker volume rm registry-data
   ```

3. **Clean up files:**
   ```bash
   rm -f Dockerfile.test index.html app-v*.py Dockerfile.v* requirements.txt \
        registry-config.yml cleanup-registry.sh
   rm -rf auth/ certs/
   ```

## 📚 Key Takeaways

- **Local Registry**: Use for development and testing
- **Authentication**: Implement proper access control
- **TLS**: Secure registry communications
- **Versioning**: Use semantic versioning for images
- **Storage**: Configure appropriate storage backends
- **Maintenance**: Regular cleanup and monitoring

## 🔍 Troubleshooting

### Common Issues:
1. **Connection refused**: Check if registry is running and port is accessible
2. **Authentication failed**: Verify credentials and registry configuration
3. **TLS errors**: Check certificate configuration and client setup
4. **Push/pull failures**: Ensure proper image tagging and registry URL

### Debug Commands:
```bash
# Check registry status
curl http://localhost:5000/v2/

# Verify authentication
docker login localhost:5000

# List registry contents
curl -u admin:secret123 http://localhost:5000/v2/myapp/tags/list

# Check registry logs
docker logs registry-config
```

## 🎯 Next Steps

- Set up production-grade registry (Harbor, AWS ECR, Azure ACR)
- Implement CI/CD integration with registries
- Learn about image scanning and vulnerability management
- Explore registry backup and disaster recovery
- Study Kubernetes registry integration 