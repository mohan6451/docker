FROM almalinux:9
run dnf install nginx -y
CMD ["nginx", "-g", "daemon off;"]
LABEL author="Mohan" \
      version="3.0"
EXPOSE 8088/tcp