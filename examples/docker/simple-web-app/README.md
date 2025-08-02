# Simple Web Application - Docker Example

This is a simple Node.js web application designed to demonstrate Docker concepts and best practices.

## 🚀 Quick Start

### Prerequisites
- Docker installed on your machine
- Basic knowledge of command line

### Building the Image
```bash
# Navigate to the example directory
cd examples/docker/simple-web-app

# Build the Docker image
docker build -t simple-web-app .

# Run the container
docker run -p 3000:3000 simple-web-app
```

### Testing the Application
Once the container is running, you can test the application:

```bash
# Health check
curl http://localhost:3000/health

# Main endpoint
curl http://localhost:3000/

# API info
curl http://localhost:3000/api/info

# Random number generator
curl http://localhost:3000/api/random
```

## 📁 Project Structure

```
simple-web-app/
├── Dockerfile          # Container definition
├── package.json        # Node.js dependencies
├── app.js             # Express.js application
└── README.md          # This file
```

## 🐳 Docker Concepts Demonstrated

### 1. Base Image Selection
- Uses `node:18-alpine` for a lightweight base image
- Alpine Linux provides security and size benefits

### 2. Multi-stage Build Preparation
- Copies package files first for better layer caching
- Installs dependencies before copying source code

### 3. Security Best Practices
- Creates a non-root user (`nodejs`)
- Changes ownership of application files
- Runs application as non-root user

### 4. Health Checks
- Implements health check endpoint
- Docker health check monitors application status

### 5. Port Exposure
- Exposes port 3000 for web traffic
- Maps container port to host port

## 🔧 Customization

### Environment Variables
You can customize the application using environment variables:

```bash
docker run -p 3000:3000 \
  -e NODE_ENV=production \
  -e PORT=3000 \
  simple-web-app
```

### Volume Mounting
For development, you can mount the source code:

```bash
docker run -p 3000:3000 \
  -v $(pwd):/app \
  -e NODE_ENV=development \
  simple-web-app
```

## 🧪 Testing

### Unit Tests
```bash
# Run tests in container
docker run simple-web-app npm test
```

### Integration Tests
```bash
# Test the running application
curl -f http://localhost:3000/health
```

## 📊 Monitoring

### Container Logs
```bash
# View application logs
docker logs <container-id>
```

### Container Stats
```bash
# Monitor resource usage
docker stats <container-id>
```

### Health Check Status
```bash
# Check container health
docker inspect <container-id> --format='{{.State.Health.Status}}'
```

## 🚨 Troubleshooting

### Common Issues

1. **Port already in use**
   ```bash
   # Use a different port
   docker run -p 3001:3000 simple-web-app
   ```

2. **Permission denied**
   ```bash
   # Check if Docker daemon is running
   docker ps
   ```

3. **Application not responding**
   ```bash
   # Check container logs
   docker logs <container-id>
   ```

### Debug Commands
```bash
# Enter the container
docker exec -it <container-id> /bin/sh

# Check running processes
docker exec <container-id> ps aux

# Check network connectivity
docker exec <container-id> curl localhost:3000/health
```

## 📚 Learning Objectives

After completing this example, you should understand:

- ✅ How to create a basic Dockerfile
- ✅ Best practices for Node.js applications
- ✅ Security considerations in containers
- ✅ Health check implementation
- ✅ Port mapping and networking
- ✅ Container lifecycle management
- ✅ Debugging techniques

## 🔗 Related Examples

- [Multi-stage Build Example](../multi-stage-build/)
- [Docker Compose Example](../../multi-service/web-app/)
- [Kubernetes Deployment](../../../kubernetes/simple-deployment/)

---

**Happy Docker Learning! 🐳** 