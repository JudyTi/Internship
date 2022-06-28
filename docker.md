# Notes from Wael's video
## Microservices Architecture
It's an architectural style that structures an application as a collection of services that are
- highly maintainable and testable
- loosely coupled
- independently deployable
- owned by small team

enables the rapid, frequent, reeliable delivery of large, complex applications and envolve its tech stack.

## Container
It's just a **process** on the machine uses **namespaces** and **control groups (cgroup)** to provide isolation  
Isolated fully contained application that runs with all its dependencies

## Docker
Relies on Linux kernel cgroup to isolate applications into namespaces.    
Runs on same kernel, shares on same kernel.   
But isolated and conatined in a namespace that seperated from the host.  

## Container vs. VM
**Container** shares the host operating system and kernel.   
**VM** abstracts hardware and having guest operating system.  

## Dockerfile
`FROM` : base container that will run and start building application   
`WORKDIR` : the root directory of container (similar to `pwd`)   
`COPY`   
`RUN`   
`CMD` : what process/going to run when docker container starts   

## Building container
Build a simple app
#### Single Stage Build
1. Dockerfile
2. Build image: `docker build -t [image_name] .`
  * `docker build -t mycontainer:v0.0.1 .`
3. Run & test App

Other Command:
- `docker history`
   * Show all layers that we are added to container
   * When build a container, almost all command in dockerfile lay on the top of file system
   * Layers can be used, cached -> unique ID, inspected, changed
   * **Disadvantage**: add size to container image   
   e.g. `docker history [image_name]`
- `docker image ls`
   * docker image is in repository after building
   * List the images in docker engine   
   e.g. `docker image ls | grep mycontainer`
- `docker run`
   * Runs the container (docker container starts running)
   * `--rm`: *By default a container’s file system persists even after the container exits.* This makes debugging a lot easier (since you can inspect the final state) and you retain all your data by default.    
   But if you are running short-term foreground processes, these container file systems can really pile up. If instead you’d like Docker to automatically clean up the container and remove the file system when the container exits
   * `-it`: interactive mode   
   e.g. `docker run --rm -it --name [container_name] [image_name]`
- `docker ps`
   * Lists teh running container   
   e.g. `docker ps | grep mycontainer`   
   `sudo ps aux | grep app`   
   `cat /proc/13434/cgroup`
- `docker exec`
   * Interact with container
   * `it`: interactively open to conatiner
   * Enters the container, puts user in the root of entry directory
   e.g. `docker exec -it [container_name] /bin/sh`   
   `ls`   
   `ps aux`  

Since this **container runs in a seperated and isolated network namespace**, so **can't it see on the host network**.  
-> Checking Ports Linux With `netstat`    
`netstat -antp | grep 8181` would fail on host

- Get IP address
   * Inside container:    
   `ip a` (if have a shell or bash)
   * Outside container on host network:    
   `docker inspect`    
   works for images, containers, network outputs JSON formated informtaion
   e.g. `docker inspect mycontainer | grep -i ip`

- `curl [IP_address]:8181`
   * Without port mapping, it's only listening on the container IP address and own network namespace
   * Have to use IP address and port to access application

Port Mapping `-p`
* If want to expose port outside container to the host
* `docker run --rm -it --name [container_name] -p 9393:8181 [image_name]`   
Maps container's port 8181 to host's port 9393, so now we can reach it through host

- `vim t`  
   Look at IP table rules on the host that are created by conatiner
-  Stop mapping, if stop container 


#### Multi Stage Build

## Container Networking

## Compose

# [Notes from YouTube video](https://www.youtube.com/watch?v=3c-iBn73dDE)
## What is a container?
- A way to package application with all the necessary dependencies
- Portable artifact, easily shared and moved around between developer and operation teams
- Make development and deployment efficiently

Technically,
- Layers of images
- Mostly, **Linux Base Image** because small in size
- Application image on the top

## Why container useful?
### Application development
##### Before container
On local development environment, developers have to install same binaries of services
* installation process is different on different OS environment
* many steps can go wrong
##### After container
Don't need to install any services on own OS
* container has own isolated environment
* packaged with all needed configuration
* one command to install application
* can run same app with 2 different version at same time

### Application deployment
##### Before container
* Developer will produce artifacts/databases/other services with instructions of how to install and configure those on server
* Operation will handle setting up the environment to deployment
 
Configuration needed on server -> dependency version conflict  
Textual guid of deployment -> misunderstandings

##### After container
Don't need to install any services on own OS
* Developer and operations work together to package the application in container
* No envrionment configuration needed on the server

Run a docker command that pulls container image in Repository

## Where do containers live?
Container Repository (private/public e.g. Docker Hub)
