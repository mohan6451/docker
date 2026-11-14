My understanding after watching, exploring and practising Docker:

docker: is the orchestration platform used to build, ship and run the application. by using the base machine resource with the help of docker engine. 

docker engine  places the role to communicate between the containers and host kernel.

dockerfile:
    File that contains the step by step instructions on how to build the environment, Includes the base OS, system packages, appcode, app runtime, app libraries, metadata.
    name of the file can be dockerfile default or we can place the custom name.

docker image: is the static template build from the dockerfile.

docker container: is the live, running instances of the docker image. 

docker-compose: its in yaml format. its a tool for defining and running the multi containers applications. 
    To run an application it has frontend, backend(multiple services) and databases. everytime we need to run the build, run commands. they is chance for human errors and lot of human effort to make the services run. so instead we can give all the instructions and place them in a file. so that we can make the services running with just by running this file. it helps to build the images, services, volumes, networks.

        docker compose up -d --build 
        docker compose ps
        docker compose logs  - we can also use docker compose logs <container> to check the logs of the container
        docker compose down  - to stop and remove the containers build
.dockerignore: 
    the file which tells docker which files and folders to ignore when building the docker images. just need to provide the files or folders location. 
    When we run docker build command, the Docker CLI gathers everything in your project folder (called build context) and sends it to the Docker daemon. CLI filters out any matching files and directories before sending the context to the daemon.

    it helps to build the images faster, smaller, cleaner and better security (.env, secrets)

    you can check the .dockerignore file from this path for reference.


Docker hub:

where we can get the base images, custom images and we can also push our custom build images make them private or public simply as a registry to store the images as the library 

Docker Hub is a cloud-based container registry that serves as a central library for storing, sharing, and distributing Docker images.


Volume: container are epheramals as we know so to we need the vol to be mounted to not loose the data. and mount back when the container restarts. volumes keep data safe outside the container lifecycle.

types: named and unnamed
named: (-v name:/app/data)
    managed directly by docker in the dedicated storage area on the host machine (docker volume inspect <volume-name>)
unnammed:
bind mounts: (-v ./src:./app/src)
    maps an exact file or folder path from host machine directly into container
tmpfs mounts: stores temporary data strictly in the host machine's RAM (never written to disk). Best for sensitive session tokens or high-speed temporary caches.



docker networks:
isolated virtual network that allows containers to communicate securely with each other, the host machine and the outside internet.

