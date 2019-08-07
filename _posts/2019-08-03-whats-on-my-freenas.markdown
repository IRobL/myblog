---
layout: post
title:  "What's on my FreeNAS"
date:   2019-07-15 12:00:00 -0500
categories: backups private-cloud nas
---

The two greatest banes in my technological existance thus far have been...

1. preventing the default behaviors of my OS and text editors from changing without my concent, and
2. keeping up with where my data lives in the myriad of technologies I've adopted.

While there's not a whole lot I can do about issue 1, this post is aligned at aiding in situation 2 by documenting every configuration I make to my FreeNAS server.  In a perfect world, I write a terraform provider for FreeNAS, and then run a production FreeNAS server, and also a staging FreeNAS server where I test that my terraform code really works.  Sadly, this is not that world.  I'm instead going to blog on my configuration changes to FreeNAS.


###### ZFS Pools/ Arrays

Storage -> Pools -> Add

- Create x1 three HDD and one SDD (cache) ZFS Pool where 'all your data goes'.  Note: My needs are of much less than 2TB of data.  Some will find it important to create more pools for high performance iSCSI exposure, but this is not the case for me due to frugality.


###### Datasets (folders)

Storage -> Pools -> 3dot button for your pool -> Add Dataset

I've made the following Datasets

- `archive` (NFS read, smb)
- `movs` (NFS read)
- `creative` (NFS r/w IP limit)
- `hoarding` (NFS r/w IP limit)
- `backups/repo_backups` (NFS r/w IP limit)
- `infrastructure/burp_certs` (NFS r/w IP limit)
- `infrastructure/freenas_file_system_backups` (NFS r/w IP limit)
- `infrastructure/gitlab_repos` (NFS r/w IP limit)


###### Save Config (HDD encryption codes)

System -> General -> Save Config


You put this secret file on your secret bearing USB drive and delete it from your dev laptop??  Well... assume you can leave it on your dev laptop I guess maybe...


###### Plugins

I avoid plugins because I avoid configuring software manually.  Here's the list of software I need.

- SyncThing - This is for backing up crap on your phone.  I just followed the GUI to install to http://192.168.1.96/syncthing/.
  - There are bugs with the install, and you will get file creation errors like `synchthing folder missing` and `synchthing folder marker missing`, so follow this first google hit that talks about the problem https://www.ixsystems.com/community/threads/syncthing-directory-access-denied.49697/
    - So there's a really defensive guy on that forum apparently, sorry about that, it's kind of an odd forum.  The actual solution I used was to...
      - `iocage exec syncthing` to get in the jail
      - `mkdir /sync_folders && chmod 0777 sync_folders && chown -r syncthing /sync_folders` to create a place where syncthing can write to
      - Then in the SyncThing Web GUI, goto `Actions` -> `settings` -> `Default Folder Path` and make it be `/sync_folders`


###### Jails

- DLNA Server - https://www.ixsystems.com/community/threads/how-to-install-minidlna-into-iocage-freenas-11-2.68978/


