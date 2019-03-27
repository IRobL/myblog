---
layout: post
title:  "The Kubernetes Ecosystem"
date:   2019-03-19 12:00:00 -0500
categories: infrastructure kubernetes
---

Here's a high level list of links in a logical order that you need to learn about Kubernetes in order to be effective with it.  Kubernetes is broken up into components.  When you 'install' kubernetes, you only have some pretty basic functionality out-of-the-box and need extra features like `ingress` and `dns` to actually get things going.  


- The [Internal Docker Registry](https://github.com/ubuntu/microk8s/blob/master/docs/registry.md):  You push your docker images here and Kubernetes pulls from this place.

- [Ingress Resources](https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress): The resource responsible for exposing HTTP and HTTPS requests using an [ingress controller]](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html).

- [Service_Resources](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies): The resource resposible for exposing Layer 4 (TCP/UDP) access to containers within a cluster.


- [Helm](https://helm.sh/docs/): A way of packaging complex, multi-container applications using yaml files similar to how dockercompose works.

- [Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM): Micro-service visibility (and security) and adds Manifest kinds that enable you to control how traffic is routed to services, and touches on authentication too!


- []()


#
