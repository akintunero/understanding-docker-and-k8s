# Lab 02: Working with Docker Images

## 🎯 Learning Objectives
- Pull images from Docker Hub
- Build custom images with Dockerfile
- Tag and manage images
- Optimize image size and layers
- Understand image layers and caching

## ⏱️ Estimated Time: 45 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic understanding of containers (Lab 01)
- Internet connection for pulling images

## 🚀 Lab Exercises

### Exercise 1: Pulling Images from Docker Hub

1. **Search for images:**
   ```bash
   # Search for nginx images
   docker search nginx
   
   # Search for python images
   docker search python
   ```

2. **Pull specific images:**
   ```bash
   # Pull latest nginx
   docker pull nginx:latest
   
   # Pull specific version
   docker pull nginx:1.21-alpine
   
   # Pull python image
   docker pull python:3.9-slim
   ```

3. **List local images:**
   ```bash
   # List all images
   docker images
   
   # List with specific format
   docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
   ```

### Exercise 2: Building Your First Image

1. **Create a simple application:**
   ```bash
   # Create a directory for your app
   mkdir my-first-app
   cd my-first-app
   
   # Create a simple HTML file
   cat << EOF > index.html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My First Docker App</title>
   </head>
   <body>
       <h1>Hello from Docker!</h1>
       <p>This is my first Docker application.</p>
   </body>
   </html>
   EOF
   ```

2. **Create a Dockerfile:**
   ```bash
   # Create Dockerfile
   cat << EOF > Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   ```

3. **Build the image:**
   ```bash
   # Build the image
   docker build -t my-first-app:latest .
   
   # Build with specific tag
   docker build -t my-first-app:v1.0 .
   ```

4. **Run your custom image:**
   ```bash
   # Run the container
   docker run -d -p 8080:80 --name my-app my-first-app:latest
   
   # Test the application
   curl http://localhost:8080
   ```

### Exercise 3: Image Management

1. **Tag images:**
   ```bash
   # Tag existing image
   docker tag my-first-app:latest my-first-app:stable
   
   # Tag with different name
   docker tag my-first-app:latest myregistry.com/my-first-app:latest
   ```

2. **Inspect image details:**
   ```bash
   # Inspect image
   docker inspect my-first-app:latest
   
   # Get image history
   docker history my-first-app:latest
   
   # Get image layers
   docker image inspect my-first-app:latest --format='{{.RootFS.Layers}}'
   ```

3. **Remove images:**
   ```bash
   # Remove specific image
   docker rmi my-first-app:stable
   
   # Remove all unused images
   docker image prune -a
   ```

### Exercise 4: Multi-stage Build

1. **Create a Node.js application:**
   ```bash
   # Create package.json
   cat << EOF > package.json
   {
     "name": "simple-node-app",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.2"
     }
   }
   EOF
   
   # Create app.js
   cat << EOF > app.js
   const express = require('express');
   const app = express();
   const port = 3000;
   
   app.get('/', (req, res) => {
     res.send('Hello from Node.js Docker app!');
   });
   
   app.listen(port, () => {
     console.log(`App listening at http://localhost:${port}`);
   });
   EOF
   ```

2. **Create multi-stage Dockerfile:**
   ```bash
   # Create multi-stage Dockerfile
   cat << EOF > Dockerfile
   # Build stage
   FROM node:18-alpine AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   
   # Production stage
   FROM node:18-alpine AS production
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci --only=production
   COPY --from=builder /app/app.js ./
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   ```

3. **Build and compare images:**
   ```bash
   # Build multi-stage image
   docker build -t node-app:multi-stage .
   
   # Create single-stage Dockerfile for comparison
   cat << EOF > Dockerfile.single
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   
   # Build single-stage image
   docker build -f Dockerfile.single -t node-app:single .
   
   # Compare image sizes
   docker images | grep node-app
   ```

### Exercise 5: Image Optimization

1. **Create optimized Dockerfile:**
   ```bash
   # Create optimized Dockerfile
   cat << EOF > Dockerfile.optimized
   FROM node:18-alpine
   WORKDIR /app
   
   # Copy package files first for better caching
   COPY package*.json ./
   RUN npm ci --only=production && npm cache clean --force
   
   # Copy application code
   COPY app.js ./
   
   # Create non-root user
   RUN addgroup -g 1001 -S nodejs && \
       adduser -S nodejs -u 1001
   
   # Change ownership
   RUN chown -R nodejs:nodejs /app
   USER nodejs
   
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   ```

2. **Build optimized image:**
   ```bash
   # Build optimized image
   docker build -f Dockerfile.optimized -t node-app:optimized .
   
   # Compare all image sizes
   docker images | grep node-app
   ```

3. **Analyze image layers:**
   ```bash
   # Analyze image layers
   docker history node-app:optimized
   
   # Get detailed layer information
   docker image inspect node-app:optimized --format='{{.RootFS.Layers}}'
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Custom Base Image

