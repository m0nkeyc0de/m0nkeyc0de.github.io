---
title: "Network lab with Arista cEOS-lab on GNS3"
date: 2025-01-04T10:08:53+01:00
slug: gns3_ceoslab
type: posts
draft: false
categories:
  - Linux
  - Networking
tags:
  - ubuntu
  - gns3
  - arista
---
Having a network lab on GNS3 is a good way to learn things without messing with your production network.

This guide isn't complete yet. Next things to be done:

* Solve the `xdg-open` problem when opening the *Auxiliary Console*
* Make lab device accessible through SSH and HTTPS through the management interface (eth0)
* Configure cEOS properly to have all features working well (MLAG and OSPF/ISIS)

## GNS3 installation
To not clutter my workstation, I installed the remote server on a dedicated *HP EliteDesk 800 G5 mini* (with an *Intel Core i5-9500* CPU and 16GB of RAM) running *Ubuntu 24.04 LTS*.

Installation works perfectly by following the [documentation](https://docs.gns3.com/docs/getting-started/installation/remote-server).
```
curl https://raw.githubusercontent.com/GNS3/gns3-server/master/scripts/remote-install.sh > gns3-remote-install.sh
bash gns3-remote-install.sh --with-iou --with-i386-repository
```

There is a HTTP user interface on port 3080.

## cEOS setup
I chosed cEOS-lab because it is lighter than vEOS-lab. It can be downloaded after creating an account on the [Arista Customer Support page](https://www.arista.com/en/support/customer-support).

Arista documentations:

* [cEOS-lab in GNS3](https://arista.my.site.com/AristaCommunity/s/article/ceos-lab-in-gns3)
* [vEOS/cEOS GNS3 Labs](https://arista.my.site.com/AristaCommunity/s/article/veos-ceos-gns3-labs)

Import the downloaded container image into docker.
```
sudo docker import --change 'VOLUME /mnt/flash/' cEOS64-lab-4.33.1F.tar.xz ceos64lab:4.33.1F
```

Create a new template in GNS3 using an existing docker image.

* Start command
```
/sbin/init systemd.setenv=INTFTYPE=eth systemd.setenv=ETBA=1 systemd.setenv=SKIP_ZEROTOUCH_BARRIER_IN_SYSDBINIT=1 systemd.setenv=CEOS=1 systemd.setenv=EOS_PLATFORM=ceoslab systemd.setenv=container=docker systemd.setenv=MGMT_INTF=eth0
```

* Environment variables (must match values of the start command)
```
INTFTYPE=eth
ETBA=1
SKIP_ZEROTOUCH_BARRIER_IN_SYSDBINIT=1
CEOS=1
EOS_PLATFORM=ceoslab
container=docker
MGMT_INTF=eth0
```

Then you can add a device to your topology and start it.

Note that:

* The main console will throw a lot of systemd errors when trying to start the logging service.
* You'll need to connect to the auxiliary console and run `FastCli` command to get a shell if you don't set up the management interface (or break it).
* The `terminal monitor` command has no effect : no logs are displayed.
* The `reload` command is not working on cEOS.