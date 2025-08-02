# Day 7: Advanced Topics and Best Practices

## Objective:
Explore advanced topics and best practices in Docker.

## Activities:
1. **Learn about Docker Compose for Multi-Container Applications**:
    - Introduction to Docker Compose
    - Writing a `docker-compose.yml` file
    - Managing multi-container applications

2. **Explore Best Practices for Writing Efficient Dockerfiles and Managing Images**:
    - Structuring Dockerfiles
    - Reducing image size
    - Leveraging caching

3. **Review Security Considerations When Running Containers**:
    - Running containers as non-root users
    - Limiting container capabilities
    - Regularly updating images

---

## Docker Compose for Multi-Container Applications

### Introduction to Docker Compose
Docker Compose is a tool for defining and running multi-container Docker applications. It allows you to use a YAML file to configure your application’s services, networks, and volumes.

#### Installing Docker Compose:

    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose --version


### Writing a `docker-compose.yml` File
A `docker-compose.yml` file defines the services, networks, and volumes for your application.

#### Example `docker-compose.yml`:

    version: '3'
    services:
    web:
    image: nginx
    ports:
    - "8080:80"
    db:
    image: mysql
    environment:
    MYSQL_ROOT_PASSWORD: example


### Managing Multi-Container Applications
With Docker Compose, you can start, stop, and manage multi-container applications easily.

#### Starting the Application:

    docker-compose up -d


#### Stopping the Application:

    docker-compose down


---

## Best Practices for Writing Efficient Dockerfiles and Managing Images

### Structuring Dockerfiles
A well-structured Dockerfile is crucial for building efficient images.

#### Example Dockerfile:

# Use an official Python runtime as a parent image
    FROM python:3.8-slim
    
    # Set the working directory
    WORKDIR /app
    
    # Copy the current directory contents into the container at /app
    COPY . /app
    
    # Install any needed packages specified in requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Make port 80 available to the world outside this container
    EXPOSE 80
    
    # Define environment variable
    ENV NAME World
    
    # Run app.py when the container launches
    CMD ["python", "app.py"]


### Reducing Image Size
Minimize the size of your Docker images to improve build times and reduce the attack surface.

#### Tips to Reduce Image Size:
- Use multi-stage builds
- Choose a minimal base image
- Clean up temporary files

### Leveraging Caching
Docker caches the results of the stages in your Dockerfile to speed up subsequent builds.

#### Example:
    
    # Use a minimal base image
    FROM alpine:3.12
    
    # Install dependencies
    RUN apk add --no-cache python3-dev
    
    # Add application source code
    COPY . /app
    
    # Install any needed packages
    RUN pip install --no-cache-dir -r /app/requirements.txt
    
    # Run the application
    CMD ["python3", "/app/app.py"]


---

## Security Considerations When Running Containers

### Running Containers as Non-Root Users
Avoid running containers as root to minimize security risks.

#### Example:
    
    # Use an official Node.js runtime as a parent image
    FROM node:14
    
    # Create and change to the app directory
    WORKDIR /usr/src/app
    
    # Copy application dependency manifests to the container image
    COPY package*.json ./
    
    # Install production dependencies
    RUN npm install --only=production
    
    # Copy local code to the container image
    COPY . .
    
    # Create a non-root user and switch to it
    RUN useradd --user-group --create-home --shell /bin/false appuser
    USER appuser
    
    # Run the web service on container startup
    CMD [ "node", "server.js" ]

### Limiting Container Capabilities
Use Docker’s security options to limit the capabilities of your containers.

#### Example:


    docker run --cap-drop=ALL --cap-add=NET_ADMIN --cap-add=SYS_TIME my_secure_image


### Regularly Updating Images
Ensure your images are regularly updated to include the latest security patches.

#### Example:

    docker pull nginx:latest


---

## Additional Resources:
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/)

---
