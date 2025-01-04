---
title: "Fixing Linux command permissions"
date: 2025-01-04T10:08:53+01:00
slug: linux_command_permissions
type: posts
draft: false
categories:
  - Linux
tags:
  - fedora
  - sudo
  - setuid
  - tcpdump
  - tshark
  - wireshark
  - minicom
  - picocom
---
Some often used commands don't work out-of-the-box with an unprivileged user. This page will explain how to get around.

## Sudoers
For applications that must be run as root, adding configuration in `/etc/sudoers.d/` is a solution. 

My handy-dandy commands for Fedora:
```
%wheel ALL=(root) NOPASSWD:/usr/bin/dnf update
%wheel ALL=(root) NOPASSWD:/usr/bin/sync
%wheel ALL=(root) NOPASSWD:/usr/bin/nmap
%wheel ALL=(root) NOPASSWD:/usr/bin/fastboot
%wireshark ALL=(root) NOPASSWD:/usr/sbin/tcpdump
```

## Wireshark or tshark as non-root
Lauching `wireshark` or `tshark` as root is discouraged by the application itself.

To capture packets with a non-root user, just add it to the `wireshark` group.

## ttyUSB
By default accessing `/dev/ttyUSBx` with `picocom` or `minicom` does not work with standard privileges.

Add your user to the group `dialout` and the problem is solved.
```
[root@fedora ~]# ls -lh /dev/ttyUSB0 
crw-rw----+ 1 root dialout 188, 0  4 jan 16:41 /dev/ttyUSB0
```

## socket: Operation not permitted
This happens with `ping` command in Debian WSL (or with the `tcpdump` command). Instead of running them with `sudo`, the `setuid` bit can be set on the executable.

```
[root@fedora ~]# ls -lh /bin/ping
-rwxr-xr-x. 1 root root 162K  7 sep 02:00 /bin/ping

[root@fedora ~]# chmod u+s /bin/ping

[root@fedora ~]# ls -lh /bin/ping
-rwsr-xr-x. 1 root root 162K  7 sep 02:00 /bin/ping
```
**CAUTION** : there may be security concerns as the command is now executed with root privileges for every user (you could overwrite anything on your system with the `-w` option of `tcpdump`). Using `sudoers` is a safer solution.