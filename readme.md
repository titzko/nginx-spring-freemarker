# Overview
This project is a proof of concept that demonstrates how to use Freemarker templates in a Spring Boot application together with Nginx, which serves as a reverse proxy. The idea is that the Freemarker templates use server-side includes (SSI), which should be resolved by Nginx.

# Prerequisites
Before getting started, make sure you have the following prerequisites:

- Docker installed on your local machine
- Basic familiarity with Docker and containerization
- Basic familiarity with Spring Boot and Freemarker

# Instructions

Follow these instructions to get the project up and running:


1. Find your host's internal IP address:

   > ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}'

2. Adjust the following part of the `docker_nginx/default.conf`:

```plaintext
# Docker nginx reverse proxy proxy_pass entry
location /sample {
   ssi on;
   proxy_pass http://192.168.178.31:8080/sample;
}
```

- Replace `192.168.178.31` with your internal IP address.
- Note that `localhost` won't work, since `localhost` (or `127.0.0.1`) is the loopback address, which means that requests are looped back to the same device.
- When you run Nginx inside a Docker container, it is isolated from the host machine. If you use `localhost` inside the Nginx container, it refers to the container itself and not the host machine.
- To allow the Nginx reverse proxy container to access your local device, you should use the host's internal IP address.

3. Go into docker_nginx dir and create the Docker image:

   > sudo docker build -t nginx-reverse-proxy .

4. Start the Docker container:

   > docker run -d -p 80:80 nginx-reverse-proxy

5. Start the Tomcat from the SpringBoot-App

6. Access `http://localhost/sample` (serverd by nginx) and compare it to `http://localhost:8080/sample` (served by the spring-app without nginx)

# Troubleshoot

1. Get the container id via `docker ps`. 
2. Check the logs: `docker logs <container-id>`
3. Check if the spring-boot app is running correctly: `http://localhost:8080/sample`
4. Go into the container and check if the config file is correct
 - `docker exec -it <container-id> bash`
 - `cat /etc/nginx/conf.d/default.conf ` should match the default.conf file