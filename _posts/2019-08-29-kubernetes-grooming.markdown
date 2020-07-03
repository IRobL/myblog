---
layout: post
title:  "Kubernetes - Grooming a Fresh Install"
date:   2019-03-19 12:00:00 -0500
categories: infrastructure kubernetes
---

I'd like this blog post to be about how to configure your k8s cluster with a user.


#### How to produce a token that can be used via the --token parameter in kubectl

```
kubectl get secrets/default-token-cfzhc -n kube-system -o=jsonpath={.data.token} | base64 --decode
```


#### Create a Namespace for a User




#### Create the Service Account

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

#### Give the service account the admin-user role

Is this needed?
```
???
```

#### Extract the login token and use it to login to the dashboard

Please note: the dashboard is basically read-only because with Kubernetes, it's better to manage the system using files that are stored in a Git repo.  

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
