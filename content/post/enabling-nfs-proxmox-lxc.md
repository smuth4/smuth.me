+++
tags = ["lxc", "proxmox", "nfs"]
date = "2018-10-11"
title = "Enabling NFS mounts in Proxmox 5.2 LXC containers"
+++

Normally, Proxmox doesn't allow mounting NFS mounts directly in containers due to security concerns. In the past it was possible to modify apparmor profiles directly in order to allow it, as seen [here](https://forum.proxmox.com/threads/lxc-nfs.23763/) and [here](https://www.svennd.be/mount-nfs-lxc-proxmox/). However, as of [this commit](https://git.proxmox.com/?p=pve-container.git;a=commit;h=5a63f1c5d3b995dd682a70e7fbd1364240e09278), that method is no longer an option, due to the apparmor profiles now being generated dynamically when the containers are started (you can view these profiles at `/var/lib/lxc/${CID}/apparmor/lxc-${CID}_<-var-lib-lxc>`). Instead, modifying apparmor directly is no longer necessary, as you can now add the undocumented `feature` setting to the [pct.conf](https://pve.proxmox.com/wiki/Manual:_pct.conf) files for individual containers. For instance:

```
features: mount=nfs
```

allows NFS mounting for the specified container, although a restart will be necessary for the setting to take effect. You can verify this by running `grep nfs /var/lib/lxc/${CID}/apparmor/lxc-${CID}_\<-var-lib-lxc\>`, which return a line like `mount fstype=nfs,`. Other filesystems can also be allowed by separating the different types with semicolons.

This change was released in version 2.0-28 of the pve-container package, so it's easy tell if you are affected:

```
$ dpkg -s pve-container | grep '^Version:'
Version: 2.0-28
```

Since this change only applies on container start-up, it's possible to upgrade the package first without impacting any running containers.
