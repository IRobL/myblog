---
layout: post
title:  "The Kubernetes Ecosystem"
date:   2019-03-19 12:00:00 -0500
categories: infrastructure kubernetes
---

Here's a high level list of links in a logical order that you need to learn about Kubernetes in order to be effective with it.  Kubernetes is broken up into components.  When you 'install' kubernetes, you only have some pretty basic functionality out-of-the-box and need extra features like `ingress` and `dns` to actually get things going.

- [Helm](https://helm.sh/docs/developing_charts/): A way of allowing kubernetes manifest files to take in parameters almost identical to how Openshift's parameter's concept works.

- The [Internal Docker Registry](https://github.com/ubuntu/microk8s/blob/master/docs/registry.md):  You push your docker images here and Kubernetes pulls from this place.

- The [Load Balancer](https://kubernetes.io/docs/concepts/services-networking/#loadbalancer) service type:  This service type allows you to expose a service through a dedicated loadbalancer.  Usually this is something cloud providers offer.

- https://www.consul.io/docs/platform/k8s/service-sync.html

- [Metal LB](https://metallb.universe.tf/):  This 'plugin' allows you to provide the Load Balancer type service when using a bare-metal k8s cluster.

- [Ingress Resources](https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress): The resource responsible for exposing HTTP and HTTPS requests using an [ingress controller]](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html).

- [Service_Resources](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies): The resource resposible for exposing Layer 4 (TCP/UDP) access to containers within a cluster.


- [Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM): Micro-service visibility (and security) and adds Manifest kinds that enable you to control how traffic is routed to services, and touches on authentication too!


- []()


#
