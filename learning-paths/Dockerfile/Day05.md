# Day 5: Advanced Docker Networking and Service Discovery

Today, we will explore advanced networking concepts in Docker, focusing on service discovery and networking strategies that enhance communication and scalability in multi-container environments.

## Objective

- Understand Docker's networking capabilities and how to configure them for complex applications.
- Learn about service discovery mechanisms in Docker.
- Implement best practices for managing Docker networks.

## Topics

### Advanced Docker Networking

Docker provides several networking options to facilitate communication between containers and external systems:

- **Bridge Networking**: The default mode, creating a virtual network for containers on the same Docker host[1][4].

- **Host Networking**: Containers share the host's network stack, allowing direct access to the host's network interfaces[5].

- **Overlay Networking**: Enables communication across multiple Docker hosts, ideal for complex applications deployed in Docker Swarm or Kubernetes[1][6].

#### Creating Custom Networks

Custom networks provide improved isolation and control over communication between containers:

````
docker network create my_custom_network
docker run --network=my_custom_network nginx
````


Custom networks allow containers to communicate using container names as hostnames, leveraging built-in DNS resolution[3][7].

### Service Discovery

Service discovery allows containers to find and communicate with each other without hardcoding IP addresses:

- **DNS-Based Service Discovery**: Docker sets up DNS entries for services, allowing containers to resolve service names to IP addresses within the same network[2][6].

  
Example:
````
services:
web:
image: nginx
networks:
- my_network

db:
image: postgres
networks:
- my_network

networks:
my_network:
driver: bridge
````


In this setup, the `web` service can connect to the `db` service using the hostname `db`.

- **External Tools**: For more complex scenarios, tools like Consul or Traefik can be used for dynamic service discovery and load balancing[2][8].

### Best Practices for Docker Networking

- **Choose the Right Network Driver**: Use bridge for simple setups and overlay for multi-host communication[3].

- **User-Defined Networks**: Avoid using the default bridge network; create custom networks for better isolation[3].

- **Avoid Subnet Overlaps**: Plan your network ranges carefully to prevent conflicts[3].

- **Only Expose Required Ports**: Minimize the attack surface by exposing only necessary ports[3].

## Activities

### Activity 1: Set Up a Custom Network

1. Create a custom network using Docker CLI.
2. Run two containers on this network and verify they can communicate using container names.

Example:
````
docker network create my_app_network
docker run -d --name web --network=my_app_network nginx
docker run -d --name db --network=my_app_network postgres
````


Verify connectivity:
````
docker exec -it web ping db

````

### Activity 2: Implement DNS-Based Service Discovery

1. Use Docker Compose to define services with a shared network.
2. Test service discovery by connecting one service to another using their service names.

Example `docker-compose.yml`:

````
version: '3.8' services:
web:
image: nginx
networks:
- app_net db:
  image: postgres
  environment:
  POSTGRES_PASSWORD: example
  networks:
- app_net networks:
  app_net:
````


### Activity 3: Explore External Service Discovery Tools

1. Research tools like Consul or Traefik.
2. Consider setting up a basic configuration to understand how they enhance service discovery.

## Additional Resources

For further reading and exploration, check out these resources:

- [Mastering Docker Network](https://www.hostmycode.in/tutorials/mastering-docker-network)
- [Docker Networking Best Practices](https://devtodevops.com/docker-network-best-practices/)
- [Docker Swarm Networking](https://docs.docker.com/engine/swarm/networking/)
- [Service Discovery with Consul](https://www.consul.io/docs/discovery)
- [Traefik Documentation](https://doc.traefik.io/traefik/)