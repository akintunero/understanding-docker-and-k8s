
# Day 3: Environment Variables and Advanced Configurations

## Objective

- Use environment variables for dynamic configurations in Docker Compose.
- Explore advanced options like `depends_on`, restart policies, and health checks.

## Topics

### Using Environment Variables in Docker Compose

Environment variables allow you to customize configurations dynamically without hardcoding values in your `docker-compose.yml` file. Docker Compose supports environment variables defined in three ways:

1. **Inline in the Compose File**: Directly within the `docker-compose.yml`.

```
services:
  web:
    image: nginx
    environment:
      - NGINX_HOST=example.com
      - NGINX_PORT=80
```

2. **Using an `.env` File**: A separate file that stores environment variables.

Example `.env` file:

```
NGINX_HOST=example.com
NGINX_PORT=80
```

Reference in `docker-compose.yml`:

```
services:
  web:
    image: nginx
    environment:
      - NGINX_HOST
      - NGINX_PORT
```

3. **Shell Environment Variables**: Variables from your shell environment can be used directly.

### Advanced Configuration Options

#### `depends_on`

The `depends_on` option defines dependencies between services, ensuring that a service starts only after its dependencies are started.

```
services:
  web:
    image: nginx
    depends_on:
      - db
  db:
    image: mysql
```

#### Restart Policies

Restart policies ensure that containers are restarted automatically under specific conditions, enhancing reliability.

```
services:
  web:
    image: nginx
    restart: always
```

Common policies include `no`, `always`, `on-failure`, and `unless-stopped`.

#### Health Checks

Health checks monitor the health of a service, allowing Compose to react to unhealthy states.

```
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

## Activities

### Activity 1: Use Environment Variables with an `.env` File

1. Create an `.env` file in your project directory with the following content:

```
MYSQL_ROOT_PASSWORD=examplepassword
MYSQL_DATABASE=mydb
```

2. Modify your `docker-compose.yml` to use these variables:

```
version: '3.8'
services:
  db:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

3. Run `docker-compose up` and verify that the database is created with the specified environment variables.

### Activity 2: Configure Service Dependencies with `depends_on`

1. Add a `depends_on` configuration to ensure your web service starts after the database service.

```
services:
  web:
    image: nginx
    depends_on:
      - db
  db:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=examplepassword
      - MYSQL_DATABASE=mydb
```

2. Test by running `docker-compose up` and observing the startup order.

### Activity 3: Implement Restart Policies and Health Checks

1. Add a restart policy to your web service to ensure it restarts automatically if it fails.

2. Implement a health check for your database service to monitor its status.

```
services:
  db:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=examplepassword
      - MYSQL_DATABASE=mydb
    restart: on-failure
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 5

  web:
    image: nginx
    depends_on:
      - db
    restart: always
```

3. Run your setup and use `docker ps` to check the health status of your containers.

---

## Additional References

For more detailed information on these topics, check out these resources:

- [Using Environment Variables in Compose](https://docs.docker.com/compose/environment-variables/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Healthcheck Documentation](https://docs.docker.com/engine/reference/builder/#healthcheck)
- [Managing Container Restart Policies](https://docs.docker.com/config/containers/start-containers-automatically/)
