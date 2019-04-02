---
layout: post
title:  "Deployments on your Private Cloud"
date:   2019-03-31 12:00:00 -0500
categories: infrastructure kubernetes
---

## Verbose Background on How Many Physical Machines your Cloud Should Consist of and their Purpose (skip me)

I've been struggling getting my new private-cloud online and operational.  Back in the 2000s, private clouds were easy.  Install Debian on a spare machine, and install a bunch of stuff on it.  Map a bunch of NFS shares to various PCs to increase overall usable storage of the system.  

###### 1) You Need an ESXi Machine
That works fine, but what if you need to restart that debian machine because you're trying out a new kernel for one of ther services on there to use...  that means downtime for the entire cloud??  And what if the upgrade takes longer than expected?  And you wind up backing out of it, only to need to try again later, or worse, what if the upgrade hard-fails for some reason and you need a new box?  Do you even remember all the services you were exposing from your pet server?  The solution here is to use separate VMs for separate services.  Just install ESXi and create a VM for each thing manually...

But what if your ESXi goes away forever?  You have to either do backups, or have a really good memory for manually rebuilding things again in a few days (hopefully you're not super busy when you need to bring things back online, right?).  So the solution to that is to **codify your deployments** to ESXi using a little **Ansible** and either the **Terraform** plugin or **packer** (I'm using packer because I'm a nut).  

###### 2) You Need a NAS Machine with iSCSI
But what if a hard-drive goes down... you've got some serious work on your hands... unless you use a **NAS with hot-swappable drives and set things up as an array**.  Then when a hard drive goes down, go to your NAS, pull it out, and replace it with a new one, not even bothering to reboot anything!!!  

###### 3) You Need a Container Orchestration (workload Node) Machine
Then there's the issue of running out of RAM.  VMs are great... but they take time to boot up, require lot of configuration to have your apps boot up and run correctly, and each VM wastes about a gig of ram because each VM boots up it's own linux kernel.  The solution is **Kubernetes** on an Ubuntu server (or some containerization-based solution somewhere... you could even put it in a VM, though you'll get worse performance that way).  


## How to Arrange your Private Cloud's Core Services


###### 1) GitLab - Code Repo Hosting

Ok, once you've got your your ESXi server storing it's VMs on the NAS correctly (I'm skipping that step for now)... You'll need a Gitlab instance.  Gitlab is a smart thing to install in a VM because there's so much persistence involved, and there's a lot of processes running in the background too.  Only simply things should be deployed to Kubernetes really, like things that listen on port xyz, and respond to web requests, reading and writing from a simple database/ message system as needed.  

###### 2) Jenkins - Automated Process Executor/ Scheduler

Jenkins is pretty complex too, but I'm going to deploy it to Kubernetes because I think it will be easier to deal with that way in the long run.  

###### 3) Backups Process

You need to do backups of your gitlab repos.  It's just how it's gotta be.  Jenkins should be backed up too, technically.  
