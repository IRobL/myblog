---
layout: post
title:  "how i setup Proxmox PVE-8"
date:   2023-09-28 12:00:00 -0500
categories: private-cloud
---

## I Finally Migrated from ESXi

So, when I first added support for ESXi, it was back in 2012 or so.  At the time I was deeply saddened that there wasn't anything in the FOSS community around hypervisor technology.  When I first began working with ESXi, I assumed I was in for tons of vendor lock in and manual configuration steps, and to my surprise, with a little elbow grease, this wasn't the case.  I was happy with ESXi, and even found that the free version of ESXi let me completely saturate my motherboard with RAM.

Fast forward to 2023, something went wrong with my ESXi install.  Perhaps the USB flash drive I used for installation failed.  I didn't have a backup.  When I went to upgrade ESXi, I discovered version 8 of ESXi was no longer free and available to me.  Further, the distribution of ESXi 7 was completely bungled.  I had the option to either re-construct the ~2015 installation media for ESXi 6, or just install Proxmox.  I wound up doing the latter.  Here's how!


## 1.  Download the iso, and use `dd` to burn it to my usb drive.

Here's the command I used on my mac:

```
# Troublshoot to find the right device name and unmount any partitions that have mounted
sudo diskutil umount /dev/disk2s3
cd ~/Downloads
sudo dd bs=10M if=proxmox-ve_8.1-2.iso of=/dev/disk2  status=progress
sync
sudo diskutil eject /dev/disk2
```

It just worked because they didn't screw up the distribution like VMWare did and bundled syslinux correctly in their ISO!


## 2.   Plug in the install USB stick and install

###### TL;DR
- Just follow along the GUI to install as default for most things
- Choose XFS or ZFS as your partition type
- 4 NVMe disks over ZFS-1 is ideal

I just followed the GUI to install, updating the hostnames, but otherwise defaulting things.  I had to change what port the network cable was plugged into to match up with the default configurations, but this was the only minor issue I had before getting the Web GUI of my fresh Proxmox server operational.

The storage layout was however quite tricky!  In ESXi, the hypervisor OS was designed from the ground up to be a hypervisor, and so it's boot partition is only read from once during boot, and thereafter runs completely from RAM.  That's a really cool feature, but Proxmox is just a bunch of open source tools installed on top of Debian linux, so it doesn't have this cool lives-in-ram feature.

Also, Proxmox is designed assuming it will be installed on top of some kind of ZFS partition.  If you do anything else, then weird and sluggish things will happen!  You need at least XFS for copy-on-write functionality.

I figure I'll get the most out of Proxmox if I use a 4-disk layout.  The boot partition "device" will be the ONLY virtual device I use, and will be a ZFS Raid10 setup.  This gives me plenty of performance by striping two disks, on top of redundancy of also mirroring them.

Supposedly, there's a bug where Proxmox writes something like 30gb per day to it's boot partition.  It's kinda crazy, but they can't figure out what's doing it or stop it.  That means that even if you're not using proxmox very much, it's still going to be chewing through your drives, so you'll be money ahead if you just set things up to be failure tolerant and hopefully when your first drive fails, you notice in time and can swap in a replacement before it's too late.


## 3. Setup ssh authentication

So, obviously setup your `~/.ssh/config` file and `/root/.ssh/authorized_keys` files you can easily ssh into this beast.


## 4. Install System Monitoring Tools

You'll need some tools in order to perform some basic system analysis and optimization.

```
apt-get install iotop sysstat nload lm-sensors -y
```

Usage is pretty simple:

```
iotop

iostat -dx 5

nload

sensors |grep Package
```

## 5. Create API ticket

Go into the UI, and create a service account token.  In the Proxmox UI, see...

```
Datacenter -> permissions -> API Tokens -> Add (button)
```

## 6. Tie in your SSL Certs

Here's a snippet from a script I run when I need to update the https certs.

```
  my_domain="proxmox.example.com"

  cat_of_fullchain=$(cat data/letsencrypt/live/${my_domain}/fullchain.pem)
  cat_of_privkey=$(cat data/letsencrypt/live/${my_domain}/privkey.pem)

  curl -v -k -X POST ${TF_VAR_proxmox_url}/api2/json/nodes/proxmox/certificates/custom \
    -H "Authorization: PVEAPIToken=${TF_VAR_proxmox_user}=${TF_VAR_proxmox_token}" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    --data-urlencode "key=${cat_of_privkey}" \
    --data-urlencode "restart=1" \
    --data-urlencode "force=1" \
    --data-urlencode "certificates=${cat_of_fullchain}"
```

Basically I use certbot to create all my certs, and the above snippet finds those certs (.pem files) and pushes them into Proxmox over their http API.  I can't believe I declined pusuing a job with them because I worried it drew me away from a former workplace that was tangentially related to a passion I held at the time =/


## 7. Enable PCI Passthrough

Official docs: https://pve.proxmox.com/wiki/PCI(e)_Passthrough


## 8. MitM proxy

MitM Proxy is essential if you're developing or working with terraform plugins. 

Ref: https://askubuntu.com/questions/1176109/how-do-i-install-mitmproxy-on-ubuntu-18-0-4

```
apt install mitmproxy
```

Server Usage:

```
mitmweb --listen-host 0.0.0.0 --web-host 0.0.0.0  --mode reverse:https://127.0.0.1:8006 --ssl-insecure
```

When you want to run through the mitm process, just run the mitm server, and specify :8080 as your proxmox url's port.  It's very slow...


## 9. Update storage configs

`/etc/pve/storage.cfg`

```
dir: local
	path /var/lib/vz
	content iso,vztmpl,backup,snippets

zfspool: local-zfs
	pool rpool/data
	sparse
	content images,rootdir
```


## (XFS) On Passing Through an Intel igpu

ref: https://3os.org/infrastructure/proxmox/gpu-passthrough/igpu-passthrough-to-vm/#proxmox-configuration-for-igpu-full-passthrough

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction initcall_blacklist=sysfb_init video=simplefb:off video=vesafb:off video=efifb:off video=vesa:off disable_vga=1 vfio_iommu_type1.allow_unsafe_interrupts=1 kvm.ignore_msrs=1 modprobe.blacklist=radeon,nouveau,nvidia,nvidiafb,nvidia-gpu,snd_hda_intel,snd_hda_codec_hdmi,i915"
```


## (ZFS) On Passing Through an Intel igpu

ZFS systems don't use grub for setting the kernel command.  Instead you'll need to set things up this way:

(/etc/kernel/cmdline)
```
root=ZFS=rpool/ROOT/pve-1 boot=zfs intel_iommu=on iommu=pt
```


(/etc/modules)
```
.
.
.
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

Then run these (for the respective changes you made):

```
/etc/kernel/postinst.d/zz-proxmox-boot
update-initramfs -u -k all
```

