# Helm Doc
[Introduction to Helm 3](https://gitlab.es.f5net.com/cloud/helm/-/blob/helm_notes/introduction-to-helm3.md)

# helm
**Kubernetes is by design complex and relies on many layers to deliver an application.** 

Consider that we build an application.  Next we deploy this application to Container.  Then for k8s we wrap this container in a pod.  As we may need more than one instance of this application we will need multiple pods and therefor we need a k8s service. If we now want to expose this service outside of K8s we need an externally available feature such as a *NodePort, LoadBalancer* or *Ingress service*.  Now if we also need to store credentials and certificates which means we now need the Secrets.  In addition custom configurations may need to be stored in ConfigMaps which are another k8s object to track.  Next we may require storage and or volumes for our data.  

All of this needs be installed in a particular order initially and any later upgrades may need to upgrade all or some of these objects.  If there is a need to rollback then the order needs to be considered again.

All of these objects, their relationships, dependencies, versions etc can be tracked by **Helm** reducing the install, updates and possible removal to single line commands. When using Helm we create a Chart that contains the kubernetes object configuration, this is passed to helm which creates a manifest file that is applied to kubetctl and deployed as a helm release to the kubernetes namespace.

`docker run —rm -it -name mytoolbox toolbox`

`kubectl get node`

`helm verison`

## Create chart
`helm create mychart`

`ls`    
`cd mychart/`    
`ls -l`

> Chart.yaml     
> chart/    
> templates/    
> values.yaml     => Default values of mychart (formated, templete file)

`helm lint .`

See actual file used    
`helm template .`

`helm install [name] .`
