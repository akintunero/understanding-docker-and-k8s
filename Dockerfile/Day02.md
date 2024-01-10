
# Day 2: Intermediate Dockerfile Instructions

On this day, we will delve deeper into Dockerfile instructions and explore how to use them effectively to build more robust and flexible Docker images.

## Objective

- Understand the differences between `CMD` and `ENTRYPOINT`.
- Learn how to use environment variables with `ENV`.
- Explore the use of `WORKDIR` for setting working directories.
- Understand how to use `VOLUME` for data persistence.

## Topics

### CMD vs. ENTRYPOINT

Both `CMD` and `ENTRYPOINT` define the default command that runs when a container starts, but they serve slightly different purposes:

- **`CMD`**: Sets default commands and/or parameters, which can be overridden by command-line arguments when starting a container.

  ```
  CMD ["python", "app.py"]
  ```

- **`ENTRYPOINT`**: Configures a container to run as an executable. It is not overridden by command-line arguments unless you use the `--entrypoint` flag.

  ```
  ENTRYPOINT ["python", "app.py"]
  ```

**Combination Example**:

You can combine both to set default parameters while allowing flexibility:

  ```
  ENTRYPOINT ["python"]
  CMD ["app.py"]
  ```

### Using ENV for Environment Variables

The `ENV` instruction sets environment variables inside the container. These can be used to configure applications dynamically.

  ```
  ENV APP_ENV=production
  ENV APP_DEBUG=false
  ```

### WORKDIR Instruction

The `WORKDIR` instruction sets the working directory for any subsequent instructions in the Dockerfile. If it doesn’t exist, it will be created.

  ```
  WORKDIR /app
  ```

### VOLUME for Data Persistence

The `VOLUME` instruction creates a mount point with specified path and marks it as holding externally mounted volumes from native host or other containers.

  ```
  VOLUME /data
  ```

## Activities

### Modify an Existing Dockerfile

1. Take the Dockerfile from Day 1 and modify it to include environment variables using `ENV`.
2. Set a working directory using `WORKDIR`.

  ```
  FROM python:3.9-slim
  WORKDIR /app
  COPY . /app
  RUN pip install --no-cache-dir -r requirements.txt
  # Set environment variables
  ENV FLASK_ENV=production
  ENV FLASK_DEBUG=0
  EXPOSE 5000
  CMD ["python", "app.py"]
  ```

### Experiment with CMD vs. ENTRYPOINT

1. Create a Dockerfile that uses both `CMD` and `ENTRYPOINT`.
2. Build and run the image, experimenting with overriding commands at runtime.

  ```
  FROM ubuntu:20.04
  ENTRYPOINT ["echo"]
  CMD ["Hello, World!"]
  ```

Run with different commands:

  ```
  docker run <image-name>
  docker run <image-name> "Different message"
  ```

## Additional References

For more detailed information on these topics, consider exploring the following resources:

- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Understanding Docker CMD vs ENTRYPOINT](https://phoenixnap.com/kb/docker-cmd-vs-entrypoint)
- [Docker ENV Instruction](https://docs.docker.com/engine/reference/builder/#env)
- [Docker WORKDIR Instruction](https://docs.docker.com/engine/reference/builder/#workdir)
- [Managing Data in Containers](https://docs.docker.com/storage/)

