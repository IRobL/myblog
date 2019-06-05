---
layout: post
title:  "Working with Helm and K8s"
date:   2019-05-01 12:00:00 -0500
categories: infrastructure kubernetes
---

One of the great things that Openshift came with that K8s didn't have out of the box was the ability to pass variables into your Kubernetes manifest files.  Don't worry, you can get the same kind of thing in Kubernetes by using a product called Helm, though it brings with it a little extra overhead (and I'm actually starting to appreciate it's overhead as well).  

With Helm, you can define your Kubernetes deployments as Helm Charts.  You'll get to pass variables into your K8s yamls and you'll also (read need) the `helm` CLI tool to deploy them to your cluster.  For example, if you had a local helm chart defined and wanted to deploy it, you would run the below command.

```
helm install --name my-chart-stage .
```

You could then list of all your deployments:

```
helm list
```

After making changes to your chart you would want to roll out your latest changes with:

```
helm upgrade my-chart-stage .
```

Finally, delete your deployment with:

```
helm delete my-chart-stage
```

Pretty cool, huh?

How helm Templates work:  https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md


###### Download a suitable Binary

I used the links on this page to get v2.10.0 https://github.com/helm/helm/releases/tag/v2.10.0


###### Use helm to install tiller (the server part of helm) into your k8s cluster

helm init --tiller-image=k8s-images:tiller-v2.10.0
