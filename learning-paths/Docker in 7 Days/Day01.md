
# Day 1: Introduction to Docker


---
## Activities:

### 1. **Read about Docker's Foundational Concepts and Benefits**


Docker is a deployment platform that enables developers to develop, package, and run applications in containers. They are small and isolated—packages which contain all the executable code, libraries, and other dependencies necessary for an application to execute.

**Key Concepts:**

- **Containerization**: The process of packaging an application and its dependencies into a container, which can run reliably in different computing environments.
- **Images**: Read-only templates used to create Docker containers. They contain the application's code, libraries, dependencies, and runtime environment.
- **Containers**: Running instances of Docker images that include everything needed to execute an application.
- **Dockerfile**: A script containing a series of commands that Docker uses to build an image.
- **Docker Hub**: A registry where Docker images are stored, shared, and managed. It can be public or private.

**Benefits of Docker:**
- Portability: Write once, run anywhere.
- Consistency: Identical environments for development, testing, and production.
- Efficiency: Containers use fewer resources than traditional virtual machines.
- Scalability: Easily scale your application horizontally by deploying multiple containers.
- Isolation: Each container runs independently, which ensures security and isolation.

---

### 2. **Install Docker Desktop on Your Machine**

Docker Desktop is a local application that allows you to build, run, and manage Docker containers on your machine.

**Steps to Install Docker Desktop:**
1. Visit the [Docker Desktop official page](https://www.docker.com/products/docker-desktop) and download the installer for your operating system (Windows, macOS, or Linux).
2. Run the installer and follow the installation instructions.
3. After installation, verify that Docker is running by opening a terminal or command prompt and typing:

```bash
docker --version
```

You should see output indicating the installed version of Docker, e.g., `Docker version 20.10.8, build 3967b7d`.

**Common Installation Issues:**
- Ensure that virtualization is enabled in your machine's BIOS.
- If you encounter networking issues, Docker Desktop may need to be granted access through your firewall or VPN settings.

---

### 3. **Explore the Docker Interface and Familiarize Yourself with the Dashboard**

After installing Docker Desktop, open the application. The Docker dashboard provides a user-friendly interface for managing containers and images.

**Key Areas to Explore:**
- **Containers/Apps**: View and manage running containers.
- **Images**: Explore Docker images installed locally on your machine.
- **Volumes**: Storage used by containers to persist data.
- **Networks**: Manage the different networks Docker containers use for communication.

Take time to familiarize yourself with the controls, like starting, stopping, and deleting containers, pulling images from Docker Hub, and creating new containers.

---

### 4. **Watch Introductory Videos or Tutorials**

Watching video tutorials can help solidify your understanding of Docker. Here are some recommended resources:

- **[Docker 101 Tutorial](https://www.docker.com/101-tutorial)**: This official tutorial covers Docker basics, including running a simple application in a container.
- **YouTube Docker Playlist**: Search for beginner-friendly videos on YouTube to complement the Docker 101 Tutorial.
- **Pluralsight Docker Course**: If you have access to Pluralsight, they offer a range of Docker courses, from beginner to advanced levels.

Take notes during the tutorials and feel free to experiment by running sample commands.

---

## Sample Docker Commands:

Here are a few Docker commands that you can use after installation to explore Docker further:

1. **Check Docker version**:
    ```bash
    docker --version
    ```

2. **Pull an image from Docker Hub**:
    ```bash
    docker pull hello-world
    ```

3. **Run a Docker container**:
    ```bash
    docker run hello-world
    ```

4. **List running containers**:
    ```bash
    docker ps
    ```

5. **List all containers (including stopped ones)**:
    ```bash
    docker ps -a
    ```

6. **List all Docker images**:
    ```bash
    docker images
    ```

7. **Stop a running container**:
    ```bash
    docker stop <container_id>
    ```

8. **Remove a container**:
    ```bash
    docker rm <container_id>
    ```

9. **Remove a Docker image**:
    ```bash
    docker rmi <image_id>
    ```

10. **Remove all stopped containers**:
    ```bash
    docker container prune
    ```

---

## Additional Documentation and Resources

To deepen your understanding of Docker, it is recommended to refer to the following official documentation and community resources:

1. **[Docker Documentation](https://docs.docker.com/)**: The official Docker docs provide extensive and well-structured information about all features, commands, and best practices.
2. **[Docker Hub](https://hub.docker.com/)**: Explore pre-built images and learn about image management.
3. **[Docker Cheat Sheet](https://dockerlabs.collabnix.com/docker/cheatsheet/)**: A quick reference guide for commonly used Docker commands.
4. **[Docker GitHub Repository](https://github.com/docker)**: If you're interested in contributing to Docker's open-source codebase, this is the place to go.
5. **[Docker In 5 Minutes | Docker Explained |](https://www.youtube.com/watch?v=59I0-bMiaLY)**:  Docker Tutorial For Beginners | Simplilearn

---