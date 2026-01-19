FROM nginx
RUN rm -rf /usr/share/nginx/html/index.html
COPY index.html /usr/share/nginx/html/index.html
LABEL author="Mohan" \
      version="3.0"
EXPOSE 8088/tcp