# Docker Quick Reference

## 🐳 Essential Commands

### Container Management
```bash
# Run container
docker run <image>

# Run in background
docker run -d <image>

# Run with port mapping
docker run -p 8080:80 <image>

# Run with volume
docker run -v /host:/container <image>

# Run with environment variable
docker run -e VAR=value <image>

# Run with custom name
docker run --name my-container <image>

# Stop container
docker stop <container>

# Start container
docker start <container>

# Restart container
docker restart <container>

# Remove container
docker rm <container>

# Remove all stopped containers
docker container prune
```

### Image Management
```bash
# List images
docker images

# Pull image
docker pull <image>

# Build image
docker build -t name:tag .

# Tag image
docker tag <image> <new-name>:<new-tag>

# Push image
docker push <image>

# Remove image
docker rmi <image>

# Remove all unused images
docker image prune -a
```

### Information and Inspection
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Container logs
docker logs <container>

# Follow logs
docker logs -f <container>

# Container info
docker inspect <container>

# Container stats
docker stats <container>

# Execute command in container
docker exec -it <container> /bin/bash

# Copy files
docker cp <container>:/path /local/path
docker cp /local/path <container>:/path
```

## 🏗️ Dockerfile Commands

### Basic Instructions
```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy files
COPY package*.json ./
COPY . .

# Run commands
RUN npm install

# Expose port
EXPOSE 3000

# Default command
CMD ["npm", "start"]

# Environment variables
ENV NODE_ENV=production

# User
USER nodejs

# Volume
VOLUME ["/app/data"]

# Health check
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
```

### Multi-stage Build
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["npm", "start"]
```

## 🔧 Docker Compose

### Basic Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Rebuild services
docker-compose build

# Scale services
docker-compose up --scale web=3
```

### Compose File Structure
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    environment:
      - NGINX_HOST=localhost
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 🌐 Networking

### Network Commands
```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Connect container to network
docker network connect my-network <container>

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network
```

### Network Types
```bash
# Bridge network (default)
docker run --network bridge <image>

# Host network
docker run --network host <image>

# None network
docker run --network none <image>

# Custom network
docker run --network my-network <image>
```

## 💾 Volumes and Storage

### Volume Commands
```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove unused volumes
docker volume prune
```

### Volume Types
```bash
# Named volume
docker run -v my-volume:/app/data <image>

# Bind mount
docker run -v /host/path:/container/path <image>

# Anonymous volume
docker run -v /container/path <image>
```

## 🔒 Security

### Security Best Practices
```bash
# Run as non-root user
docker run --user 1000:1000 <image>

# Read-only filesystem
docker run --read-only <image>

# Drop capabilities
docker run --cap-drop=ALL <image>

# Security options
docker run --security-opt=no-new-privileges <image>
```

### Image Scanning
```bash
# Scan image for vulnerabilities
docker scan <image>

# Build with security scanning
docker build --security-opt=no-new-privileges .
```

## 📊 Monitoring and Debugging

### Resource Monitoring
```bash
# Container stats
docker stats

# System info
docker system df

# Disk usage
docker system prune

# Memory usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### Debugging Commands
```bash
# Interactive shell
docker exec -it <container> /bin/bash

# Check container processes
docker exec <container> ps aux

# Check container network
docker exec <container> netstat -tulpn

# Check container logs
docker logs <container> --tail 100
```

## 🚀 Advanced Features

### BuildKit
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Build with BuildKit
docker build --progress=plain .

# Multi-platform build
docker buildx build --platform linux/amd64,linux/arm64 .
```

### Docker Registry
```bash
# Login to registry
docker login

# Tag for registry
docker tag <image> registry.com/<image>

# Push to registry
docker push registry.com/<image>

# Pull from registry
docker pull registry.com/<image>
```

## 🔧 Troubleshooting

### Common Issues
```bash
# Container won't start
docker logs <container>

# Port already in use
docker ps -a | grep :80

# Out of disk space
docker system prune -a

# Permission denied
sudo usermod -aG docker $USER

# Network connectivity
docker exec <container> ping google.com
```

### Performance Optimization
```bash
# Use .dockerignore
echo "node_modules" > .dockerignore

# Multi-stage builds
# See multi-stage example above

# Layer caching
# Copy package files before source code

# Alpine base images
# Use alpine variants when possible
```

## 📝 Tips and Tricks

### Useful Aliases
```bash
# Add to ~/.bashrc or ~/.zshrc
alias dps='docker ps'
alias dpsa='docker ps -a'
alias dimg='docker images'
alias dlog='docker logs'
alias dexec='docker exec -it'
alias dstop='docker stop'
alias drm='docker rm'
alias drmi='docker rmi'
```

### Environment Variables
```bash
# Set Docker host
export DOCKER_HOST=tcp://192.168.1.100:2376

# Set Docker context
docker context use my-context

# Set default registry
export DOCKER_REGISTRY=my-registry.com
```

---

**Happy Docker Learning! 🐳** 