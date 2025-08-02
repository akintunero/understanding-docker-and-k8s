# Lab 03: Advanced Container Management

## 🎯 Learning Objectives
- Master container lifecycle management
- Understand resource management and limits
- Practice advanced container operations
- Learn container cleanup and maintenance
- Implement container health checks

## ⏱️ Estimated Time: 40 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic container operations (Labs 1-2)
- Understanding of Docker commands

## 🚀 Lab Exercises

### Exercise 1: Container Lifecycle Management

1. **Create containers with different restart policies:**
   ```bash
   # Create container with no restart policy
   docker run -d --name no-restart nginx:alpine
   
   # Create container with always restart policy
   docker run -d --name always-restart --restart always nginx:alpine
   
   # Create container with unless-stopped restart policy
   docker run -d --name unless-stopped --restart unless-stopped nginx:alpine
   
   # Create container with on-failure restart policy
   docker run -d --name on-failure --restart on-failure:3 nginx:alpine
   ```

2. **Test restart policies:**
   ```bash
   # Stop containers and observe behavior
   docker stop no-restart always-restart unless-stopped on-failure
   
   # Check container status
   docker ps -a
   
   # Start Docker daemon and check restart behavior
   sudo systemctl restart docker
   docker ps -a
   ```

### Exercise 2: Resource Management

1. **Create containers with resource limits:**
   ```bash
   # Container with CPU and memory limits
   docker run -d --name resource-limited \
     --cpus=0.5 \
     --memory=512m \
     --memory-swap=1g \
     nginx:alpine
   
   # Container with specific CPU cores
   docker run -d --name cpu-pinned \
     --cpuset-cpus=0 \
     nginx:alpine
   
   # Container with memory reservation
   docker run -d --name memory-reserved \
     --memory-reservation=256m \
     --memory=1g \
     nginx:alpine
   ```

2. **Monitor resource usage:**
   ```bash
   # Check container stats
   docker stats resource-limited cpu-pinned memory-reserved
   
   # Get detailed resource information
   docker inspect resource-limited --format='{{.HostConfig.CpuQuota}}'
   docker inspect resource-limited --format='{{.HostConfig.Memory}}'
   ```

3. **Stress test containers:**
   ```bash
   # Run stress test in container
   docker run -it --rm --name stress-test \
     --cpus=1 \
     --memory=512m \
     stress-ng:latest \
     stress-ng --cpu 2 --memory 4 --timeout 30s
   ```

### Exercise 3: Advanced Container Operations

1. **Container checkpoint and restore:**
   ```bash
   # Create a container with a long-running process
   docker run -d --name checkpoint-test \
     ubuntu:20.04 \
     bash -c "while true; do echo 'Running...'; sleep 5; done"
   
   # Create a checkpoint
   docker checkpoint create checkpoint-test checkpoint1
   
   # Stop the container
   docker stop checkpoint-test
   
   # Restore from checkpoint
   docker start --checkpoint checkpoint1 checkpoint-test
   
   # Check if process is still running
   docker logs checkpoint-test
   ```

2. **Container update operations:**
   ```bash
   # Create a container
   docker run -d --name update-test nginx:alpine
   
   # Update container configuration
   docker update --cpus=1 --memory=1g update-test
   
   # Verify updates
   docker inspect update-test --format='{{.HostConfig.CpuQuota}}'
   docker inspect update-test --format='{{.HostConfig.Memory}}'
   ```

### Exercise 4: Container Health Checks

1. **Create container with health check:**
   ```bash
   # Create a simple web application
   cat << EOF > health-check-app.py
   from flask import Flask
   import time
   
   app = Flask(__name__)
   start_time = time.time()
   
   @app.route('/')
   def home():
       return 'Hello, World!'
   
   @app.route('/health')
   def health():
       # Simulate startup delay
       if time.time() - start_time < 10:
           return 'Starting up...', 503
       return 'Healthy', 200
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   EOF
   
   # Create Dockerfile with health check
   cat << EOF > Dockerfile.health
   FROM python:3.9-alpine
   WORKDIR /app
   COPY health-check-app.py .
   RUN pip install flask
   EXPOSE 5000
   HEALTHCHECK --interval=5s --timeout=3s --start-period=15s --retries=3 \
     CMD wget --no-verbose --tries=1 --spider http://localhost:5000/health || exit 1
   CMD ["python", "health-check-app.py"]
   EOF
   
   # Build and run container with health check
   docker build -f Dockerfile.health -t health-check-app .
   docker run -d --name health-test health-check-app
   ```

2. **Monitor health status:**
   ```bash
   # Check health status
   docker ps
   
   # Get detailed health information
   docker inspect health-test --format='{{.State.Health.Status}}'
   docker inspect health-test --format='{{.State.Health.Log}}'
   
   # Watch health status changes
   docker inspect health-test --format='{{.State.Health.Status}}' && sleep 5 && echo "---" && docker inspect health-test --format='{{.State.Health.Status}}'
   ```

