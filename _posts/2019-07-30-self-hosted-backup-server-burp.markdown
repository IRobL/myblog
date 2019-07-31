---
layout: post
title:  "Tuning BURP - A Mature Self-Hosted Backup Solution"
date:   2019-07-15 12:00:00 -0500
categories: backups private-cloud
---

Backups are important.  Your best stuff should already be shared on GitHub, but some stuff is nice and isn't already setup in it's own git repository.  Plus, knowing that your stuff is safely on your own NAS can be more conmforting than having it on someone else's computers.

###### I backup these kinds of things
- My non-cloud based password manager
- My SSH Keys???
- My email client configurations??
- My web browser configurations??
- My WIP Writing

###### I store centrally, these kinds of things
- My music
- My videos
- My pictures (well.. not really...)
- My source code (GH + GitLab)
- My Text editor configurations
- My linux dotfiles
- My ToDo list (gist)
- My sensative dotfiles and SSH configs

Ok, so clearly, I don't need to backup EVERYTHING, but generally, I backup my home folder in linux.  Because that's 35GB we're talking about, I exclude stuff from that folder that I know takes up a lot of space, and isn't actually all that useful to backup.


## Getting started tuning BURP

Burp's clients are configured on the server in `/etc/burp/clientconfdir/`.  Every client has it's own unique configuration file, for example:

(`/etc/burp/clientconfdir/my-laptop`)

```
password = SOMESHAREDSECRET

. incexc/linux_workstation
```

For your password, it must match whatever the server's `burp-server.conf` file sets password to.  Then see the `. incexec/linux_workstation` line?  That's where we are inheriting our backup configurations from a common configuration file.  So let's look at that `linux_workstation` file which I use to backup both my laptops, and my development desktop.

(`/etc/burp/clientconfdir/incexc/linux_workstation`)

```
include_glob=/home/*
exclude=/home/.ecryptfs
include=/etc/fstab

########################
# Home Folder Excludes #
########################

# Common
exclude_regex=/home/.+/Downloads
exclude_regex=/home/.+/bin
exclude_regex=/home/.+/dev
exclude_regex=/home/.+/tmp
exclude_regex=/home/.+/packer_cache
exclude_regex=/home/.+/VirtualBox\sVMs

# System Files
exclude_regex=/home/.+/\.Private
exclude_regex=/home/.+/\.ecryptfs

# General Applications
exclude_regex=/home/.+/\.cache
exclude_regex=/home/.+/\.config
exclude_regex=/home/.+/\.local
exclude_regex=/home/.+/\.thunderbird
exclude_regex=/home/.+/\.openshot
exclude_regex=/home/.+/\.minecraft
exclude_regex=/home/.+/\.mozilla

# Developer
exclude_regex=/home/.+/\.atom
exclude_regex=/home/.+/\.vagrant.d
exclude_regex=/home/.+/\.m2
exclude_regex=/home/.+/\.rvm
exclude_regex=/home/.+/\.gem
exclude_regex=/home/.+/\.node
exclude_regex=/home/.+/\.npm
exclude_regex=/home/.+/\.nvm
exclude_regex=/home/.+/\.platformio
exclude_regex=/home/.+/\.basher
exclude_regex=/home/.+/\.bundle
exclude_regex=/home/.+/\.kube
exclude_regex=.*node_modules/.*

max_file_size=400MB
```

Ok, those configurations will get you close, but you'll probably need to test them by running your backup and then listing all the files that were backed up and adding various additional exclude patterns until you have a manageable backup that is still complete enough to be useful.


###### Run your backups

```
$ burp -a b
```

###### Save a list of files backed up for review

```
$ burp -a l > /tmp/backup_list
```

From that list, you'll just be able to scroll down it and see folders that contain massive amouns of files.  You'll want to ignore such folders unless you deam them to be of value.

