---
title: "GNOME Boxes"
date: 2024-02-04T13:15:49+01:00
slug: 2024-02-04-gnome_boxes
type: posts
draft: false
categories:
  - linux
tags:
  - virtualisation
  - fedora
  - boxes
---
Boxes is the built-in virtual machine manager included with the GNOME desktop environment. As it's there, let's give it a try !

## File transfer
Drag and drop works, but you may want to mount some folders. The SPICE protocol does the trick:
* Install **spice-webdavd** and **spice-client-gtk** in the guest VM and restart it
* Share the folders through the Boxes settings
* Go in **+ Other Locations** in the guest VM Nautilus file explorer
* An entry **Spice client folder** containing the shared folders should appear


## Advanced VM settings
It's simplicity comes at the expense of configuration options. Fortunately it runs QEMU/KVM under the hood thus advanced settings can be managed with **virt-manager**. Once installed, simply connected to **QEMU/KVM user session** and all the Boxes VMs will show up.

## Storage location
By default VMs are stored in `~/.local/share/gnome-boxes` which I don't really like. There is no easy option to change that but I'm still going to move it to a prefered place.

Mount the new volume as root.
```
mkdir /lab/
chown :wheel /lab/ # Assuming the user is in wheel
chmod 0775 /lab/
echo "UUID=00000000-1111-2222-3333-4444444444   /lab    btrfs   defaults,noatime,compress=zstd:1,x-systemd.device-timeout=0 0 0" >> /etc/fstab
systemctl daemon-reload
mount -a
```

Relocate the files and symlink them in the original location.
```
mv ~/.local/share/gnome-boxes /lab/gnome-boxes
ln -s /lab/gnome-boxes/ ~/.local/share/gnome-boxes
```