### Exercise 5: Container Cleanup and Maintenance

1. **Clean up containers and images:**
   ```bash
   # List all containers (including stopped)
   docker ps -a
   
   # Remove stopped containers
   docker container prune -f
   
   # Remove containers with specific filters
   docker ps -a --filter "status=exited" --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
   docker ps -a --filter "status=exited" -q | xargs docker rm
   
   # Remove containers older than 24 hours
   docker ps -a --filter "until=24h" -q | xargs docker rm
   ```

2. **Clean up images:**
   ```bash
   # List all images
   docker images
   
   # Remove dangling images
   docker image prune -f
   
   # Remove unused images
   docker image prune -a -f
   
   # Remove images older than 7 days
   docker image prune -a --filter "until=168h" -f
   ```

3. **System-wide cleanup:**
   ```bash
   # Check Docker system usage
   docker system df
   
   # Remove unused data
   docker system prune -f
   
   # Remove everything (use with caution)
   docker system prune -a -f --volumes
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Container Orchestration Simulation

1. **Create a simple orchestration script:**
   ```bash
   #!/bin/bash
   
   # Container orchestration script
   CONTAINER_NAME="orchestrated-app"
   MAX_RESTARTS=3
   RESTART_COUNT=0
   
   start_container() {
       echo "Starting container..."
       docker run -d --name $CONTAINER_NAME \
         --restart unless-stopped \
         --memory=512m \
         --cpus=0.5 \
         nginx:alpine
   }
   
   check_health() {
       if docker ps | grep -q $CONTAINER_NAME; then
           echo "Container is running"
           return 0
       else
           echo "Container is not running"
           return 1
       fi
   }
   
   restart_container() {
       if [ $RESTART_COUNT -lt $MAX_RESTARTS ]; then
           echo "Restarting container (attempt $((RESTART_COUNT + 1)))"
           docker restart $CONTAINER_NAME
           RESTART_COUNT=$((RESTART_COUNT + 1))
           sleep 5
           check_health
       else
           echo "Max restart attempts reached"
           exit 1
       fi
   }
   
   # Main orchestration logic
   start_container
   
   while true; do
       if check_health; then
           echo "Container healthy, monitoring..."
           sleep 30
       else
           restart_container
       fi
   done
   EOF
   
   chmod +x container-orchestrator.sh
   ```

2. **Test the orchestrator:**
   ```bash
   # Run the orchestrator in background
   ./container-orchestrator.sh &
   ORCHESTRATOR_PID=$!
   
   # Simulate container failure
   docker stop orchestrated-app
   
   # Monitor the orchestration
   sleep 10
   
   # Check if container was restarted
   docker ps -a | grep orchestrated-app
   
   # Stop the orchestrator
   kill $ORCHESTRATOR_PID
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you manage container lifecycle effectively?
- [ ] Do you understand resource limits and their impact?
- [ ] Can you implement health checks?
- [ ] Can you perform container cleanup and maintenance?
- [ ] Do you understand restart policies?
- [ ] Can you monitor container performance?

### Skills Demonstrated:
- ✅ Advanced container lifecycle management
- ✅ Resource monitoring and limits
- ✅ Health check implementation
- ✅ Container cleanup and maintenance
- ✅ Restart policy configuration
- ✅ Performance monitoring

## 🔍 Troubleshooting

### Common Issues:

1. **Container won't start:**
   ```bash
   # Check container logs
   docker logs <container-name>
   
   # Check resource availability
   docker system df
   
   # Check Docker daemon status
   docker info
   ```

2. **High resource usage:**
   ```bash
   # Monitor resource usage
   docker stats
   
   # Check container limits
   docker inspect <container-name> --format='{{.HostConfig.Memory}}'
   
   # Adjust resource limits
   docker update --memory=1g <container-name>
   ```

3. **Health check failures:**
   ```bash
   # Check health check configuration
   docker inspect <container-name> --format='{{.Config.Healthcheck}}'
   
   # Test health check manually
   docker exec <container-name> <health-check-command>
   
   # Check health check logs
   docker inspect <container-name> --format='{{.State.Health.Log}}'
   ```

## 📚 Additional Resources

- [Docker Container Management](https://docs.docker.com/engine/reference/commandline/container/)
- [Docker Health Checks](https://docs.docker.com/engine/reference/builder/#healthcheck)
- [Docker Resource Limits](https://docs.docker.com/config/containers/resource_constraints/)

## 🎉 Lab Completion

Congratulations! You've completed the Advanced Container Management lab. You now understand:
- Container lifecycle management
- Resource monitoring and limits
- Health check implementation
- Container cleanup and maintenance
- Advanced container operations

**Next Lab**: [Lab 04: Docker Networking](../docker/lab-04-docker-networking.md)

---

**Happy Learning! 🐳** 