network drivers:
bridge (default): 
    Creates a private internal network on your host machine. Containers attached to a custom bridge network can find and talk to each other automatically using their container names (via Docker's built-in DNS).

host:  
    Removes network isolation between the container and the host machine, letting the container use the host's IP address and ports directly.

none:
    Completely disables all networking for a container, isolating it from both other containers and external networks.

docker network create <name>: Creates a custom bridge network.

dockerfile building instructions:

FROM <image:tag>    : sets the base image
WORKDIR <path>      : set the default working directory
RUN <command>       : execute the shell command during image build stage
COPY <source> <destination>     :copies the file/folder to the container's file system (build context)
ADD <source> <destination>    : same as copy instruction + download the context from internet and can un tar the .tar files
ARG <key:value>     :build time variables at the build stage, not available inside container
ENV <key:value>     :to provide variables at availbe while the container runs
LABEL <key:value>       :to provide the metadata 
EXPOSE <port>       : provide only info. about the open ports
ONBUILD <instruction>       : adds the trigger instruction to an image that executes automatically later when another dockerfile uses this image as its base image.  
USER <username>     : switches the active user inside the container for all subsequents commands for container security
ENTRYPOINT [ "executable", "param" ]        : similar to CMD but here the executable is dedicated, that can't be overridden at runtime
CMD [ "executable", "param" ]       :default startup command that needs to run the container

note: If both ENTRYPOINT and CMD are defined, CMD acts as default arguments passed to ENTRYPOINT

commands:
docker pull + docker create ImageID + docker start ContainerID

docker run = docker pull + create + start

docker pull <image name>:<tag>  -> to pull the image form docker hub
docker create <image name> or <id> -> to create the container from image
docker start <container ID> -> to start the container

commands we use most common:

docker build -t <image:version> <location of the file>      : to build the image
docker run -d <image:version>       :to run the container                                          
docker run -d --name <container-name> -p <host-port>:<container-port> --network <network-name> -v <host-path-or-volume>:<container-path> <image>:<tag>
docker run -d --name my-nginx -p 8080:80 --network app-net -v ./html:/usr/share/nginx/html nginx:1.27
docker exec -it <container> bash        :to login the container
docker ps       :to show the all running containers
docker images       :to get all the availabe images
docker inspect <container/vol/network/images>      :to retrive the metadata or details about
docker volume ls        :to list all the vol. 
docker network --help     :to get details about the network
docker rm <container_id/name>      :to remove the container
docker rmi <image>      :to remove the image
docker stop <container>     :to stop the container


to debug
docker run --rm -it <image> bash : creates, starts & run a temporary container and automatically deletes as soon as you exit.
docker logs <container_id>      :to get the logs of the container
docker compose logs     :to get the logs of compose run command 
docker compose logs --tail=50 <container>       :to get the top 50 logs of the container


1. Docker Engine

Definition: Docker Engine is a client-server application that builds, runs, and manages lightweight software containers by sharing the host machine's operating system kernel.
• Core Components:
	• Docker CLI (docker): The command-line interface used to enter commands (e.g., docker build, docker run).
	• REST API: The communication bridge that translates CLI commands into instructions for the background server.
	• Docker Daemon (dockerd): The background service that manages Docker objects, including images, containers, networks, and volumes.
• Role with the Kernel:
	• During Setup & Shutdown: Docker Engine acts as an active manager, instructing the host kernel (via containerd and runc) to create isolated workspaces (namespaces) and set hardware limits (cgroups).
	• During Execution: Once started, a container runs as a standard isolated process and communicates directly with the host kernel via system calls, ensuring near-native performance without translation overhead.

2. Docker Compose

Definition: Docker Compose is an orchestration tool used to define, configure, and run multi-container applications using a single YAML configuration file (compose.yaml).
• Problem It Solves: Real-world applications consist of multiple components (frontend, backend microservices, and databases). Running individual docker build and docker run commands for each service requires heavy manual effort and increases the risk of human error.
• How It Works: All setup instructions are written once inside compose.yaml. Running a single command (docker compose up) automatically builds images, creates private networks, attaches storage volumes, and starts all services together.
• Key Benefits:
	• Automation: Replaces repetitive manual terminal commands with a single, repeatable workflow.
	• Consistency: Eliminates configuration typos across ports, environment variables, and volumes.
	• Built-in Networking: Services communicate automatically using their service names instead of hardcoded IP addresses.

3. .dockerignore

Definition: A .dockerignore file is a plain-text configuration file that specifies which files and directory paths Docker must exclude when building an image.
• How It Works: Before running docker build, the Docker CLI filters out matching files from the build context so they are never sent to the Docker Daemon or copied into the image.
• Path Syntax: You can specify individual file names (config.json), nested file paths (backend/config/local.json), or entire folders (node_modules/, frontend/dist/), all evaluated relative to the project root.
• Key Benefits:
	• Faster Builds: Excludes heavy local folders (such as node_modules or venv).
	• Smaller Images: Keeps unnecessary logs, tests, and temporary files out of the final image.
	• Improved Security: Prevents sensitive files (such as .env secrets, SSH keys, and .git history) from being baked into image layers.

4. Docker Hub

Definition: Docker Hub is a cloud-based container registry that serves as a central library for storing, sharing, and distributing Docker images.
• Core Functions:
	• Pull Base & Official Images: Download trusted, pre-built images for operating systems, runtimes, and databases (e.g., ubuntu, node, postgres) to use as a foundation for custom apps.
	• Push Custom Images: Upload your own custom-built images to the cloud for backup, team collaboration, or server deployment.
	• Access Control: Manage repositories as Public (open for anyone to download) or Private (restricted to authorized users and teams).


