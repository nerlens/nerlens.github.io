---
layout: post
title:  "Basic Docker Intro"
date:   2019-01-14 12:42:30 -0501
categories: blog
author: Stavros
---

## Resources

- [Official Docs](https://docs.docker.com)
- [Lab](https://labs.play-with-docker.com/)
- [Docker Hub](https://hub.docker.com/)

## Installation

- Instructions on [Docker Docs](https://docs.docker.com) site.

## Docker Hub

- Docker hub is a 'registry' of offical repositories
- Image format - registry.docker.io/namespace/repository:tag
- Official repositories such as "Alpine" are in root namespace
  - No need to specify registry or namespace
- Only mandatory field is 'repository'
  - If you do not specify 'registry':
    - Docker will assume 'Docker Hub':
  - If you not not specify namespace:
    - Docker will assume 'root'
  - If you do not specify 'tag':
    - Docker will assume 'latest'

## Basic informational commands

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

## Running containers

### Note: Docker will run the container as long as the application inside it is running

```powershell
# Run container along with a command
# These containers will be ephemeral
# The container will stop once the command completes
docker container run alpine:3.6 uptime
docker container run alpine ps
docker container run alpine uname -a
docker container run alpine date


# List containers
docker ps       # show running containers
docker ps -l    # show latest created container
docker ps -n 2  # show last 'n' number of containers
docker ps -a    # show ALL containers

# Run interactive Alpine Linux shell
docker container run -ti alpine sh

```