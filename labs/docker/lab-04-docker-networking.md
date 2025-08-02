# Lab 04: Docker Networking

## 🎯 Learning Objectives
- Understand Docker network types and drivers
- Create and manage custom networks
- Configure container communication
- Implement service discovery
- Practice network troubleshooting

## ⏱️ Estimated Time: 50 minutes

## 📋 Prerequisites
- Docker installed and running
- Basic container operations (Labs 1-3)
- Understanding of networking concepts

## 🚀 Lab Exercises

### Exercise 1: Understanding Network Types

1. **List existing networks:**
   ```bash
   # List all networks
   docker network ls
   
   # Get detailed network information
   docker network inspect bridge
   docker network inspect host
   docker network inspect none
   ```

2. **Understand default networks:**
   ```bash
   # Bridge network (default)
   docker run -d --name bridge-container nginx:alpine
   
   # Host network
   docker run -d --name host-container --network host nginx:alpine
   
   # None network
   docker run -d --name none-container --network none nginx:alpine
   ```

3. **Compare network connectivity:**
   ```bash
   # Check bridge container connectivity
   docker exec bridge-container ping -c 3 google.com
   
   # Check host container connectivity
   docker exec host-container ping -c 3 google.com
   
   # Check none container connectivity
   docker exec none-container ping -c 3 google.com
   ```

### Exercise 2: Creating Custom Networks

1. **Create different network types:**
   ```bash
   # Create bridge network
   docker network create --driver bridge my-bridge-network
   
   # Create overlay network (for Swarm)
   docker network create --driver overlay my-overlay-network
   
   # Create network with custom subnet
   docker network create --driver bridge --subnet=172.20.0.0/16 my-custom-network
   
   # Create network with custom gateway
   docker network create --driver bridge --gateway=172.20.0.1 my-gateway-network
   ```

2. **Inspect network configurations:**
   ```bash
   # Inspect network details
   docker network inspect my-bridge-network
   
   # Get network IP range
   docker network inspect my-bridge-network --format='{{.IPAM.Config}}'
   
   # List containers in network
   docker network inspect my-bridge-network --format='{{.Containers}}'
   ```

### Exercise 3: Container Communication

1. **Create containers in custom network:**
   ```bash
   # Create web application container
   docker run -d --name web-app \
     --network my-bridge-network \
     -p 8080:80 \
     nginx:alpine
   
   # Create database container
   docker run -d --name database \
     --network my-bridge-network \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=myapp \
     postgres:13-alpine
   
   # Create cache container
   docker run -d --name cache \
     --network my-bridge-network \
     redis:alpine
   ```

2. **Test container communication:**
   ```bash
   # Test web app to database connection
   docker exec web-app ping -c 3 database
   
   # Test web app to cache connection
   docker exec web-app ping -c 3 cache
   
   # Test database to cache connection
   docker exec database ping -c 3 cache
   ```

3. **Check container IP addresses:**
   ```bash
   # Get container IPs
   docker inspect web-app --format='{{.NetworkSettings.Networks.my-bridge-network.IPAddress}}'
   docker inspect database --format='{{.NetworkSettings.Networks.my-bridge-network.IPAddress}}'
   docker inspect cache --format='{{.NetworkSettings.Networks.my-bridge-network.IPAddress}}'
   ```

### Exercise 4: Service Discovery

1. **Create containers with service names:**
   ```bash
   # Create frontend service
   docker run -d --name frontend \
     --network my-bridge-network \
     -p 3000:3000 \
     node:18-alpine \
     sh -c "npm install -g http-server && http-server -p 3000"
   
   # Create backend service
   docker run -d --name backend \
     --network my-bridge-network \
     -p 5000:5000 \
     python:3.9-alpine \
     sh -c "pip install flask && python -c \"from flask import Flask; app = Flask(__name__); app.run(host='0.0.0.0', port=5000)\""
   ```

2. **Test service discovery:**
   ```bash
   # Test frontend to backend communication
   docker exec frontend wget -O- http://backend:5000
   
   # Test backend to frontend communication
   docker exec backend wget -O- http://frontend:3000
   
   # Test DNS resolution
   docker exec frontend nslookup backend
   docker exec backend nslookup frontend
   ```

### Exercise 5: Network Isolation

1. **Create isolated networks:**
   ```bash
   # Create development network
   docker network create --driver bridge dev-network
   
   # Create production network
   docker network create --driver bridge prod-network
   
   # Create staging network
   docker network create --driver bridge staging-network
   ```

2. **Deploy applications to different networks:**
   ```bash
   # Deploy to development
   docker run -d --name dev-app \
     --network dev-network \
     nginx:alpine
   
   # Deploy to production
   docker run -d --name prod-app \
     --network prod-network \
     nginx:alpine
   
   # Deploy to staging
   docker run -d --name staging-app \
     --network staging-network \
     nginx:alpine
   ```

