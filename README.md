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

docker ps -> running containers
docker pull nginx -> pulling the nginx image
docker images -> images in the local docker repo

docker pull + docker create ImageID + docker start ContainerID

docker run = docker pull + create + start
