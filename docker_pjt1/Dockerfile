FROM ubuntu:latest

#set working directory
WORKDIR /app

#copy the files from host file system to the image file system
COPY . /app/

#install the packages
RUN apt-get update && apt-get install -y python3 python3-pip

#Run command to start app
CMD ["python3", "app.py"]