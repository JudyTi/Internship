# Resources
[Kubernetes' documentation](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)   
[YouTube video](https://www.youtube.com/watch?v=X48VuDVv0do)   
[K8s Architecture](https://zhuanlan.zhihu.com/p/382229383)
[K8s' concepts from Aliyun](https://developer.aliyun.com/article/718798)

# [Notes from Wael's & YouTube video](https://www.youtube.com/watch?v=X48VuDVv0do)
## Concepts
### Basic of K8s
##### Definition
Open source **container orchestration tool** to **manage containerized applications** in different **deployment environment**(physical, virtual, cloud, hybrid) developed by Google

A way to abstract multiple hardware resources into a single cluster and provide hardware resources to workloads/Pods of containers.    
An orchestration management system of multiple hardware resources relies on OS (e.g. Linux)

##### Why useful
Trend to Microservices   
=> increasng usage of containers   
=> make K8s useful as a container orchestration tool

##### What features does it offer
- High availablility: no downtime
- Scalability: high performance
- Disaster recovery: backup and restore

### Core Components of K8s
- **Node**: a simple server (physical/VM)

<img src="./images/Node.png" alt=node width="300"/> <img src="./images/Node-1.png" alt=node width="500"/>

##### Abstraction of container
- **Pod**
    * Basic component/smallest deployable unit of K8s
    * Abstraction over container (create *running environment or a layer* on top of container).   
        => only interact with K8s layer
    * On Worker Node
    * Can have one or more containers
    * All containers share the same network namespace (1 interface called `eth0` by default)
    * Usually 1 application per Pod
    * Each Pod gets its own IP address to communicate

##### Communication
If a container die and re-created, new IP address will assigned  
=> inconvenience as need to adjust everytime   
=> Service is used

- **Service** - Communication
    * *Static/Permanent IP address* that can attached to each Pod
    * *Load balancer* - virtual IP address     
        Distributed systems and containers will have replica of our application on different Node/servers to prevent if one server crashed     
        Catch the request and forward it to less busy Pod
    * Kube proxy setup and maintain network rules
    * Lifecycle of Pod and Service are NOT connected   
        => If a container die and re-created, IP address will stay  
        => onvenience as don't need to adjust everytime
    * **Externel Service**
        + App should be accessible through browser
            => Need Externel Service
        + a service that opens communication from externel sources (e.g. app)
        + URL is not practical: [IP address of Node]:[port number of the service]   
            => Ingress
    * **Internel Service**
        + a service that not open to public requests (e.g. database)

Endpoint - the local Pod IP address for the Service -> where to forward traffic
Create an object -> label object -> find object through adding selector to definition and choose the same label


- **Ingress** - Route traffic into cluster
    * URL with domain name
    * Public requests -> ingress -> service

##### External configuration
Need to do complex adjustment if the URL of database changed (usually need to adjust inside app file)
- **ConfigMap**
    * External configuration to application
    * Usually contain configuration data (e.g. URL of database)
    * Credentials (username/password) of database can be put into ConfigMap, but not secure   
        => so avoid to do that  
        => Secret
    * Connected to the Pod
- **Secret**
    * Just like ConfigMap
    * Use to store secret data
    * In base64 encoded format, not plain-text format
    * Connected to the Pod

##### Data presistence
If database container/Pod restarted, data would be gone
- **Volumes**
    * Attach storage on a local machine / remote storage outside of the K8s cluster  
        => All data presisted
    * Storage as a hard drive plugged intp the K8s cluster  
        => K8s doesn't manage data presistence

##### Pod Blueprint with replicating mechanism
Distributed systems and containers will have replica of our application on different Node/servers
- **Deployment**
    * For STATELESS applications Pod
    * Blueprint for application Pod  
        => How many replicas I want for application    
        => Create Demployments     
        => Scale up/down the number of replicas
    * Abstraction of Pods
- **StatefulSet**
    * For STATEFUL applications/database Pod, as database has a state which is data    
        => can't get replica using Deployment   
        => data consistency
    * Blueprint for application Pod  
        => How many replicas I want for application
        => Create Demployments    
        => Scale up/down the number of replicas
    * Abstraction of Pods
    * Harder than Deployment  
        => DB are often hosted outside of K8s cluster

**Workload**   
A combination of a computing object(ReplicaSet) and the controlling object

### Architecture of K8s
Reources: CPU | RAM | Storage    
Master servers -> less resources    
Worker servers actually do the job -> more resources

![K8s_arch](/images/K8s_archtecture.png)

Freely add new servers
To add new Master/Node server:
1. get new bare server
2. install all the master/worker node processes
3. add it to the cluster   
=> :thumbsup: Infinitely increase power & resources of K8s clusters

##### Worker Node server/Node
Compute resources to run containers, provision storage, networking

Many Nodes in K8s cluster, communicating via Services
- Node - Worker machine in K8s cluster
    + Each Node has multiple Pods on it
    + 3 processes must be installed on every Node
        1. Container Runtime - Docker
        2. Kubelet - Node manager   
            Interacts with both the container and node   
            Starts the pod with a container inside  
            Assigns resources from node to the container
        3. Kube proxy   
            Forwarding requests from Services to Pods   
            Handles networking rules
    + Worker Nodes - Cluster server that acually do the work

##### Master server/Control Plane
Services that manage state

<img src="./images/master.png" alt=master width="800"/>

How to interact with this cluster?
How to:
- schedule Pod?
- monitor?
- re-schedule/re-start Pod?
- join new Node?    

=> **Master server**

- 4 processes run on every master node
    1. API Server  
        + When user want to deploy a new application on K8s cluster, interact with it using Client
        + Cluster Gateway which gets initial requests of any updates into cluster and queries from cluster
        + Cluster Gatekeeper for authentication
        + some request -> API Server -> validates requests -> other processes -> schedule Pods
        + Only 1 entrypoint to cluster   
            => Good for Security
    2. Scheduler
        + schedule new Pod -> API Server -> Scheduler -> start application Pod on one of worker nodes -> Kublet (actually starts Pod with a container)
        + Intelligently see all worker nodes, see available resources on each of them and decide
        + Decide on which Node new Pod should be scheduled
    3. Controller manager
        + Detect cluster state changes (e.g. crashing of Pods)
        + Controller manager -> Scheduler -> start application Pod on one of worker nodes -> Kublet (actually re-starts Pod with a container)
    4. etcd
        + Key Value Store - the cluster brain
        + Cluster changes get stored in the Key Value Store
        + Where **cluster state information** are stored, so that
            * API server knows whether cluster is healthy
            * Scheduler knows what resources are available
            * Controller manager knows whether the cluster state changes   
        => Master server can communicate with Worker Nodes

K8s cluster has many Master servers
- API server is load balanced
- etcd is distributed storage across all master servers

### Local setup
#### What is `minikube`?
**One node** K8s cluster that runs in a **Virtual Box** on computer that can use for testing K8s on local setup.

One node cluster where a Master processes and Node processes run on ONE Node/machine with docker container runtime pre-installed
- Create Virtual Box on computer
- Node runs in Virtual Box

#### What is `kubectl`?
Command line tool for K8s cluster to let user interact with cluster

To interact with cluster, need to interact with Master processes' API server using one of 3 clients
- UI
- API
- CLI -> kubectl (most powerful)

## Command
`apt install docker.io`    
`apt install net-tools`

`k3d cluster create demo -p 8081:80@loadbalancer -p 30080:30080@server:0`    
`netstat -antp | grep 30080`

`kubectl get node`

`kubectl cluster-info`

Didn't edit context of `kubectl`, interacting with the default namespace    
Didn't specify configuration option for `kubectl`, by default it will look for configuration to connect to cluster `~/.kube/config`
`cat ~/.kube/config`     
- Services certificate for accessing the cluster
- Server address
- Name, context for the username and access tocken for cluster

### Create Pod
`kubectl create -f pod.yaml`  
- `kubectl create secret docker-registry dockerhub --docker-server="https://index.docker.io/v1/" --docker-username=judytian --docker-password="" --docker-email=` 
- `kubectl get secrets`
- `kubectl delete secret dockerhub`

`kubectl describe pod mycontainer`

`kubectl get pod` 
- `kubectl get pod -o wide`

`kubectl delete pod`

### Create Service
Why we need a Service to be able to access this Pod from my browser?    
The Service create a virtual IP address
- has endpoint => endpoint is container
- IP address unique to the cluster

Service has `selector` which has to match the label we created in the Pod

`kubectl create -f service.yaml`  

`kubectl get service`

`kubectl get endpoints [service-name]`   
=> the endpoint that a Service pointing to

### If a Pod is deleted
There is no endpoints in the Service.   
If create Pod again, a new IP address will assign to Pod if you look at the endpoints of mycontainer

### Expose Service
Have to create a Service that would allow access from the underlay network because this is all still an overlay network, which means that this is at work inside the cluster.    
`ip a`    
=> Need to expose the Service to a port on IP address, so that I can access it from browser
1. Node port   
   - Open a specific port on the Node where the workload is
   - Forward traffic from that Node to the Service

   `docker ps`   
   `docker exec -it ccec82b34846 sh`   
   `iptables-save | grep 30080`
   
   `kubectl create -f nodeport.yaml`     
   `kubectl get service`    
2. Load balancer

### Create Ingress
The Ingress controller is a service that satisfies the configuration of network ingress from outside of cluster     

If this K8s cluster was running directly on the machine and not inside the Docker container, we would access their service through port 80.   
But because it's starting in a Docker container, there's another layer of redirection => use port 8181

`kubectl create -f ingress.yaml`

`kubectl get ingress`

Get Node IP address
`kubectl get node -o wide`