3. **Test network isolation:**
   ```bash
   # Test dev to prod communication (should fail)
   docker exec dev-app ping -c 3 prod-app
   
   # Test prod to staging communication (should fail)
   docker exec prod-app ping -c 3 staging-app
   
   # Test within same network (should work)
   docker run -d --name dev-app2 --network dev-network nginx:alpine
   docker exec dev-app ping -c 3 dev-app2
   ```

### Exercise 6: Advanced Networking

1. **Create network with custom DNS:**
   ```bash
   # Create network with custom DNS
   docker network create --driver bridge \
     --opt com.docker.network.bridge.name=my-bridge \
     --opt com.docker.network.driver.mtu=1500 \
     my-custom-dns-network
   
   # Run container with custom DNS
   docker run -d --name dns-test \
     --network my-custom-dns-network \
     --dns 8.8.8.8 \
     --dns 8.8.4.4 \
     nginx:alpine
   ```

2. **Test custom DNS:**
   ```bash
   # Test DNS resolution
   docker exec dns-test nslookup google.com
   
   # Test custom DNS servers
   docker exec dns-test cat /etc/resolv.conf
   ```

## 🧪 Challenge Exercise

### Advanced Challenge: Multi-Tier Application Network

1. **Create a complex network architecture:**
   ```bash
   # Create networks for different tiers
   docker network create frontend-network
   docker network create backend-network
   docker network create database-network
   
   # Create load balancer
   docker run -d --name load-balancer \
     --network frontend-network \
     -p 80:80 \
     nginx:alpine
   
   # Create web servers
   docker run -d --name web-server-1 \
     --network frontend-network \
     --network backend-network \
     nginx:alpine
   
   docker run -d --name web-server-2 \
     --network frontend-network \
     --network backend-network \
     nginx:alpine
   
   # Create application servers
   docker run -d --name app-server-1 \
     --network backend-network \
     --network database-network \
     python:3.9-alpine \
     sh -c "pip install flask && python -c \"from flask import Flask; app = Flask(__name__); app.run(host='0.0.0.0', port=5000)\""
   
   docker run -d --name app-server-2 \
     --network backend-network \
     --network database-network \
     python:3.9-alpine \
     sh -c "pip install flask && python -c \"from flask import Flask; app = Flask(__name__); app.run(host='0.0.0.0', port=5000)\""
   
   # Create database
   docker run -d --name database \
     --network database-network \
     -e POSTGRES_PASSWORD=password \
     postgres:13-alpine
   ```

2. **Test network connectivity:**
   ```bash
   # Test frontend to backend
   docker exec web-server-1 ping -c 3 app-server-1
   docker exec web-server-2 ping -c 3 app-server-2
   
   # Test backend to database
   docker exec app-server-1 ping -c 3 database
   docker exec app-server-2 ping -c 3 database
   
   # Test isolation (should fail)
   docker exec load-balancer ping -c 3 database
   ```

## 📊 Lab Assessment

### Self-Check Questions:
- [ ] Can you create and manage custom networks?
- [ ] Do you understand different network drivers?
- [ ] Can you configure container communication?
- [ ] Can you implement service discovery?
- [ ] Do you understand network isolation?
- [ ] Can you troubleshoot network issues?

### Skills Demonstrated:
- ✅ Network type understanding
- ✅ Custom network creation
- ✅ Container communication
- ✅ Service discovery implementation
- ✅ Network isolation
- ✅ Network troubleshooting

## 🔍 Troubleshooting

### Common Issues:

1. **Container cannot reach external network:**
   ```bash
   # Check network configuration
   docker network inspect bridge
   
   # Check container network settings
   docker inspect <container-name> --format='{{.NetworkSettings}}'
   
   # Test DNS resolution
   docker exec <container-name> nslookup google.com
   ```

2. **Containers cannot communicate:**
   ```bash
   # Check if containers are in same network
   docker network inspect <network-name>
   
   # Check container IP addresses
   docker inspect <container-name> --format='{{.NetworkSettings.Networks.<network-name>.IPAddress}}'
   
   # Test connectivity
   docker exec <container-name> ping <target-container>
   ```

3. **Port binding issues:**
   ```bash
   # Check port bindings
   docker port <container-name>
   
   # Check host port usage
   netstat -tulpn | grep <port>
   
   # Use different port
   docker run -p <host-port>:<container-port> <image>
   ```

## 📚 Additional Resources

- [Docker Networking](https://docs.docker.com/network/)
- [Network Drivers](https://docs.docker.com/network/drivers/)
- [Service Discovery](https://docs.docker.com/network/bridge/)

## 🎉 Lab Completion

Congratulations! You've completed the Docker Networking lab. You now understand:
- Docker network types and drivers
- Custom network creation and management
- Container communication and service discovery
- Network isolation and security
- Network troubleshooting techniques

**Next Lab**: [Lab 05: Data Management with Volumes](../docker/lab-05-data-management.md)

---

**Happy Learning! 🐳** 