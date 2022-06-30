# Resources
[Kubernetes' documentation](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)   
[YouTube video](https://www.youtube.com/watch?v=X48VuDVv0do)   
[K8s Architecture](https://zhuanlan.zhihu.com/p/382229383)

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
