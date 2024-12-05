---
layout: default
title: Container
parent: How To
nav_order: 2
last_modified_date:   2024-12-06 00:24:48
---

# Container
## How to deploy Asp.Net Core Api from VS to k3s
1. Make sure the value of **ASPNETCORE_HTTP_PORTS** in Properties/launchSettings.json is equivalent to the value of **EXPOSE** in Dockerfile
1. Build the image in **Release** configuration, whereas in **Debug** configuration it is non-workable without Visual Studio
1. Push the tag image to localhost:5000 register
1. Create a Chart with helm, using **LoadBlancer** for type, specifying ip for **ExternalIP** 
1. Install the Chart with helm
1. Check the result, that's it

### Some useful command
1. helm
    - helm create userwebapi
    - helm package userwebapi
    - helm install userwebapi ./userwebapi-1.0.0.tgz
    - helm uninstall userwebapi
    - helm list
1. kubectl
    - kubectl get svc
    - kubectl describe svc userwebapi
    - kubectl get pods
    - kubectl describe pod userwebapi-xxxx
    - kubectl logs userwebapi-xxxx