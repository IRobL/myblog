---
layout: post
title:  "how i setup Proxmox PVE-8"
date:   2023-09-28 12:00:00 -0500
categories: private-cloud
---

## I Finally Migrated from ESXi

So, when I first added support for ESXi, it was back in 2012 or so.  At the time I was deeply saddened that there wasn't anything in the FOSS community around hypervisor technology.  When I first began working with ESXi, I assumed I was in for tons of vendor lock in and manual configuration steps, and to my surprise, with a little elbow grease, this wasn't the case.  I was happy with ESXi, and even found that the free version of ESXi let me completely saturate my motherboard with RAM.

Fast forward to 2023, something went wrong with my ESXi install.  Perhaps the USB flash drive I used for installation failed.  I didn't have a backup.  When I went to upgrade ESXi, I discovered version 8 of ESXi was no longer free and available to me.  Further, the distribution of ESXi 7 was completely bungled.  I had the option to either re-construct the ~2015 installation media for ESXi 6, or just install Proxmox.  I wound up doing the latter.  Here's how!


## 1.  Download the iso, and us `dd` to burn it to my usb drive.

Here's the command I used on my mac:

```
sudo dd bs=10M if=proxmox-ve_8.0-2.iso of=/dev/disk2  status=progress
```

It just worked because they didn't bungle the distribution and bundled syslinux correctly in their ISO!


## 2.   Plug in the install USB stick and install

I just followed the GUI to install, updating the hostnames, but otherwise defaulting things.  I had to change what port the network cable was plugged into to match up with the default configurations, but this was the only minor issue I had before getting the Web GUI of my fresh Proxmox server operational.

The storage layout was however quite tricky!  In ESXi, the hypervisor OS was designed from the ground up to be a hypervisor, and so it's boot partition is only read from once during boot, and thereafter runs completely from RAM.  That's a really cool feature, but Proxmox is just a bunch of open source tools installed on top of Debian linux, so it doesn't have this cool lives-in-ram feature.  

Also, I'm pretty sure Proxmox is assumed to be installed on top of some kind of ZFS partition.  If you do anything else, weird, sluggish things will happen!  

I figure I'll get the most out of Proxmox if I use a 4-disk layout.  The boot partition "device" will be the ONLY virtual device I use, and will be a ZFS Raid10 setup.  This gives me plenty of performance by striping two disks, on top of redundancy of also mirroring them.  

Supposedly, there's a bug where Proxmox writes something like 30gb per day to it's boot partition.  It's kinda crazy, but they can't figure out what's doing it or stop it.  That means that even if you're not using proxmox very much, it's still going to be chewing through your drives, so you'll be money ahead if you just set things up to be failure tolerant and hopefully when your first drive fails, you notice in time and can swap in a replacement before it's too late.  



## 3. Setup ssh authentication

So, obviously setup your `~/.ssh/config` file so you can easily ssh into this beast.  

## 4. Create API ticket

Go into the UI, and create a service account token.  In the Proxmox UI, see...

```
Datacenter -> permissions -> API Tokens -> Add (button)
```

## 5. Tie in your SSL Certs

Here's a snippet from a script I run when I need to update the https certs.  

```
  cat_of_fullchain=$(cat data/letsencrypt/live/proxmox.example.com/fullchain.pem)
  cat_of_privkey=$(cat data/letsencrypt/live/proxmox.example.com/privkey.pem)

  curl -v -k -X POST ${TF_VAR_proxmox_url}/nodes/proxmox/certificates/custom \
    -H "Authorization: PVEAPIToken=${TF_VAR_proxmox_user}=${TF_VAR_proxmox_token}" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    --data-urlencode "key=${cat_of_privkey}" \
    --data-urlencode "restart=1" \
    --data-urlencode "force=1" \
    --data-urlencode "certificates=${cat_of_fullchain}"
```

Basically I use certbot to create all my certs, and the above snippet finds those certs (.pem files) and pushes them into Proxmox over their http API.  
