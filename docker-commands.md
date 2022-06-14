# Docker notes:

## Container:

- A way to package applicaiton with all the necessary dependencies and configuration
- Portable artifact, easily sare and moved around
- Made of images. Layers of images stacked.
- Base of the most of the containers have mostly Linux base image, because small in size (Alpine or other Linux distribution)
- Application image on top of base image
- Containers live in a container repository

## Docker image:

- Image is the actual package
- Artifact that can be moed around

## Docker container:

- Container is when the image is pulled on the local machine and the application inside the image actually starts
- Container environment is created
- Container is a running environment for Image
- port binded: talk to application running inside of container

## CONTAINER Port vs HOST Port:

- Multiple containers can run on your host machine (laptop / PC)
- Your host machine has only certain ports avaialble
- Conflict when same port on host machine

> Docker hub is public repo container

## **docker run** vs **docker start** difference:

- **docker run** creates a new container from the image
- **docker start** works with container and not the images. It restarts the stopped container.

## Docker network:

- Docker creates it's isolated docker network where the containers are running in. When two containers are deployed in the same local network, they can talk to each other just using the container name without localhost, port number, etc.
- Applications running outside of the docker network connects to the docker network from the host using the localhost and port number.

# Commands:

## Basic commands:

```bash
# Pulls the images into the local machine
$ docker pull [image_name]

# Shows all the images pulled
$ docker images
```

> TAG is the version of the image

```bash
# Displays all the running containers
$ docker ps


# Shows all the containers which are running or not running
$ docker ps -a

# Displays the containers of that image only
$ docker ps -a | grep [image_name]
```

---

## Start and stop the containers:

```bash
# Stops the container
$ docker stop [container_id]

# Starts the container
$ docker start [container_id]

```

---

## **docker run** command:

```bash
# Starts the image in a container
$ docker run [image_name]
# Pulls the image and starts the container (docker pull + docker start)

# Runs postgres v10.10
$ docker run postgres:10.10


# -d runs the container in the detached mode
$ docker run -d [image_name]


# -p is used for port binding
$ docker run -p [host_port]:[container_port] [image_name]

$ docker run -p6000:6379 redis:latest


# --name gives container the name provided.
$ docker run --name [some_container_name] [image_name]

$ docker run -d -p6001:6370 --name redis-older redis:4.0
```

---

## Troubleshooting commands:

```bash
# Displays the logs the continer is producing.
$ docker logs [container_id or container_name]

$ docker logs redis-older


# Logs only the last part
$ docker logs [container_id] | tail


# Streams the log
$ docker logs [container_id] -f
```

```bash
# -it opens the terminal of the container.
$ docker exec -it [container_id or container_name] /bin/bash
# -it = interactive terminal
```

---

## Display available networks:

```bash
# Shows all the available network
$ docker network ls
```

---

## Deleting commands:

```bash
# Deleting the container
$ docker rm [container_id]

# Deleting the image
$ docker rmi [image_id]
```

# Example commands:

## Pulling the images

```bash
$ docker pull mongo

$ docker pull mongo-express
```

## Creating docker network

```bash
$ docker network create mongo-network
```

## Start mongodb

```bash
$ docker run -d \
>    -p 27017:27017 -d \
>    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
>    -e MONGO_INITDB_ROOT_PASSWORD=password \
>    --name mongodb \
>    --net mongo-network mongo \
>    mongo

# -e setup the environment variable
# --net run mongodb in mongo-network created
```

## Start mongo-express

```bash
# Connects mongo-express with the mongodb container
$ docker run -d \
> -p 8081:8081 \
> -e ME_CONFIG_MONGODB_ADMINUSERNAME=mongoadmin \
> -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
> --net mongo-network \
> --name mongo-express \
> -e ME_CONFIG_MONGODB_SERVER=mongodb \
> mongo-express
```

# Docker compose

## Creating a docker compose file

The name of the file created is `mongo.yaml`.

```yaml
version: '3' # version of docker compose
services:
  mongodb: # container name
    image: mongo
    ports:
      - 27017:27017 # HOST:CONTAINER
    environment:
      - MONGO_INITDB_ROOT_USERNAME=mongoadmin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8080
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongoadmin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
# Containers talk to each other using the container name
# Docker compose takes care of creating a common network
```

