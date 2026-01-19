# docker

Containerisation


HTML CSS JavaScript -> Frontend
Java or .NET -> Backend

J2EE -> Java Enterprise Edition
Servlets and JSP(Java Server pages) for viewing -> spring also depends on this
a single file -> frontend+backend
roboshop.war + dependencies(libraries) jar files
roboshop.ear (enterprise archeive)

A simple change in anything should again follow the release process


Monolithic
=============
Frontend -> Backend(Entire app should be in single programming language)
API
.war
Apache tomcat, JBoss Web Server

Microservices
=============
Catalogue, Cart, Shipping
API
every component in diff programming language
MBs
Containers

Booting time

Virtulisation			Containerisation
Costly					Cheaper
More boot time			less boot time
more size				less size
resource utilisation	resource utilisation is more
is less
More privacy			Less privacy
Less portable			More portable

sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

a docker group is created
only root user and users inside docker group will have acess to docker cli

sudo usermod -aG docker ec2-user

exit and login again

docker ps -> all running containers

images

AMI = OS + System Packages + App run time + App code + App libraries -> Amazon machine image

Docker image = Bare minimum OS(10Mb) + System Packages + App run time + App code + App libraries -> Amazon machine image
500Mb

Image is static, Server is running version of OS

if you run image you get container(running instance of image)

docker ps -> all running docker containers
docker ps -a -> all containers
docker pull nginx -> pulling the nginx image
docker images -> images in the local docker repo

docker pull + docker create ImageID + docker start ContainerID

docker run = docker pull + create + start

docker pull <image name>:<tag>  -> to pull the image form docker hub
docker create <image name> or <id> -> to create the container from image
docker start <container ID> -> to start the container

or 

docker run <imagename:tag> -> to pull + create container + start container
docker run -d <imagename:tag> -> to run background

docker stop containerID -> to stop the container
docker rm <container ID> -> to rm the container
docker rmi <containerID> -> to remove the container image

To change the port no. of the container: port binding

docker run -d -p <host port>:<container port> nginx

note: 1. host port means from this machine which port you want to assign to nginx.
      2. container port is from docker which port it need to ping the nginx container. 
ex: nginx ip is 8099 then cmd should be | docker run -d -p 8099:80 nginx  |  URL = http://IP:8099 | 

            -d -> detach | -p -> port binding

docker rm -f <container ID> -> to force remove the container without stopping

To login to the container:

docker exec -it <containerID> bash

:/usr/share/nginx/html -> where iis file is located

to get the container/image details: docker inspect <container ID>/<imageID>



