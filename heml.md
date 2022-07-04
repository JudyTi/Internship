# helm
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