## Starting docker container using docker compose file

```bash
$ docker-compose -f mongo.yaml up
# -f file name
# up Starts all the containers in the .yaml file
```

## Stopping docker container using docker compose file

```bash
$ docker-compose -f mongo.yaml down
# down Shut downs all the containers
# Also removes the network. Network is recreated when restarted
```

# Dockerfile

Dockerfile is a blueprint for building docker images. The name must be `Dockerfile`.

## Creating a Dockerfile

```Dockerfile
# All the commands in the dockerfile will apply to the container environemnt
# None will affect the host environment

# Start by basing it on another image
# This dockerfile is based on node
# node is installed inside of image
FROM node:13-alpine

# Configuring environment variables is optional. This is already done in docker compose file
# It is better to define environment variables extrenally in a docker compose file
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

# RUN - execute any Linux command
# This directory is created inside of the container and not on the host
RUN mkdir -p /home/app

# This COPY command executes on the host
# COPY [source] [destination]
# Copies the file in the host inside of the container image in the path /home/app
COPY . /home/app

# Executes the entrypoint linux command inside of the container
CMD["node", "/home/app/server.js"]

# CMD = entrypoint command
# You can have multtiple RUN commands but CMD is just once
```

## Build docker image from Dockerfile

```bash
$ docker build -t [image_name]:[tag] [location_of_Dockerfile]

$ docker build -t my-app:1.0 .
```

> **_Note:_** When you adjust the **Dockerfile**, you **MUST** rebuild the image.  
> The old image cannot be overwritten

# Image naming in Docker registries

> registryDomain/imageName:tag

    $ docker pull [registryDomain]/[imageName]:[tag]

    $ docker tag my-ap:1.0 [registryDomain]/[imageName]:[tag]

    $ docker push [registryDomain]/[imageName]:[tag]

# Docker Volumes

Used for data persistance in docker. Used for

- Database
- Other stateful applications

When do we need Docker Volumes?

- Container runs on a host
- Container has Virtual File System where data is stored
- No persistance
- Data is gone when restarting or removing the container and starts from fresh state

What is a Docker Volumne?

- Host has physical file system
- Plug physical file system path to the container file system
- Folder in physical host file system is mounted into the virtual file system of Docker
- Data gets automatically replicated
- When container writes to it's file system, it gets replicated in the host file system and vice-versa

## Types of docker volumes:

There are different types of docker volumes and different ways of creating them.

1. Host Volumes

   - You decide where on the host file system the reference is made
   - i.e. which folder on the host file system you mount into the container

   ```bash
   docker run -v [host_dir]:[container_dir]
   ```

2. Anonymous Volumes

   - Create a volume by just be referencing the container directory
   - You don't specify which directory on the host should be mounted. That's taken care by Docker.
   - Directory is automatically created by Docker under
     > /var/lib/docker/volumes/random-hash/\_data
   - For each container, a folder is generated that gets mounted automaticaaly to the container

   ```bash
   docker run -v [container_dir]
   ```

3. Named Volumes

   - Improvement on Anonynous Volumes. It specifies the name of the folder on the host file system.
   - You can reference the volume by name.
   - Should be used in production.

   ```bash
   docker run -v name:[container_dir]
   ```

## Docker Volumes in docker-compose

Example using **_Named Volume_**:

```yaml
version: '3' # version of docker compose
services:
    mongodb: # container name
        image: mongo
        ports:
            - 27017:27017 # HOST:CONTAINER
        volumes:
            - db-data:/var/lib/mysql/data # [reference_name]:[path_in_the_container]

    mongo-express:
        image: mongo-express
        ...

volumes:
    db-data
```

The name of the file is `docker-compose.yaml`.

```yaml
version: '3' # version of docker compose
services:
  mongodb: # container name
    image: mongo
    ports:
      - 27017:27017 # HOST:CONTAINER
    environment:
      - MONGO_INITDB_ROOT_USERNAME=mongoadmin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db # [host-volume-name]:[path-inside-of-the-container]

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongoadmin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

volumes:
  mongo-data:
    driver: local
```

## Docker Volume Locations

### WIndows:

> C:\ProgramData\docker\volumes

### Linux:

> /var/lib/docker/volumes

### Mac:

> /var/lib/docker/volumes
