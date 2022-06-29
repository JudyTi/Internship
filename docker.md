# Table of Contents
- [0. Summary & Questions](#summary-questions)
- [1. Notes from Wael's video](#notes-from-waels-video)
   - [1.1 Concepts](#concepts)
   - [1.2 Building container](#building-container)
      - [1.2.1 Single Stage Build](#single-stage-build)
         - [Common Commands for image & container building](#other-command)
         - [Port Mapping](#port-mapping--p)
         - [Upload/Publish Container](#uploadpublish-container)   
      - [1.2.2 Multi Stage Build](#multi-stage-build)   
   - [1.3 Container Networking](#container-networking)
      - [1.3.1 `bridge`, `host`, and `none`](#docker-network)
      - [1.3.2 `veth` pair](#veth)
      - [1.3.3 `docker network inspect`](#docker-network-inspect)
      - [1.3.4 Create a bridge and attach container to the network](#create-a-bridge-and-attach-container-to-the-network)
- [2. Notes from YouTube video](#notes-from-youtube-video)
   - [2.1 Concepts](#concepts-1)
   - [2.2 Basic Commands](#basic-commands)
   - [2.3 Debug Container](#debug-container)
   - [2.4 Docker Networks](#docker-networks)
   - [2.5 Docker Compose](#docker-compose)
   - [2.6 Dockerfile](#dockerfile-1)
   - [2.7 Docker Repository](#docker-repository)

# Summary & Questions

- Container is a much more lightweight way than VM to package application with its configurations, dependencies, environments.  
   => can be shared easily between developers and operation

- Roadmap
   1. Have an application
   2. Build container for application     
      Base image: `docker pull [image]` from repo     
      Run new container for application: `docker run`    
      To use terminal for application: `docker exec`   
   3. Docker compose   
      Easier way to manage nultiple containers => similar to `Kubernetes`   
   4. Dockerfile    
      Knows where to start when run the image and start the container    
   5. Push image to remote 
      1. Build image
      2. Change tag (`docker tag`) so that it knows which repo to push to
      3. `docker push`
<br><br />
- What is Docker Network?   
   Networking is about communication among processes, and Docker’s networking is no different.    
   Docker networking is primarily used to establish communication between Docker containers and the outside world via the host machine where the Docker daemon is running.   
   Using `veth` pair

- What is the different between port and `veth` pair?  
   They work together to let data know where it should go.   
   => Make communication between Docker containers and the outside world

   The difference is in level of [protocol stack](https://en.wikipedia.org/wiki/Protocol_stack):
   + **port** is on **Transport layer**
   + **`veth` pair(interface)** is on **Link layer**

# [Notes from Wael's video](https://gitlab.es.f5net.com/cloud/labs/docker)
## Concepts
### Microservices Architecture
It's an architectural style that structures an application as a collection of services that are
- highly maintainable and testable
- loosely coupled
- independently deployable
- owned by small team

enables the rapid, frequent, reeliable delivery of large, complex applications and envolve its tech stack.

### Container
+ Difference between **Operating system(OS)** and **Kernel**:
   - **Operating system(OS)** 
      * Def: system program that runs on the computer to provide an interface to the computer user so that they can easily operate on the computer
      * The package of data and software that manages the resources of the system
      * In addition to the responsibilities of Kernel, OS is also responsible for **protection and security of the computer**
   - **Kernel**
      * Def: a system program that controls all programs running on the computer
      * The important program in the OS
      * **A bridge between software and hardware of the system**

Container is just a **process** on the machine uses **namespaces** and **control groups (cgroup)** to provide isolation  

+ Difference between **namespaces** and **control groups (cgroup)**:
   - **namespaces**
      * Def: a feature of the Linux kernel that partitions kernel resources such that one set of processes **sees** one set of resources while another set of processes sees a different set of resources
      * Limits **what you can see**
   - **control groups (cgroup)**
      * Def: a Linux kernel feature that limits, accounts for, and isolates the **resource usage** (CPU, memory, disk I/O, network, and so on) of a collection of processes
      * Limits **how much you can use**

Isolated fully contained application that runs with all its dependencies

### Docker
Relies on Linux kernel cgroup to isolate applications into namespaces.    
Runs on same kernel, shares on same kernel.   
But isolated and conatined in a namespace that seperated from the host.  

### Container vs. VM
**Container** shares the host operating system and kernel.   
**VM** abstracts hardware and having guest operating system.  

## Dockerfile
`FROM` : base container that will run and start building application   
`WORKDIR` : the root directory of container (similar to `pwd`)   
`COPY`   
`RUN`   
`CMD` : what process/going to run when docker container starts   

Dockerfile:
```Dockerfile
FROM golang:1.15-alpine
WORKDIR /myapp
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -o app .
CMD ["./app"]
```

## Building container
Build a simple app
#### Single Stage Build
1. Dockerfile
2. Build image: `docker build -t [image_name] .`   
   e.g. `docker build -t mycontainer:v0.0.1 .`
3. Run & test App

##### Other Command:
###### `docker history`
Show all layers that we are added to container
   * When build a container, almost all command in dockerfile lay on the top of file system
   * Layers can be used, cached -> unique ID, inspected, changed
   * **Disadvantage**: add size to container image   

e.g. `docker history [image_name]`

###### `docker image ls`
List the images in docker engine   
   * docker image is in repository after building
   
e.g. `docker image ls | grep mycontainer`

###### `docker run`
Runs the container (docker container starts running)
   * `--rm`: *By default a container’s file system persists even after the container exits.* This makes debugging a lot easier (since you can inspect the final state) and you retain all your data by default.    
   But if you are running short-term foreground processes, these container file systems can really pile up. If instead you’d like Docker to automatically clean up the container and remove the file system when the container exits
   * `-it`: interactive mode   

e.g. `docker run --rm -it --name [container_name] [image_name]`

###### `docker ps`
Lists the running container

e.g. `docker ps | grep mycontainer`

   `sudo ps aux | grep app`   
   `cat /proc/13434/cgroup`
###### `docker exec`
Interact with container
   * `-it`: interactively open to conatiner
   * Enters the container, puts user in the root of entry directory

e.g. `docker exec -it [container_name] /bin/sh`

   `ls`   
   `ps aux`  

Since this **container runs in a seperated and isolated network namespace**, so **can't it see on the host network**.  
-> Checking Ports Linux With `netstat`    
:heavy_multiplication_x: `netstat -antp | grep 8181` would fail on host

###### Get IP address
   * Inside container:    
   `ip a` (if have a shell or bash)
   * Outside container on host network:    
   `docker inspect`    
   works for images, containers, network outputs JSON formated informtaion
   e.g. `docker inspect mycontainer | grep -i ip`

- `curl [IP_address]:8181`
   * Without port mapping, it's only listening on the container IP address and own network namespace
   * Have to use IP address and port to access application

###### Port Mapping `-p`
If want to expose port outside container to the host   
   `docker run --rm -it --name [container_name] -p 9393:8181 [image_name]`   
   Maps container's port 8181 to host's port 9393, so now we can reach it through host

- `vim t`  
   Look at IP table rules on the host that are created by conatiner
-  Stop mapping, if **stop container** 

###### Stop Container
`[ctrl C]`: stop container as it runs in foreground with `-it`   
`docker stop [container_name]`: stop container as it runs in background as demon with `-d`

Until now, docker container is available in local docker engine image store, we want to **upload/publish** to allow access outside

###### Upload/Publish Container
DockerHub: create repository, publish container    
Need to change the docker image tag to match the DockerHub repository tag (append tag)
- `docker tag [image_name]:[version] [repository_tag]:[version]`
- `docker push [repository_tag]:[version]`  
   When push, docker knows where to push the image
- `docker login`    
   If repository is private

#### Multi Stage Build
Seperating **build process** and **run process**
- Minimize the size of docker image (as there are many layers)
- Starts from a base container (`builder`)  
Starts another runtime container copy the application

Dockerfile.multistage: 
```Dockerfile
FROM golang:1.15-alpine AS builder
WORKDIR /myapp
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -o app .

FROM alpine:latest
WORKDIR /code
COPY --from=builder /myapp/app .
CMD ["./app"]
```
=> One layer container with **small size & footprint**     
=> Less layer -> less size

Build image
`docker build -t mycontainer:multistage -f dockerfile.multistage .`

## Container Networking
Container: a process instead running directly on the host, just be contained in a namespace with all dependencies isolated from host      
=> start container -> start network namespaces    
=> doesn't have interfaces -> needs a way to communicate outside of network namespace  

The container was created when we run it.    
It has specific namespace that can attach interfaces => create veth pair

### Docker Network
##### Bridge Network
It attaches container to bridge (link layer)    
By default, when install Docker, all conatiners have link back to host   
`docker0` bridge is in the host network namespace     
=> Can move traffic in & out of the host to container

Look inside container, interface `eth0` (part of veth pair)

**veth pair** in Linux is a link between two namespaces. Like physical layer surface
- `brctl show docker0`
- `ip link show`   
   Show the veth interface on the host

##### Host Network
When run container, share its namespace of host with it      
`docker run --network=host`     
Use it when want to change network of host

##### None
`docker run --network=none`

### veth
##### Create veth pair
`ip link add hostside_veth type veth peer name test_veth`    
`ip link show | grep hostside_veth`

##### Move one veth pair from host to container
Change IP address inside container and set link with bridge or other configuration
- Get **process ID** of container: `docker inspect -f '{{.State.Pid}}' [container_name]`
- Get **network namespace** of container: `ip netns identify [process_ID]`
- Set peer interface inside container from host: `ip link set test_veth netns [network_namespace]`
- Set host: `ip link set hostside_veth master docker0`

Because veth is consider as one interface     
=> One side is down, the other side is down

### `docker network inspect`
Inspect networks and see what containers are connected to them

### Create a bridge and attach container to the network
Bridge network: all containers that are connected to the network share the same layer
- `docker network create [network_name] -d bridge --subnet "5.5.5.0/24"`
- `docker run --network=[network_name]`

Run container and attach it to the network

### Difference between default & user created network
User created network allow every contained added to it to get DNS entry   
=> able to ping the IP address of container: `ping [container_name]`

## Compose


# [Notes from YouTube video](https://www.youtube.com/watch?v=3c-iBn73dDE)
## Concepts
### What is a container?
- A way to package application with all the necessary dependencies
- Portable artifact, easily shared and moved around between developer and operation teams
- Make development and deployment efficiently

Technically,
- Layers of images
- Mostly, **Linux Base Image** because small in size
- Application image on the top

### Why container useful?
#### Application development
###### Before container
On local development environment, developers have to install same binaries of services
* installation process is different on different OS environment
* many steps can go wrong
###### After container
Don't need to install any services on own OS
* container has own isolated environment
* packaged with all needed configuration
* one command to install application
* can run same app with 2 different version at same time

#### Application deployment
###### Before container
* Developer will produce artifacts/databases/other services with instructions of how to install and configure those on server
* Operation will handle setting up the environment to deployment
 
Configuration needed on server -> dependency version conflict  
Textual guid of deployment -> misunderstandings

###### After container
Don't need to install any services on own OS
* Developer and operations work together to package the application in container
* No envrionment configuration needed on the server

Run a docker command that pulls container image in Repository

### Where do containers live?
Container Repository (private/public e.g. Docker Hub)

### Docker Image v.s. Docker Container
| Docker Image | Docker Container |
| ------------- | ------------- |
| The actual package | Actual start the application |
| Artifacts that can be moved around | Container environment is created |
| **Not running** | **Running** |

**Container** is a running environment of **Image**
- Container: file system, environment configuration, Application image
- Port: binded to talk to application running inside container
- Virtual file system

Everything in DockerHub are images

### Docker v.s. VM
##### Docker on OS level
Operating systems have 2 layers
| ------------- |
| 2. Layer: Application (run on Kernel layer) |
| 1. Layer: OS Kernel (communicate with hardware components) |
| Hardware (CPU and memory) |
They use same Linux Kernel, but implemented different application on top

##### Different level of absractions
Docker virtualize the application layer.   
VM virtualize the application layer and OS Kernel layer.
- Size: Docker image much smaller
- Speed: Docker conatiners start and run faster
- Compatibility: VM of any OS can run on any OS host   
So leads to a problem of Docker

##### Why Linux based docker containers don't run on Windows
If want to run a Linux application on Windows Kernel, it will fail  
Is Kernel compatable with the Docker images?

##### Restart container
When you restart the container, everything configured in that container application is gone    
=> data is lost    
=> **Docker Volumes for Data Presistence**

## Basic Commands
### Version and Tag
### Docker Commands

##### `docker pull`
Download image from repository  
> `docker pull [image_name]`  

##### `docker images`
Check all existing images
- Images have tags and versions
- No conatiner running yet

##### `docker run`  
Pull image, create container, and start image in a container -> Combine `docker pull` & `docker start`  
e.g. 
> `docker run redis`  
> `docker run redis:4.0`

- `-d`  
   Detached mode -> use terminal again
- `-p [host_port]:[container_port] [image_name]`    
   Bind container port and host port
- `--name [container_name] [image_name]`   
   Name the container. Without it, get default random name

##### `docker ps` 
List all running containers
- `-a`  
   List all running and stopped container

Shows [Container ID], [Image], [Command], [Created], [Status], [Ports], [Names]

##### `docker stop`  
Stop the container
- Attached mode `docker run`: `[ctrl C]`
- Detached mode `docker run -d`: `docker stop [container_ID]`

##### `docker start`
Start stopped container
- If want to restart container, need the [container_ID]  
  `docker start [container_ID]`


| `docker run` | `docker start` |
| ------------- | ------------- |
| Create new container from an image | Container already created, to restart a stopped container |
| Take image name w/o specific version | Take container name/ID |
| With image | With container|


##### [Ports]
Specify on which port the container is listening to the incoming requests  
**Container Port v.s. Host Port**   
How do we not have conflicts while both application are running on the same port?
- *Multiple containers* can run on host machine
- Laptop has only certain ports available
- :thumbsdown: *Conflicts* when same port on host machine
- :thumbsup: *No conflicts* when same port on different containers that binding with different ports on host machine

## Debug Container
##### `docker logs`
Get the logs information of container
> `docker logs [container_ID]/[container_name]` 

##### `docker exec`
Get the terminal of a running container
- `-it`   
   Interactive terminal
> `docker exec -it [container_ID] \bin\bash`   
   Enter the container's terminal as root user
>> `ls`  
>> `pwd`  
>> `env`  
>> `exit`


## Docker Networks
Deploying two containers in **same isolated Docker Network**, they can talk to each other using **just container name**, without localhost port no.    
**Application runs outside the Docker Network** needs to connect to them from outside from the host **using localhost port no.**  

Docker by default provide some network

##### `docker network ls`
List all auto-generated docker networks

##### `docker network create`
Create network
> `docker network create [network_name]`

##### Make container run in network
`docker run --net`   
e.g. create two container [mongoDB] and [mongoDB-express] in same docker network
```Command
# Start mongodb
docker run -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=password \
--name mongodb \
--net mongo-network \
mongo

# Start mongo-express
docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
--name mongo-express \
--net mongo-network \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
mongo-express
```

## Docker Compose
For running multiple Docker containers to make :point_up: easier   
A structured way to contain very normal common docker commands

### Compose file - `yaml`
e.g. docker run command v.s. mongo-docker-compose.yaml     
docker run command
```Command
# Start mongodb
docker run -d \
--name mongodb \                            => container name
-p 27017:27017 \                            => [host_port]:[container_port]
-e MONGO_INITDB_ROOT_USERNAME=admin \       => environment
-e MONGO_INITDB_ROOT_PASSWORD=password \
--net mongo-network \
mongo                                       => image name

# Start mongo-express
docker run -d \
--name mongo-express \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
--net mongo-network \
mongo-express
```
mongo-docker-compose.yaml (**Indentation** in `yaml` file is important)
```yaml
version: '3'                                => version of docker-compose
services:
   mongodb:                                 => container name
      image: mongo                          => image name
      ports:
         - 27017:27017                      => [host_port]:[container_port]
      environment:
         - MONGO_INITDB_ROOT_USERNAME=admin
         - MONGO_INITDB_ROOT_PASSWORD=password
   mongo-express:
      image: mongo-experss
      ports: 
         - 8081:8081
      environment:
         - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
         - ME_CONFIG_MONGODB_ADMINPASSWORD=password
         - ME_CONFIG_MONGODB_SERVER=mongodb
```
**Network configuration** is not in the docker compose    
:heavy_check_mark: Docker Compose takes care of creating a common network   
We don't need to create network and specify in which network those containers will run in

### Start docker containers using docker compose file
Docker Compose is already installed with Docker     
> `docker-compose -f [file_name] up`

Create default network `myapp_default`     
Create those two containers in that network    
=> logs of two containers are mixed because we are starting them at the same time

If there are two containers and one container **depend on** the other, can configure waiting logic in docker compose

### Stop docker containers
Stop, remove all containers created in the `yaml` file and remove network     
Use `docker stop` with names of every containers
> `docker-compose -f [file_name] down`

## Dockerfile
To build docker image from application, we have to copy the content of application (artifacts) into Dockerfile   
Blueprint for building images

Image Environment Blueprint
```
install node

set MONGO_INITDB_ROOT_USERNAME=admin
set MONGO_INITDB_ROOT_PASSWORD=password

create /home/app folder

copy current folder files to /home/app folder

start the app with: "node server.js"
```

Dockerfile
```Dockerfile
FROM node                                  => start by basing it on another image

ENV MONGO_INITDB_ROOT_USERNAME=admin \     => Optionally define environment variables
    MONGO_INITDB_ROOT_PASSWORD=password    => Alternative to define environmental variables in docker compose file

RUN mkdir -p /home/app                     => RUN -execute any Linux command
                                           => directory is created inside of the CONTAINER
COPY . /home/app                           => COPY -execute on the HOST machine

CMD ["node", "/home/app/server.js"]                  => CMD -execute entry point Linux command (only one)
                                           => Node is pre-installed because of base image
```
`RUN` command in Dockerfile will **apply to container environment, not host environment**     
Better to build **environmental variables** in **docker compose file**, can **change and overwrite** it in docker compose file **instead of rebuild the image**

### Build imgae from a Dockerfile
`docker build -t [image_name]:[tag_name] [dockerfile_location]`     

`docker images`: to see images

When you **adjust** the Dockerfile, you **must stop & delete container, and delete & rebuild the image**

`docker rm [continaer_ID]`: delete the container

`docker rmi [image_ID]`: once delete the container, can delete image

## Docker Repository
You save different tags/versions of same image

1. Have to login to private repo to authenticate => `docker login`
2. Build an image
3. Tag an image => `docker tag`    
   Naming in Docker registries: `registryDomain/imageName:tag`    
   DocherHub use shortcut that no need to include `registryDomain` when do `docker pull`     
   Original local image name, docker don't know where to push my application by default      
   => tag/rename the image to include the repo domain/address
4. `docker push`     
   Push image layer by layer

## Demo project
### Development
### Continuous Integration/Delivery
### Deployment
