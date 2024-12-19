---
layout: default
title: Ingress
parent: How To
nav_order: 2
last_modified_date:   2024-12-06 00:24:48
---
# Ingress
The Ingress concept lets your map traffic to different backends based on rules you define via K8s Api.

# Ingress Nginx Controller
ingress-nginx is an Ingress Controller for k8s using NGINX as a reverse proxy and loadbalance.

# Install
1. download [deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/cloud/deploy.yaml)
2. configure yaml: find Kind: Service which contains type: LoadBalance, and change the port in case conflict with other service as if collision happens the External Ip will be pending all the time.
3. if the External Ip is pending, may be need to specify the externalIPs right after the type.
4. after installing, the status of ingress-nginx-admission-* related pods will be completed and with 0/1 ready, that's normal if the pod of ingress-nginx-controller-* is with 1/1 ready.
5. then follow the [local testing](https://kubernetes.github.io/ingress-nginx/deploy/) 

# Description
1. ingress-nginx-controller is nginx service for handling traffic from outside of the cluster.
2. service of ingress with nginx class is used to defined a rule for how to forward the url