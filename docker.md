# [Notes from Wael's video](https://gitlab.es.f5net.com/cloud/labs/docker)
## What is a container?
It's just a **process** on the machine uses **namespaces** and **control groups (cgroup)** to provide isolation

Isolated fully contained application that runs with all its dependencies

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
#### Before container
On local development environment, developers have to install same binaries of services
    * installation process is different on different OS environment
    * many steps can go wrong
#### After container
Don't need to install any services on own OS
    * container has own isolated environment
    * packaged with all needed configuration
    * one command to install application
    * can run same app with 2 different version at same time

### Application deployment
#### Before container
    * Developer will produce artifacts/databases/other services with instructions of how to install and configure those on server
    * Operation will handle setting up the environment to deployment
    
Configuration needed on server -> dependency version conflict

Textual guid of deployment -> misunderstandings

#### After container, don't need to install any services on own OS
    * Developer and operations work together to package the application in container
    * No envrionment configuration needed on the server

Run a docker command that pulls container image in Repository

## Where do containers live?
Container Repository (private/public e.g. Docker Hub)
