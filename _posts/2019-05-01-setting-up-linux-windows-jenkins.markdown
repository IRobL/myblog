---
layout: post
title:  "Setting up Jenkins on Linux with a Windows Build Executor"
date:   2019-05-01 12:00:00 -0500
categories: infrastructure jenkins
---

###### Basics/ Getting the master online

So setup jenkins... It's really not that hard, apt-get does the trick for the most part on linux, then if you wanna get fancy, install nginx, and have it forward to jenkins which by default is listening on port 8080.  If you don't know how to make it fancy, don't bother, it's not worth the wasted time!

I usually like to make all of my infrastructure codified to be configured how it is from scratch.  Jenkins is entirely configured via `.xml` files on the build master --there are no databases!  The plugins are just downloaded jar.  All configuration is in `/var/lib/jenkins`.  All build configurations and logs are in there too!  Weird, so how do you do backups and restores?  You can just use git if you want!


###### Configuration as Code Though?
Down the road, I intend to have all the jobs mounted as an NFS share (with ZFS doing snapshots on it in the background).  After that I'll lean heavily on the `/var/lib/jenkins/init.groovy` and `/var/lib/jenkins/init.groovy.d/` scripts to have jenkins configuration as code which should work pretty well.

Refs:
  - https://wiki.jenkins.io/display/JENKINS/Post-initialization+script
  - https://gist.github.com/wiz4host/17ab33e96f53d8e30389827fbf79852e
  - Install Plugins:  https://github.com/coreos/jenkins-os/blob/master/init.groovy
  - https://coderwall.com/p/2t242g/jenkins-plugin-management-in-groovy
  - Wait for plugin install?  https://gist.github.com/TheNotary/025d30389444aed926ef58e996d34583
  - A guy's backup script: https://gist.github.com/cenkalti/5089392


###### Create a Build Agent (on windows)

Out of the box, build jobs can run directly on the master node.  But we want to run windows jobs for things like Unity and Unreal4Engine.  So we just need to...

  - Go to Manage jenkins -> Configure Global Security -> Agents: Random (radio button)
  - Follow this tut to add a build agent: https://theithollow.com/2016/04/11/add-a-jenkins-node-for-windows-powershell/
  - Install Java on our Windows-based gaming rig
  - Download the agent.jar onto the windows machine via http://yourserver/jnlpJars/agent.jar
  - Navigate to https://yourserver/computer/windows_7_agent/ assuming you named your machine `windows_7_agent` to see the `Run from agent command line:` command

Once that's done with, you should be able to execute your build jobs from jenkins on your gaming machine!

ref: https://theithollow.com/2016/04/11/add-a-jenkins-node-for-windows-powershell/
