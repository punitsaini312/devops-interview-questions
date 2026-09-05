### 1. **What is Docker?**

- **Answer:** Docker is a platform that helps you create, deploy, and run applications in containers. Containers are lightweight and package up the application with all the dependencies, making it easy to run anywhere.
- Docker is an open-source platform designed to automate applications' deployment, scaling, and management within lightweight, portable containers

### 2. **What is a Docker container?**

- **Answer:** A Docker container is a runnable instance of a Docker image. It contains everything needed to run an application, such as code, libraries, and system tools.
- A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.

### 3. **What is a Docker image?**

- **Answer:** A Docker image is a read-only template with instructions for creating a container. It includes the application and all dependencies needed to run it.
- A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries, and settings.

### 4. **What is Dockerfile?**

- **Answer:** A Dockerfile is a text file that contains a series of instructions to build a Docker image. It defines the environment and steps needed to create the image.

### 5. **How do you create a Docker container?**

- **Answer:** You create a Docker container using the command `docker run`. For example:
This runs an Nginx container and maps port 80 on the host to port 80 on the container.
    
    ```
    docker run -d -p 80:80 nginx
    
    ```
    

### 6. **What is Docker Compose?**

- **Answer:** Docker Compose is a tool that allows you to define and run multi-container Docker applications. It uses a `docker-compose.yml` file to configure your application’s services and containers.

### 7. **How do you check running Docker containers?**

- **Answer:** You can check the running containers using the command:
    
    ```
    docker ps
    
    ```
    

### 8. **What is the difference between a container and a virtual machine?**

- **Answer:** A virtual machine (VM) runs a complete operating system on top of a hypervisor, while a container shares the host OS kernel and is more lightweight, using fewer resources.

### 9. **How can you delete a Docker container?**

- **Answer:** To delete a stopped container, you use:
    
    ```
    docker rm <container_id>
    
    ```
    

### 10. **What is the purpose of the `docker pull` command?**

- **Answer:** The `docker pull` command is used to download a Docker image from a registry (like Docker Hub) to your local system.

### Docker Volume:

1. **What is a Docker volume, and why is it used?**
    - **Answer:** A Docker volume is a storage system used to save data created or needed by Docker containers. It stores the data separately from the container itself, so the data remains even if the container stops or is deleted. Volumes are useful for sharing files between containers or between the container and the host system
2. **How do you create and use a Docker volume?**
    - **Answer:** You can create a Docker volume using the command:To use it in a container, you run:This mounts the volume `my_volume` to the `/data` directory inside the container.
        
        ```lua
        lua
        Copy code
        docker volume create my_volume
        
        ```
        
        ```arduino
        arduino
        Copy code
        docker run -d -v my_volume:/data nginx
        
        ```
        
3. **How do you inspect a Docker volume?**
    - **Answer:** You can inspect a volume to get information like where it's stored on the host using:
        
        ```php
        php
        Copy code
        docker volume inspect <volume_name>
        
        ```
        
4. **How do you remove unused Docker volumes?**
    - **Answer:** You can remove all unused volumes (dangling volumes) with:
        
        ```
        Copy code
        docker volume prune
        
        ```
        

### Docker Network:

1. **What are Docker networks, and why are they important?**
    - **Answer:** Docker networks allow containers to communicate with each other. They are important for isolating containers, ensuring security, and enabling communication between services in different containers.
2. **What are the different types of Docker networks?**
    - **Answer:** Docker provides three main types of networks:
        - **bridge:** The default network, used for containers on the same host to communicate.
        - **host:** Allows a container to share the host's network stack.
        - **none:** No network, the container is isolated from all networks.
        You can also create custom networks using:
        
        ```lua
        lua
        Copy code
        docker network create my_network
        
        ```
        
3. **How do you connect a running container to a Docker network?**
    - **Answer:** You can connect a running container to a network using:
        
        ```php
        php
        Copy code
        docker network connect <network_name> <container_name>
        
        ```
        