1. **Create a custom base image:**
   ```bash
   # Create base Dockerfile
   cat << EOF > Dockerfile.base
   FROM ubuntu:20.04
   
   # Set environment variables
   ENV DEBIAN_FRONTEND=noninteractive
   ENV TZ=UTC
   
   # Install common tools
   RUN apt-get update && apt-get install -y \
       curl \
       wget \
       vim \
       git \
       && rm -rf /var/lib/apt/lists/*
   
   # Create common user
   RUN useradd -m -s /bin/bash developer
   
   # Set working directory
   WORKDIR /app
   
   # Switch to non-root user
   USER developer
   EOF
   ```

2. **Build and use custom base:**
   ```bash
   # Build base image
   docker build -f Dockerfile.base -t my-base:latest .
   
   # Create application using custom base
   cat << EOF > Dockerfile.app
   FROM my-base:latest
   
   # Install Node.js
   USER root
   RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
       apt-get install -y nodejs
   USER developer
   
   # Copy application
   COPY --chown=developer:developer app.js package*.json ./
   RUN npm install
   
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   
   # Build application image
   docker build -f Dockerfile.app -t my-app:custom-base .
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you pull images from Docker Hub?
- [ ] Do you understand image layers and caching?
- [ ] Can you build custom images with Dockerfile?
- [ ] Can you optimize image size?
- [ ] Do you understand multi-stage builds?
- [ ] Can you manage image tags and versions?

### Skills Demonstrated:
- ✅ Image pulling and searching
- ✅ Dockerfile creation and optimization
- ✅ Multi-stage build implementation
- ✅ Image tagging and management
- ✅ Layer caching optimization
- ✅ Security best practices

## 🔍 Troubleshooting

### Common Issues:

1. **Image pull errors:**
   ```bash
   # Check internet connection
   ping docker.io
   
   # Check Docker daemon
   docker info
   
   # Try with different registry
   docker pull registry.hub.docker.com/library/nginx:latest
   ```

2. **Build failures:**
   ```bash
   # Check Dockerfile syntax
   docker build --no-cache .
   
   # Check build context
   ls -la
   
   # Check Docker daemon logs
   docker system info
   ```

3. **Large image sizes:**
   ```bash
   # Analyze image layers
   docker history <image-name>
   
   # Use .dockerignore
   echo "node_modules" > .dockerignore
   
   # Use multi-stage builds
   # See multi-stage example above
   ```

## 📚 Additional Resources

- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Multi-stage Builds](https://docs.docker.com/develop/dev-best-practices/multistage-build/)
- [Docker Hub](https://hub.docker.com/)

## 🎉 Lab Completion

Congratulations! You've completed the Docker Images lab. You now understand:
- How to pull and manage images
- Dockerfile creation and optimization
- Multi-stage builds for efficiency
- Image layer caching and optimization

**Next Lab**: [Lab 03: Container Management](../docker/lab-03-container-management.md)

---

**Happy Learning! 🐳** 