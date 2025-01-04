---
title: "File transfer stuck at 99% on Linux"
date: 2024-01-21T10:08:53+01:00
slug: dirty_bytes
type: posts
draft: false
categories:
  - Linux
tags:
  - storage
---
When copying a big file to an USB drive, the copy seems to be stuck at 99% even if it's not the case.

In fact from the point of view of the copy command (GUI or CLI) the file transfer is almost finished but from the OS point of view data is still syncing from the RAM to the flash drive (which has a lower write speed than the read speed of the SSD to your RAM). In order to know when the copy is finished, the `sync` command can be run (it may take `d)

```
[root@fedora ~]# sync
```

It impacts more systems with a lot of RAM as **vm.dirty_ratio** is set instead of a fixed amount of bytes. Here are Fedora 39 defaults:

```
[root@fedora ~]# sysctl vm.dirty_ratio
vm.dirty_ratio = 20
[root@fedora ~]# sysctl vm.dirty_bytes
vm.dirty_bytes = 0
```

In order to *align* those transfer speeds and have an almost consistent progress bar, the **dirty bytes** kernel setting can be tweaked.

```
# Temporarly set vm.dirty_bytes to 10Mb
[root@fedora ~]# sysctl vm.dirty_bytes=10000000
vm.dirty_bytes = 10000000

# Ratio is now unused and set to 0
[root@fedora ~]# sysctl vm.dirty_ratio
vm.dirty_ratio = 0
```

Note that setting it permanently to a low value may impact performance when transfering data between two fast storage devices.
