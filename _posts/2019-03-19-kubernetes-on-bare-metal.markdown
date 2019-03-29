---
layout: post
title:  "Kubernetes on Bare Metal"
date:   2019-03-19 12:00:00 -0500
categories: infrastructure kubernetes
---

Main Ref: https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

This blog post comes after sinking many hours of time into setting up an openshift cluster.  I think the problem there is that RedHat is a very for-profit and corporate employee type of software company and therefore there isn't very much emphasis on making Openshift an easy to configure product.  I really wanted to learn more about Openshift, but since it requires 'vendor support' to actually start using, I decided to just use Kubernetes which is really the software that powers Openshift in the first place and probably a more appropriate starting point for learning about scalable containerized deployments.  

At the end of this tutorial, you'll be able to deploy apps to your new Microk8s server.  


###### Overview
- Install Ubuntu on a bare metal PC, choosing to use the [microk8s](https://github.com/ubuntu/microk8s) Snap during the Ubuntu installation process
- Turn on various "required" Microk8s plugins
- Setup some bash aliases on that Ubuntu machine so local development can happen
- You can restart kubernetes with `sudo snap disable microk8s; sudo snap enable microk8s` (give it time to boot back up)


## Install Ubuntu (and include the microk8s snap)

Get a basic [Ubuntu Server ISO](https://www.ubuntu.com/download/server) image from their website, using version 18.04.2 LTS.  It's smart to use their LTS release since it's more stable and such.

When you're installing, you'll see a section about adding `snaps`.  Snaps are a new Ubuntu packaging technique that allows package developers to bundle dependencies into their snap package instead of relying on the external packages which historically hasn't always been reliable.  In the case of microk8s, docker is bundled into it, and this docker engine is segregated from the system's docker (if one is installed).  To access the bundled docker, you run `microk8s.docker`.  

During the Ubuntu installation process, choose to enable the `microk8s` snap by moving the cursor over to it and using the space bar.  Snaps can also be installed post-installation, simply run the below command:

```
snap install microk8s --classic
```

Snaps don't put their configurations in `/etc`, instead you'll find your microk8s configuration files around this path `/snap/microk8s/current/known_token.csv`


## Enable Extra Functionality for microk8s

Ref:  https://github.com/ubuntu/microk8s

```
sudo snap alias microk8s.kubectl kubectl # make an alias

microk8s.status
sudo microk8s.enable dns dashboard registry ingress

# Show the dashboard in `cluter-info` requests
kubectl --namespace=kube-system label services/kubernetes-dashboard kubernetes.io/cluster-service=true

kubectl cluster-info
sudo iptables -P FORWARD ACCEPT

# Check the containers are running correctly because you're the curious type
microk8s.docker ps  ## Apparently, this isn't working now...
```


## Validate your dashboard install

```
microk8s.kubectl get nodes
microk8s.kubectl get pods
microk8s.kubectl get all
microk8s.kubectl cluster-info
```

Navigate to the `kubernetes-dashboard` url that should have been printed in that `kubectl cluster-info` command.  You won't be able to login yet though, that's next.  


## Connect remotely from development laptop

ref:  https://www.admintome.com/blog/connecting-to-your-kubernetes-cluster-remotely/

When you installed microk8s, it created an important configuration file at `/snap/microk8s/current/configs/kubelet.config`.  That file, believe it or not, is how you log into kubernetes.  To enable your development laptop to login, simply copy this file to your dev machines `~/.kube/config` file and you're good to go!  Kubernete's support for 'users' is terribly limited right now, so simply enjoy admin and be glad you don't work for an enterprise company that wants you to email support teams to create users for you and email you config files back ;)

```

scp torpedo:/snap/microk8s/current/configs/kubelet.config ~/.kube/config

# Optionally point this at some OTHER config file in ~/.kube,
# the default is config...
export KUBECONFIG_SAVED=config

# alternatively, you can point to a config file with this switch...
kubectl get pods --kubeconfig=config   
```


<!--
## Login from a Remove Machine (not currently possible!)

You'll probably want to access your k8s cluster remotely.  Logging into the cluster is [weird](https://blog.christianposta.com/kubernetes/logging-into-a-kubernetes-cluster-with-kubectl/).  In a perfect world you could run the below commands to login, but this won't work for microk8s until you create a custom user....

```
# TODO:  Create a custom user, enable https, etc., etc.!
# This section is currently broken

# Create a login profile...    # Actually... Setup a user
kubectl config set-credentials kubeuser/torpedo --username=kubeuser --password=kubepassword

# Add a cluster to your profile...  # Actually... setup a cluster
kubectl config set-cluster torpedo --insecure-skip-tls-verify=true --server=http://192.168.1.130

# Set up a profile login in our user to our cluster context
kubectl config set-context default/torpedo/kubeuser --user=kubeuser/torpedo --namespace=default --cluster=torpedo

# Switch to our profile
kubectl config use-context default/torpedo/kubeuser
```
-->


## Validate that your internal Docker registry works

Ref:  https://github.com/ubuntu/microk8s/blob/master/docs/registry.md

```
host_name=192.168.1.130
host_name=127.0.0.1   # Using localhost can be messed up if ipv6 is setup in /etc/hosts...

docker pull gcr.io/google_containers/echoserver:1.4
docker tag gcr.io/google_containers/echoserver:1.4 ${host_name}:32000/echoserver
docker push ${host_name}:32000/echoserver
```

###### Pro Tip:  
You can access the insecure registry remotely by running this from your development laptop you'll be connecting from...

```
MICROK8S_SERVER=torpedo
ssh -L 32000:localhost:32000 $MICROK8S_SERVER  "tail -f /dev/null"
```

Subsequent connections to `localhost:32000` will be routed to the microk8s servers `localhost:32000`!


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

Deployments are a really fun concept in kubernetes.  Basically, to create an app running, all you need is to create a deployment, and several kubernetes primitives will be created to support your app running.  Those primitives are:

- Replicasets: These shepherd-like objects look over your pods and increase or decrease the number that are running to match the number of 'backup/ load balancing' pods you desire.  
- Pods:  These are the dog-like objects that execute the linux application in a container.  Advanced people sometimes stick two containers in a single pod so they can speak directly to eachother over localhost, but let's not get into the weeds here ;)
- Services:  These objects are the ears of your containers.  They allow your dogs to hear the commands of their loving shepherd and perform the work requested of them.  

Go ahead, inspect all these lovely objects:

```
kubectl get deployments
kubectl get replicasets
kubectl get pods
kubectl get services # HA, we didn't actually make these!  We'll do that later
```

Ok, have you had your fill of deploying an nginx server?  Delete it by it's name:

```
kubectl delete deployments/nginx-deployment
```



## Expose a Container to the host's network via ingress controllers

- First make sure ingress is enabled with `microk8s.enable ingress`.  
- Create a simple web server deployment `kubectl run echoserver --image=gcr.io/google_containers/echoserver:1.4 --port=8080`
- Create a service resource for our new deployment  `kubectl expose deployment echoserver --type=NodePort`
- Now write an ingress rule to send requests that hit the cluster for the host named `hello.com` and :

```
microk8s.kubectl apply -f - <<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubecon
  annotations:
    nginx.ingress.kubernetes.io/nickname: "Wooohoooooo we're here!!!"
spec:
  rules:
  - host: hello.com
    http:
      paths:
      - path: '/sub-page'
        backend:
          serviceName: echoserver
          servicePort: 8080
EOF
```

Here's the final validation test, you can run this from outside the cluster and get access to the container!  

```
curl hello.com/echo
```


misc.  

http://dag.wiee.rs/howto/ssh-http-tunneling/


<!--
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
-->
