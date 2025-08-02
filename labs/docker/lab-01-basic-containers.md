# Lab 01: Basic Container Operations

## 🎯 Learning Objectives
- Run your first Docker container
- Understand container lifecycle
- Practice basic Docker commands
- Learn container inspection and management

## ⏱️ Estimated Time: 30 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic command line knowledge
- Internet connection for pulling images

## 🚀 Lab Exercises

### Exercise 1: Hello World Container

1. **Run your first container:**
   ```bash
   docker run hello-world
   ```

2. **What happened?**
   - Docker pulled the `hello-world` image
   - Created a container from the image
   - Ran the container
   - Displayed a welcome message
   - Container stopped automatically

3. **Check container status:**
   ```bash
   docker ps -a
   ```

### Exercise 2: Interactive Container

1. **Run an interactive Ubuntu container:**
   ```bash
   docker run -it ubuntu:latest /bin/bash
   ```

2. **Inside the container, try these commands:**
   ```bash
   # Check the OS
   cat /etc/os-release
   
   # Check the hostname
   hostname
   
   # Check the current directory
   pwd
   
   # List files
   ls -la
   
   # Exit the container
   exit
   ```

3. **Check container status again:**
   ```bash
   docker ps -a
   ```

### Exercise 3: Container Management

1. **Run a container in detached mode:**
   ```bash
   docker run -d --name my-nginx nginx:alpine
   ```

2. **Check running containers:**
   ```bash
   docker ps
   ```

3. **Stop the container:**
   ```bash
   docker stop my-nginx
   ```

4. **Start the container again:**
   ```bash
   docker start my-nginx
   ```

5. **Remove the container:**
   ```bash
   docker rm my-nginx
   ```

### Exercise 4: Container Inspection

1. **Run a new container:**
   ```bash
   docker run -d --name inspect-demo nginx:alpine
   ```

2. **Inspect the container:**
   ```bash
   docker inspect inspect-demo
   ```

3. **Get specific information:**
   ```bash
   # Get container IP address
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' inspect-demo
   
   # Get container status
   docker inspect -f '{{.State.Status}}' inspect-demo
   
   # Get container image
   docker inspect -f '{{.Config.Image}}' inspect-demo
   ```

4. **View container logs:**
   ```bash
   docker logs inspect-demo
   ```

5. **Follow logs in real-time:**
   ```bash
   docker logs -f inspect-demo
   ```

### Exercise 5: Container Resource Usage

1. **Check container stats:**
   ```bash
   docker stats inspect-demo
   ```

2. **Get container resource usage:**
   ```bash
   docker stats --no-stream inspect-demo
   ```

3. **Clean up:**
   ```bash
   docker stop inspect-demo
   docker rm inspect-demo
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Container Networking

1. **Run two containers and test connectivity:**
   ```bash
   # Run first container
   docker run -d --name container1 nginx:alpine
   
   # Run second container
   docker run -d --name container2 nginx:alpine
   
   # Get IP addresses
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container1
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container2
   ```

2. **Test connectivity between containers:**
   ```bash
   # Execute ping from container1 to container2
   docker exec container1 ping -c 3 <container2-ip>
   ```

3. **Clean up:**
   ```bash
   docker stop container1 container2
   docker rm container1 container2
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you run a basic container?
- [ ] Do you understand container lifecycle?
- [ ] Can you manage containers (start, stop, remove)?
- [ ] Can you inspect container details?
- [ ] Do you understand detached vs interactive mode?

### Skills Demonstrated:
- ✅ Basic container operations
- ✅ Container lifecycle management
- ✅ Container inspection and debugging
- ✅ Resource monitoring
- ✅ Network connectivity understanding

## 🔍 Troubleshooting

### Common Issues:

1. **Permission denied errors:**
   ```bash
   # Add user to docker group
   sudo usermod -aG docker $USER
   # Log out and back in
   ```

2. **Container won't start:**
   ```bash
   # Check container logs
   docker logs <container-name>
   
   # Check container details
   docker inspect <container-name>
   ```

3. **Port already in use:**
   ```bash
   # Check what's using the port
   docker ps -a | grep :80
   
   # Use a different port
   docker run -p 8080:80 nginx:alpine
   ```

## 📚 Additional Resources

- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Docker PS Command](https://docs.docker.com/engine/reference/commandline/ps/)
- [Docker Inspect Command](https://docs.docker.com/engine/reference/commandline/inspect/)

## 🎉 Lab Completion

Congratulations! You've completed your first Docker lab. You now understand:
- How to run containers
- Container lifecycle management
- Basic inspection and debugging
- Resource monitoring

**Next Lab**: [Lab 02: Working with Images](../docker/lab-02-working-with-images.md)

---

**Happy Learning! 🐳** 