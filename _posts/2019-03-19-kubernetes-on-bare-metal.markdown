---
layout: post
title:  "Kubernetes on Bare Metal"
date:   2019-03-19 12:00:00 -0500
categories: infrastructure kubernetes
---

Main Ref: https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

So this blog post comes after sinking many hours of time into setting up an openshift cluster.  I think the problem there is that RedHat is a very for-profit and corporate employee type of software and therefore their isn't very much emphasis on making Openshift an easy to configure thing.  I really wanted to learn more about Openshift, but since it requires 'vendor support' to actual start using, I decided to just user Kubernetes which is really the software that powers Openshift in the first place.  

###### Overview
- Install Ubuntu on a bare metal PC, choosing to use the [microk8s](https://github.com/ubuntu/microk8s) Snap during the Ubuntu installation process
- Setup some bash aliases on that Ubuntu machine so local development can happen
-


## Install ubuntu (and include the microk8s snap)

Get a basic [Ubuntu Server ISO](https://www.ubuntu.com/download/server) image from their website, using version 18.04.2 LTS.  It's smart to use their LTS release since it's more stable and such.

When you're installing, you'll see a section about adding `snaps`.  Snaps are this Ubuntu concept that is like a package manager, but I guess better... more modern that aptitude and such.  Choose to enable the `microk8s` snap by moving the cursor over to it and using the space bar.

Snaps can also be installed post-installation, simply run `snap install microk8s --classic`.

Snaps don't put there configurations in `/etc`, instead you'll find your microk8s configuration files around this file (your secret token) `/snap/microk8s/current/known_token.csv`



## Setup the system for local testing

```
sudo snap alias microk8s.kubectl kubectl
```


## Enable Extra Functionality for microk8s

Ref:  https://github.com/ubuntu/microk8s

```
microk8s.status
sudo microk8s.enable dns dashboard

# Show the dashboard in `cluter-info` requests
kubectl --namespace=kube-system label services/kubernetes-dashboard kubernetes.io/cluster-service=true

kubectl cluster-info
sudo iptables -P FORWARD ACCEPT


# Check the containers are running correctly
microk8s.docker ps
```


## Validate your dashboard install

```
microk8s.kubectl get nodes
microk8s.kubectl get nodes
microk8s.kubectl cluster-info
```

Navigate to the `kubernetes-dashboard` url that should have been printed in that `kubectl cluster-info` command.  You won't be able to login yet though, that's next.  



## Validate you can deploy simple containers

Ref: https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment

Applying the below deployment configuration will spin up 2 containers serving basic html content via nginx.  
```
microk8s.kubectl apply -f - <<EOF
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
EOF
```




## What is a Kubeconfig?

Ref: [kubernetes.io Kubeconfigs](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
Ref: http://docs.shippable.com/deploy/tutorial/create-kubeconfig-for-self-hosted-kubernetes-cluster/

By default, `kubectl` looks in `~/.kube/config` to find your Kubeconfig file.  This is how you organize access to a specific kuberenetes cluster.  


## Configure K8s Dashboard

Ref: https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
Better ref: http://docs.shippable.com/deploy/tutorial/create-kubeconfig-for-self-hosted-kubernetes-cluster/
SO Ref: https://stackoverflow.com/questions/42170380/how-to-add-users-to-kubernetes-kubectl

###### Create the service account

```
microk8s.kubectl create -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: svcs-acct-dply #any name you'd like
EOF
```

Validate you can get the secret name:

```
kubectl describe serviceAccounts svcs-acct-dply
```

From that output, figure out the name of the mountable secret and print it:

```
kubectl describe secrets svcs-acct-dply-token-h6pdj
```

###### Give the service account the admin-user role

Is this needed?
```
???
```

###### Extract the login token and use it to login to the dashboard

Please note: the dashboard is basically read-only because with Kubernetes, it's better to manage the system using files that are stored in a Git repo.  

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```



## How to enable external access to your pods







misc.  

http://dag.wiee.rs/howto/ssh-http-tunneling/
