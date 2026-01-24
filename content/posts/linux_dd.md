---
title: "Raw data copy with dd on Linux"
date: 2026-01-24
slug: linux_dd
type: posts
draft: false
tags:
  - storage
  - linux
---
dd is a great tool to do a raw (or binary) copy of a storage device or a file.

## Basic dd usage
In most cases this command will do the trick.

```
dd if=/path/to/source of=/path/to/destination status=progress
```

You may optimize the block size according to your source/destination (it depends...) device for improved copy speed, with the `bs=...` command line switch.

## Create an ISO image from a CD/DVD

CDs and DVDs have blank bytes at the end and those may break your checksum and bloat your image file.

Get ISO informations to know the real size.

```
isoinfo -d -i /dev/cdrom
```

Lines `Logical block size is: 2048` and `Volume size is: 187723` contain the information we need for `dd`.

Launch the copy with the values gathered above.
```
dd if=/dev/cdrom of=cdrom.iso bs=2048 count=187723 status=progress
```

## Sources
* https://www.thomas-krenn.com/en/wiki/Create_an_ISO_Image_from_a_source_CD_or_DVD_under_Linux
