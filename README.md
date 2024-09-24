# HelmCharts

On the github Repo., under Settings -> Pages, Select
- Build and deployment -> Deploy from a branch
- branch  -> choose main -> /root

This will allow it to be run as a server 

## Create custom helm charts
```shell
helm create myFirstChart
helm package myFirstChart
helm repo index .
```

## Update

### Test updates
Upgrade test on local
```shell
helm upgrade --install todo-service-app myrepo/todo-app -f values.yaml 
```

### After Testing
**After helm file changes**
```shell
helm package todo-app
helm repo index . 

# After Git push
helm repo update 
```
**Upgrade pod**
```shell
helm upgrade --install todo-service-app myrepo/todo-app
```
