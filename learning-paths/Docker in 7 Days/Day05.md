
# Day 5: Networking in Docker

## Objective:
Understand Docker networking concepts and gain proficiency in managing container networks.

## Activities:
1. **Learn about Different Network Types in Docker**:
    - Bridge Network
    - Host Network
    - Overlay Network

2. **Practice Creating Custom Networks and Connecting Containers to Them**:
    - Create a custom bridge network.
    - Connect multiple containers to the custom network.
    - Explore network isolation.

3. **Explore How Containers Communicate Within a Network**:
    - Container-to-container communication.
    - Accessing containers from the host machine.
    - Using DNS within Docker networks.

---

## Docker Network Types

### Bridge Network
The bridge network is the default network type in Docker. Containers connected to the same bridge network can communicate with each other directly.

#### Creating a Bridge Network:

    docker network create --driver bridge my_bridge_network


### Host Network
The host network allows a container to share the host's networking namespace, gaining direct access to the host's network interfaces.

#### Running a Container with Host Network:

    docker run --network host nginx


### Overlay Network
The overlay network is used for communication between containers across multiple Docker daemons. It is typically used in Docker Swarm or Kubernetes setups.

#### Creating an Overlay Network:

    docker network create --driver overlay my_overlay_network


---

## Creating Custom Networks and Connecting Containers

### Creating a Custom Bridge Network:

    docker network create my_custom_network


### Connecting Containers to the Custom Network:

    docker run -d --name container1 --network my_custom_network nginx
    docker run -d --name container2 --network my_custom_network nginx


### Exploring Network Isolation:
To ensure network isolation, Docker creates a separate namespace for each network. Containers on different networks cannot communicate with each other unless explicitly connected.

---

## Container Communication

### Container-to-Container Communication:
Containers within the same network can communicate using their container names as hostnames.

#### Example:

    docker exec -it container1 ping container2


### Accessing Containers from the Host Machine:
To access a container from the host machine, you need to map the container port to a host port using the \`-p\` flag.

#### Example:

    docker run -d -p 8080:80 nginx


### Using DNS within Docker Networks:
Docker provides an internal DNS service to containers, allowing them to resolve each other's names.

#### Example:

    docker run -d --name container3 --network my_custom_network nginx
    docker exec -it container1 ping container3


---

## Additional Resources:
- [Docker Networking Overview](https://docs.docker.com/network/)
- [Docker Network Drivers](https://docs.docker.com/network/#network-drivers)
- [Managing Docker Networks](https://docs.docker.com/network/network-tutorial-standalone/)

---

