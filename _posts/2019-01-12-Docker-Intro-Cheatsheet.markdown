---
layout: post
title:  "Basic Docker Intro"
date:   2019-01-14 12:42:30 -0501
categories: blog
author: Stavros
---

This post is intended as a Docker quick-start guide.  **DISCLAMER**: Much of the content below is derived from the following resources:

- [Just Enough Docker to be Dangerous - Free Udemy Course](https://www.udemy.com/just-enough-docker/)
- [School of Devops](https://www.schoolofdevops.net/)

## Resources

- [Official Docs](https://docs.docker.com)
- [Lab](https://labs.play-with-docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Portainer](https://www.portainer.io/)

## Installation

Instructions for installing Docker on a particular OS are on [Docker Docs](https://docs.docker.com) site.

***Note:*** When using Docker for Windows with Hyper-V, a base VM will be created in the follwoing location.

```powershell
C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks\MobyLinuxVM.vhdx
```

## Basic informational commands

***Note:*** Run Docker commands using PowerShell or Linux shell

```powershell
# Run sample application - Smoke-test
docker run hello-world

# Client and server version
docker version

# Client version
docker -v

# Info about the Docker daemon (engine)
docker system info

# Stream events running in Docker daemon
# You can run this in a separate shell to monitor live events
docker system events

# Help
docker
docker COMMAND --help

```

## Docker Hub

- [Docker Hub](https://hub.docker.com/) is a 'registry' of offical repositories
- Image format - *registry.docker.io/namespace/repository:tag*
- Official repositories such as [alpine](https://hub.docker.com/_/alpine) are in root namespace
  - No need to specify registry or namespace
- Only mandatory field is **'repository'**
  - ***If you do not specify 'registry':*** Docker will assume 'Docker Hub'
  - ***If you not not specify namespace:*** Docker will assume 'root'
  - ***If you do not specify 'tag':*** Docker will assume 'latest'

***Note:*** [alpine](https://hub.docker.com/_/alpine) is a nice lightweight Linux image

## Running containers

- Docker will *run* the container **as long as the application inside it is running**
- You can assign a name to the container using the **--name** parameter
- If no name is assigned, a random name will be given (*this can be changed later*)

***Note:*** Launching *'docker run'* or *'docker container run'* commands will **always** create a new container

```powershell
# Run container along with a command
# These containers will be ephemeral
# The container will stop once the command completes
docker container run alpine:3.6 uptime
docker container run alpine ps
docker container run alpine uname -a
docker container run alpine date
docker container run --name 'mycontainer' alpine date

# List containers
docker ps       # show running containers
docker ps -l    # show latest created container
docker ps -n 2  # show last 'n' number of containers
docker ps -a    # show ALL containers

```

## Persisting containers

```powershell
# Run interactive programs
docker container run -it alpine sh
docker container run -it ubuntu bash

# Long running container - requires the 'd' switch as well
docker container run -idt alpine sh
```

## Mounting Local Volumes in a container

```powershell
# Use -v to mount a volume
docker container run --name repos -it -v c:/_repositories:/repositories alpine sh
```

## Operations on containers

***Note:*** Refer to containers using name or (full or partial) ID

```powershell
# rename a container
docker rename sharp_jones mycontainer

# Logs - view current activity
docker logs mycontainer
docker logs mycontainer -f # constant update

# Run commands inside a container
docker exec mycontainer uptime
docker exec mycontainer ps
docker exec mycontainer la -la
docker exec mycontainer apk update
docker exec mycontainer apk add python3
docker exec mycontainer apk add vim

# Connect to running container and run program
docker exec -it mycontainer sh
docker exec -it mycontainer python3

# Detailed infornamtion about a container
docker inspect mycontainer # Returns JSON data -- file is stored in container - "LogPath"

# Copy file into container
docker container cp test.py mycontainer:/home

# Interact with copied file
docker exec mycontainer python3 /home/test.py

# Stop container gracefully (if this doesn't work a kill command will be sent)
# Can pass multiple containers - separated by a space
docker stop mycontainer

# Start a stopped containers
docker start mycontainer

# Remove container
# Can pass multiple containers - separated by a space
docker rm mycontainer # container cannot be running
docker rm mycontainer -f # remove even if running

```

## Access applications from outside

***Note:*** [nginx](https://hub.docker.com/_/nginx) is a web server image on [Docker Hub](https://hub.docker.com/)

```powershell

# Run nginx web server and map ports using -P (uppercase P)
docker container run --name 'mywebserver' -idt -P nginx

# Note the PORT mapping using the 'docker ps' command
# You can interact with the webserver using the URL http://localhost:(port) (ex. localhost:32768)

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
40abcce990ce        nginx               "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes        0.0.0.0:32768->80/tcp   mywebserver

# Specify custom port mapping using -p (lowercase p)
docker container run --name 'mywebserver' -idt -p 1234:80 nginx
```

***Note:*** [ghost](https://hub.docker.com/_/ghost) is a CMS. *The default port is 2368.*

```powershell
#create new container
docker container run -itd --name ghost -p 3001:2368 ghost:alpine

#start stopped container
docker start ghost
```

***Note:*** [Jenkins](https://hub.docker.com/r/jenkins/jenkins) is an automation engine written in Java.  *The default port is 8080.*

```powershell
# Run Jenkins
docker container run -idt --name 'Jenkins' -P jenkins/jenkins:lts-alpine

# Launch sh to aquire initial password
docker exec -it Jenkins sh
```
Retrieve password using terminal.

```sh
cd /var/jenkins_home/secrets/
cat initialAdminPassword
```

## Managing images

Create dev environments using persistent Docker containers.

```powershell
# List local images
docker images
docker image ls

# Show histiry for a particular image
docker image history ghost:alpine

# Pull image and run a persistent container using host networking
docker container run -idt --name dev-ubuntu --net host ubuntu bash

# Make changes to container
docker exec -it dev-ubuntu bash

apt-get update
apt-get install net-tools
apt-get install iptools-ping

# Changes will persist (even through reboots on underlying system)
docker stop dev-ubuntu
docker start dev-ubuntu
```

## Use [Portainer](https://www.portainer.io/) to Manage Docker Environments

### Installation

```powershell
# Create volume to persist data
docker volume create portainer_data

# Allows container to manage docker daemon
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

Browse to [http://localhost:9000](http://localhost:9000) to access the Portainer setup page and create an admin password

- **Manage containers -** add, remove, connect to console, monitor
- Catalog of App Templates for one-click deployments
- Connect to multiple instances of remove Docker servers


![Portainer](./images/Portainer_1.jpg)

## Docker Compose

- docker-compose.yml
- run commands converted into code
- provides the ability to launch complete stack of interconnected services
- provides automatic service discovery

***Example -*** [Prometheus stack](https://github.com/vegasbrianc/prometheus)

```powershell
# Clone repository containing docker-compose.yml file
git clone https://github.com/vegasbrianc/prometheus.git

# Browse to folder containing docker-compose.yml and application files
cd ./prometheus

# Launch application stack
docker-compose up -d # -d for detach mode (automaticall uses it modes)

# list applications
docker-compose ps

# Stop or remove applications
docker-compose stop
docker-compose down

```

## Build an image manually with Docker commit

***Example -*** [Facebooc](https://github.com/schoolofdevops/facebooc)

### Prepare the image

```powershell
# Clone example repository
git clone https://github.com/schoolofdevops/facebooc.git

# Run container per application instructions
docker container run -idt --name fb -p 16000:16000 ubuntu bash

# Install prerequisite utilities
docker exec -it fb bash

apt-get update
apt-get install -yq build-essential make libsqlite3-dev sqlite3

# Verify build utility locations
which gcc
which make

# From host machine, copy application source files into container
docker cp facebooc fb:/opt/

# Reconnect to container
docker exec -it fb bash

# compile code
cd /opt/facebooc
make all

# Run application
bin/facebooc

# Browse to localhost:16000 and view output
Listening on port 16000.

Sun Jan 13 18:12:23 2019 172.17.0.1 GET / 200
Sun Jan 13 18:12:23 2019 172.17.0.1 GET /static/css/normalize.css 200
Sun Jan 13 18:12:23 2019 172.17.0.1 GET /static/css/main.css 200

# View changes that have been made to container
docker container diff fb
```

### Commit and push the image to Docker Hub

```powershell
# Create an image
docker container commit fb christopherstavros/facebooc_demo:v1

# View image details
docker image ls
docker image history 08a9b52a0a03

# Push image to Docker Hub
docker login

docker image push christopherstavros/facebooc_demo:v1

```

## User a Dockerfile to create an image

***Example*** [facebooc Dockerfile](https://github.com/schoolofdevops/facebooc/blob/docker/Dockerfile)

```dockerfile
FROM  ubuntu


WORKDIR /opt/facebooc

RUN  apt-get update &&  \
     apt-get install -yq build-essential make git libsqlite3-dev sqlite3 


COPY . /opt/facebooc

RUN  make all 

EXPOSE 16000

CMD "bin/facebooc"

```

### Image build process, using Dockerfile

```powershell
# Checkout branch containing Dockerfile
cd .\facebooc\
git checkout docker

# Build image (can use 'docker build' or 'docker image build')
# Run from project dir (containing Dockerfile)
docker image build -t christopherstavros/facebooc_demo:v2 .

# View image history
docker image history christopherstavros/facebooc_demo:v2

# Push image to Docker Hub
docker image push christopherstavros/facebooc_demo:v2
```

![Image push](./images/Image_push1.jpg)


