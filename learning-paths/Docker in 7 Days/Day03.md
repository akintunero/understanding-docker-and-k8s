# **Day 3: Working with Docker Images - A Beginner’s Guide**

### **Objective**:
Understand how to create and manage Docker images.

---

## **What is a Docker Image?**

A Docker image is a read-only template that contains everything needed to run a container. It includes the application code, libraries, environment variables, and runtime. Docker images are built in layers, where each layer represents a set of changes or commands that are executed.

### **Why Use Docker Images?**
- **Portability**: Images are portable, allowing them to run consistently across different environments.
- **Reusability**: Images can be reused across different projects, reducing redundancy.
- **Versioning**: Each image layer is cached, and if a layer doesn't change, Docker will reuse it, improving build performance.

---

## **Step 1: Dockerfiles and How They are Used to Build Images**

A **Dockerfile** is a text document containing a series of instructions to build a Docker image. You specify what base image to use, what files to copy, which commands to run, and more.

### **Common Dockerfile Instructions**:
- `FROM`: Specifies the base image.
- `COPY` or `ADD`: Copies files from the local machine into the Docker image.
- `RUN`: Runs a command inside the image.
- `CMD` or `ENTRYPOINT`: Defines the command that runs when the container starts.
- `EXPOSE`: Exposes the port the container listens on.

---

## **Step 2: Creating a Simple Dockerfile**

Let’s create a simple `Dockerfile` that builds an image for a Python web app.

### **Steps**:

1. **Create a project directory** and navigate into it:

```bash
mkdir myapp
cd myapp
```

2. **Create a `Dockerfile`** using your favorite text editor, and add the following content:

```Dockerfile
# Use an official Python runtime as the base image
FROM python:3.8-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install the required Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define the command to run the app
CMD ["python", "app.py"]
```

### **Explanation**:
- `FROM python:3.8-slim`: Starts from a small Python image.
- `WORKDIR /app`: Sets `/app` as the working directory inside the container.
- `COPY . /app`: Copies the current directory on your local machine into the container.
- `RUN pip install ...`: Installs any necessary dependencies for your application.
- `CMD ["python", "app.py"]`: Specifies the command that runs when the container starts.

---

## **Step 3: Building the Image Using `docker build`**

Now that you have a Dockerfile, you can build an image from it.

### **Build the Image**:

Run the following command in the directory where your `Dockerfile` is located:

```bash
docker build -t my-python-app .
```

**Explanation**:
- `docker build`: Tells Docker to build an image.
- `-t my-python-app`: Tags the image with the name `my-python-app`.
- `.`: Specifies the build context (in this case, the current directory).

Docker will go through the `Dockerfile`, execute each instruction, and build the image layer by layer.

You can see your newly built image by running:

```bash
docker images
```

---

## **Step 4: Exploring Image Layers and Caching**

Docker images are composed of multiple layers. Each instruction in the Dockerfile creates a new layer in the image.

### **How Image Layers Work**:
- **Layers are Reused**: If a layer hasn’t changed since the last build, Docker uses the cached version to speed up the build process.
- **Ordered by Commands**: Each command in the Dockerfile, like `COPY` or `RUN`, creates a new layer.
- **Minimize Changes**: Try to structure your Dockerfile to minimize layer changes (e.g., install dependencies early, avoid copying files that change frequently).

### **Viewing Image Layers**:

To see the layers of an image, you can use the `docker history` command:

```bash
docker history my-python-app
```

This command will display all the layers in your image, showing the size and command used to create each layer.

---

## **Best Practices for Dockerfile Creation**

1. **Use Official Base Images**: Start from official images like `python`, `ubuntu`, etc.
2. **Keep Dockerfiles Simple**: Each layer adds to the size, so keep Dockerfiles lean and avoid unnecessary steps.
3. **Take Advantage of Caching**: Reuse layers by ordering commands to minimize rebuilds (e.g., copy dependencies first).
4. **Optimize with Multi-stage Builds**: If building and running a large application, you can use multi-stage builds to reduce the final image size.

---

## **Summary of Key Commands**

| **Command**               | **Description** |
|---------------------------|-----------------|
| `docker build -t <image_name> .` | Builds a Docker image from a Dockerfile. |
| `docker images`            | Lists all available Docker images. |
| `docker history <image>`   | Shows the history of an image, including layers. |
| `docker tag <source_image> <target_image>` | Tags an image with a new name or version. |

---

## **Additional Resources**

- **[Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best_practices/)**: Learn best practices for creating efficient Dockerfiles.
- **[Docker Documentation](https://docs.docker.com/)**: Find detailed guides on building and managing Docker images.
- **[Docker Hub](https://hub.docker.com/)**: Discover pre-built images for a variety of applications.

By the end of Day 3, you should be comfortable creating your own Dockerfile, building images, and understanding the concept of image layers and caching.