4. **How do you list all available Docker networks?**
    - **Answer:** You can list all the available networks with:
        
        ```bash
        bash
        Copy code
        docker network ls
        
        ```
        
5. **How does DNS work in Docker networks?**
    - **Answer:** Docker provides built-in DNS resolution for containers. Each container can resolve the names of other containers in the same network by their container names.

### Docker Compose:

1. **How do you scale services in Docker Compose?**
    - **Answer:** You can scale services by using the `-scale` option:
        
        ```php
        php
        Copy code
        docker-compose up --scale <service_name>=<number_of_instances>
        
        ```
        
2. **How do you stop and remove all containers defined in a `docker-compose.yml` file?**
    - **Answer:** You can stop and remove all containers by running:
        
        ```
        Copy code
        docker-compose down
        
        ```
        

### Dockerfile:

1. **What is the purpose of the `CMD` and `ENTRYPOINT` commands in a Dockerfile?**
    - **Answer:** Both `CMD` and `ENTRYPOINT` define the default executable that runs when a container starts. `CMD` provides default arguments to the `ENTRYPOINT`, but `ENTRYPOINT` cannot be overridden at runtime, whereas `CMD` can be.
2. **What is the difference between `COPY` and `ADD` in a Dockerfile?**
    - **Answer:** Both commands copy files from the host to the container. `COPY` is simpler and only copies files, while `ADD` can also extract local tar archives and download files from URLs.

### Docker Security:

1. **How can you secure Docker containers?**
    - **Answer:** Docker containers can be secured by:
        - Using the principle of least privilege (limiting access and permissions).
        - Running containers with non-root users.
        - Regularly updating Docker images and applying security patches.
2. **What is Docker Content Trust (DCT)?**
    - **Answer:** Docker Content Trust is a feature that enables the verification of the integrity and authenticity of Docker images. It uses digital signatures to ensure the image hasn't been tampered with.

### Miscellaneous:

1. **What is the difference between `docker stop` and `docker kill`?**
    - **Answer:** `docker stop` gracefully stops a container by sending the `SIGTERM` signal, giving the container time to terminate properly, while `docker kill` forcefully stops the container by sending the `SIGKILL` signal immediately.
2. **What is the difference between `docker exec` and `docker attach`?**
    - **Answer:** `docker exec` allows you to run commands inside a running container, whereas `docker attach` lets you connect to the container's standard input, output, and error streams.

### Q. what is the difference between CMD and ENTRYPOINT?

ans: Both ENTRYPOINT and CMD can be used to execute as your start command or they can whenever somebody runs docker command, docker run so both CMD and ENTRYPOINT server as your starting command but the only difference is ENTRYPOINT is something that you can’t change.

Whereas CMD is something that is configurable.

```bash
ENTRYPOINT ["python3"]
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
    
              You can make any change in the CMD.
    ENTRYPOINT ["python3"]
CMD ["manage.py", "runserver", "0.0.0.0:8052"]
```

### Q. What is distroless image?

ans: A distroless image is a very minimalistic and lightweight docker image that will have only the runtime environment if you are choosing a distroless image called a python distroless image, so in this image, if you want to run a basic find command. you’ll get error saying “find command/executable is not found” Even you can’t perform command like “ls”. The only purpose of this environment is to just execute the slash app or python application.  

### Q. What is multistage docker image?

ans: A multistage Docker image is a way to create smaller and more efficient Docker images by using multiple `FROM` statements in a single Dockerfile. Each `FROM` statement starts a new stage, allowing you to copy only the necessary files from one stage to the next. This method is especially useful for compiling code in one stage (with all the build tools) and then copying only the final output to the next stage, reducing the size of the final image by excluding unnecessary build dependencies.

COPY --from=build /app/dist /usr/share/nginx/html

NOTE: Docker use virtual ethernet for the netwoking in the docker.

docker stats —no-stream 
(we use this command to check the live streaming of the resources usage)
