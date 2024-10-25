
# Day 5: Deployment and Troubleshooting

## Objective

- Deploy multi-container applications in different environments (development, staging, production).

- Learn how to troubleshoot common issues in Docker Compose setups.

## Topics

### Using Multiple Compose Files for Different Environments

Docker Compose allows you to use multiple Compose files to define different configurations for various environments. This enables you to maintain separate settings for development, testing, and production.

- **Base Compose File (`docker-compose.yml`)**: Contains common configurations.

- **Override File (`docker-compose.override.yml`)**: Contains environment-specific overrides.

Example:

**docker-compose.yml**

```
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```

**docker-compose.override.yml (for development)**

```
version: '3.8'
services:
  web:
    volumes:
      - ./web:/usr/share/nginx/html
    environment:
      - NGINX_ENV=development
  db:
    environment:
      MYSQL_DATABASE: dev_db
```

To use the override file, simply run:

```
docker-compose up
```

### Debugging Container Issues

Troubleshooting involves identifying and resolving issues that may arise during the deployment and operation of Docker containers.

- **Logs**: Use `docker-compose logs` to view logs from all services.

```
docker-compose logs web
```

- **Exec into Containers**: Use `docker exec` to run commands inside a running container for debugging purposes.

```
docker exec -it <container_name> /bin/bash
```

- **Inspect Containers**: Use `docker inspect` to view detailed information about containers.

```
docker inspect <container_name>
```

### Best Practices for Deploying Applications with Docker Compose

- **Version Control**: Keep your Compose files under version control for easy management and rollback.

- **Environment Variables**: Use environment variables to manage configuration across different environments.

- **Resource Constraints**: Define resource limits to ensure that services do not consume more resources than allocated.

---

## Activities

### Activity 1: Create Separate Compose Files for Different Environments

1. Create a `docker-compose.override.yml` file for development settings.

2. Modify the override file to include additional configurations specific to the development environment.

**Example: `docker-compose.override.yml` (for development)**

```
version: '3.8'
services:
  web:
    volumes:
      - ./web:/usr/share/nginx/html
    environment:
      - NGINX_ENV=development
  db:
    environment:
      MYSQL_DATABASE: dev_db
```

3. Run your application using the override file and verify the settings.

### Activity 2: Debug Common Issues

1. Use `docker-compose logs` to identify issues in your services.

2. Exec into a running container using `docker exec` to inspect the environment and troubleshoot problems.

3. Use `docker inspect` to gather detailed information about a specific container.

**Example Commands**:

```
docker-compose logs web
docker exec -it <web_container_id> /bin/bash
docker inspect <web_container_id>
```

### Activity 3: Deploy Your Application Using Production Configuration

1. Create a separate Compose file or modify existing ones for production settings.

2. Ensure that sensitive information is managed securely using environment variables or secrets management tools.

3. Deploy your application using the production configuration and test its functionality.

---

## Additional References

For further reading and exploration, check out these resources:

- [Docker Compose Documentation](https://docs.docker.com/compose/)

- [Compose File Reference](https://docs.docker.com/compose/compose-file/)

- [Managing Container Logs](https://docs.docker.com/config/containers/logging/)

- [Docker Inspect Command](https://docs.docker.com/engine/reference/commandline/inspect/)

- [Best Practices for Docker Compose](https://docs.docker.com/compose/best-practices/)
