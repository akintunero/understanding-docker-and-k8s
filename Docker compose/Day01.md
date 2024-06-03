
# Day 1: Introduction to Docker Compose

## Objective

- Understand what Docker Compose is and its advantages.
- Learn the basic structure of a `docker-compose.yml` file.
- Set up and run a simple multi-container application.

## Topics

### Overview of Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. With Docker Compose, you can use a YAML file (`docker-compose.yml`) to configure your application's services, networks, and volumes. It simplifies the process of managing multiple containers by allowing you to start, stop, and manage them as a single unit.

### Installing Docker Compose

To install Docker Compose:

1. **Linux**: Follow the official [Docker Compose installation guide](https://docs.docker.com/compose/install/).
2. **Windows/Mac**: Docker Desktop includes Docker Compose by default.

Verify the installation:

```
docker-compose --version
```

### Basic Syntax of `docker-compose.yml`

A `docker-compose.yml` file defines services, networks, and volumes for your application. Below is an example structure:

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

- **`version`**: Specifies the Compose file format version.
- **`services`**: Defines the containers (e.g., `web`, `db`).
- **`image`**: Specifies the Docker image to use.
- **`ports`**: Maps container ports to host machine ports.
- **`environment`**: Sets environment variables for the container.

### Running Applications with `docker-compose`

- **Start the application**:

```
docker-compose up
```

This command builds, creates, starts, and attaches to containers defined in the `docker-compose.yml`.

- **Stop the application**:

```
docker-compose down
```

This command stops and removes containers, networks, and volumes created by `up`.

---

## Activities

### Activity 1: Install Docker Compose on Your System

1. Follow the installation instructions for your operating system from the [official documentation](https://docs.docker.com/compose/install/).
2. Verify your installation by running:

```
docker-compose --version
```

### Activity 2: Create a Simple `docker-compose.yml` File

1. Create a new directory for your project (e.g., `my-app`) and navigate into it.
2. Create a file named `docker-compose.yml`.
3. Define two services (a web server and a database) in the file:

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

### Activity 3: Run Your Application with Docker Compose

1. Start your application using:

```
docker-compose up
```

2. Access the web server at `http://localhost:8080`.
3. Stop your application using:

```
docker-compose down
```

---

## Additional References

For more detailed information on these topics, check out these resources:

- [Official Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Installing Docker Compose](https://docs.docker.com/compose/install/)
- [Docker Networking Overview](https://docs.docker.com/network/)
- [Docker Compose CLI Commands](https://docs.docker.com/compose/reference/overview/)
