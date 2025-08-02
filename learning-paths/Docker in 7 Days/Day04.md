# Day 4: Managing Containers

**Objective:** Gain proficiency in managing containers.

## Activities:
1. **Practice starting, stopping, and restarting containers.**
2. **Learn about container lifecycle management and how to use commands like `docker start` and `docker exec`.**
3. **Explore container logs and resource management.**

---

### 1. Practice Starting, Stopping, and Restarting Containers

Containers are lightweight, portable, and can run any application that has been packaged with its dependencies. Managing the lifecycle of containers is a fundamental skill for any developer or operations engineer.

#### Starting a Container
To start a container, use the `docker run` command. This command not only starts the container but also creates it if it does not already exist.

---

    docker run --name my_container -d nginx

- \`--name my_container\`: Assigns a name to the container.
- \`-d\`: Runs the container in detached mode (in the background).
- \`nginx\`: The name of the image to use.
---
#### Stopping a Container
Stopping a running container is crucial for managing resources and ensuring that applications do not consume unnecessary system resources.


    docker stop my_container


#### Restarting a Container
Restarting a container can be useful when you need to apply configuration changes or updates.


    docker restart my_container


---

### 2. Container Lifecycle Management

Understanding the full lifecycle of a container, from creation to removal, allows you to manage applications effectively.

#### Starting a Stopped Container
To start a container that has been previously stopped, use the `docker start` command.


    docker start my_container


#### Executing Commands Inside a Running Container
To run a command inside a running container, you can use the `docker exec` command. This is useful for troubleshooting, running updates, or executing any command line operations inside the container.


    docker exec -it my_container /bin/bash

- \`-it\`: Combines \`-i\` (interactive) and \`-t\` (pseudo-TTY), allowing you to interact with the container’s terminal.
- \`/bin/bash\`: Opens a bash shell inside the container.

---

### 3. Exploring Container Logs and Resource Management

Logging and resource management are critical for monitoring and optimizing container performance.

#### Viewing Container Logs
To view the logs of a container, use the `docker logs` command. This helps in debugging and monitoring the application running inside the container.


    docker logs my_container


You can also follow the logs in real-time using the \`-f\` (follow) flag.


    docker logs -f my_container


#### Resource Management
Efficient resource management ensures that containers do not consume more than their fair share of system resources. Docker allows you to limit CPU and memory usage of containers.

- **Limiting CPU Usage:**
 

    docker run --name my_container --cpus="1.5" -d nginx
  
  This command limits the container to 1.5 CPU cores.

- **Limiting Memory Usage:**
 

    docker run --name my_container -m 512m -d nginx
 
  This command limits the container to 512MB of RAM.

---

### Additional Resources

- **Docker Documentation**: Comprehensive guide and reference for Docker commands and best practices.
  - [Docker Docs](https://docs.docker.com/)

- **Docker Cheatsheet**: A quick reference for common Docker commands.
  - [Docker Cheatsheet](https://dockerlabs.collabnix.com/docker/cheatsheet/)

- **Play with Docker**: An online playground for Docker.
  - [Play with Docker](https://labs.play-with-docker.com/)

- **Docker Tutorials**: Interactive tutorials to learn Docker.
  - [Katacoda Docker Tutorials](https://www.katacoda.com/courses/docker)

By the end of this day, you should be comfortable with managing container lifecycles, viewing logs, and managing resources for Docker containers. 
