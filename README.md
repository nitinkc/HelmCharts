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

