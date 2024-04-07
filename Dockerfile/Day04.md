# Day 4: Scaling and Load Balancing with Docker Compose

Today, we will focus on advanced Docker Compose techniques, specifically scaling services and implementing load balancing. These concepts are essential for managing large-scale, distributed applications.

## Objective

- Learn how to scale services using Docker Compose.
- Understand how to implement load balancing within a Docker Compose setup.
- Explore the role of Docker networking in scaling and load balancing.

## Topics

### Scaling Services with Docker Compose

Docker Compose allows you to scale services horizontally by running multiple instances of the same service. This is particularly useful for stateless applications like web servers.

- **Scaling Command**:
  Use the `docker-compose up --scale` command to define the number of instances for a service.

  Example:
````
docker-compose up --scale web=3
````


This command starts three instances of the `web` service defined in your `docker-compose.yml`.

- **Replica Configuration**:
  Alternatively, you can define replicas directly in the `docker-compose.yml` file (Compose v3+):
```
version: '3.8' services:
web:
image: nginx:latest
deploy:
replicas: 3
restart_policy:
condition: on-failure
ports:
- "8080:80"
  networks:
- app_network networks:
  app_network:
  driver: bridge

```


### Load Balancing

When multiple instances of a service are running, load balancing ensures that incoming requests are distributed evenly across all instances.

- **Docker's Built-in Load Balancing**:
  Docker automatically handles basic load balancing when services are scaled. It uses an internal DNS system to distribute requests across containers in the same network.

- **External Load Balancers**:
  For more advanced use cases, you can integrate external load balancers like NGINX or Traefik.

Example: Using NGINX as a reverse proxy for load balancing:

````
version: '3.8' services:
web:
image: nginx:latest
deploy:
replicas: 3
networks:
- app_network

text
proxy:
image: nginx
volumes:
- ./nginx.conf:/etc/nginx/nginx.conf
ports:
- "8080:80"
networks:
- app_network

networks:
app_network:
driver: bridge

````

### Networking in Scaling and Load Balancing

- **Custom Networks**:
  Use custom networks to isolate services and enable communication between scaled instances.

- **DNS-Based Service Discovery**:
  Docker's internal DNS resolves service names to IP addresses of all running containers for that service, enabling load balancing.

## Activities

### Activity 1: Scale a Web Service

1. Create a `docker-compose.yml` file with a simple web server (e.g., NGINX).
2. Scale the web service to run three instances using the `--scale` flag.
3. Verify that all instances are running using `docker ps`.

Example:

````
version: '3.8' services:
web:
image: nginx:latest
ports:
- "8080:80"

````

Command:

````
docker-compose up --scale web=3
````


### Activity 2: Implement Load Balancing with NGINX

1. Create an NGINX configuration file (`nginx.conf`) to act as a reverse proxy.
2. Update your `docker-compose.yml` file to include an NGINX proxy service.
3. Test the setup by sending multiple requests and observing how they are distributed across instances.

Example `nginx.conf`:

````
events {} http {
upstream backend {
server web1;
server web2;
server web3;
}

text
server {
listen 80;

    location / {
        proxy_pass http://backend;
    }
}

}

````

Update `docker-compose.yml`:
````
version: '3.8' services:
web1:
image: nginx:latest
networks:
- app_network web2:
  image: nginx:latest
  networks:
- app_network web3:
  image: nginx:latest
  networks:
- app_network proxy:
  image: nginx
  volumes:
- ./nginx.conf:/etc/nginx/nginx.conf
  ports:
- "8080:80"
  networks:
- app_network networks:
  app_network:
````


### Activity 3: Test Scaling and Load Balancing

1. Use tools like `curl` or Postman to send requests to your application.
2. Observe how requests are distributed across different instances using logs or container IDs returned in responses.

## Additional References

For further reading and exploration, check out these resources:

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Scaling Services with Docker Compose](https://docs.docker.com/compose/compose-file/compose-file-v3/#deploy)
- [NGINX as a Reverse Proxy](https://www.nginx.com/resources/glossary/reverse-proxy/)
- [Docker Networking Overview](https://docs.docker.com/network/)