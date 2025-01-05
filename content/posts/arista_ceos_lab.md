---
title: "Network lab Arista cEOS-lab"
date: 2025-01-04T12:08:53+01:00
slug: arista_ceos_lab
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
Work in progress...

## Lab environment
All the lab runs on a *HP EliteDesk 800 G5 mini* (with an *Intel Core i5-9500* CPU and 16GB of RAM) running *Ubuntu 24.04 LTS*.

Arista cEOSlab Docker image can be downloaded after creating an account on the [Arista Customer Support page](https://www.arista.com/en/support/customer-support).

Differences with the bare-metal version:
* The `reload` command is not working on cEOS due to platform limitation.
* The `terminal monitor` command has no effect as no logs are displayed when something happens.
* The default system MAC is 00:00:00:00:00:00 and this needs to be tweaked to have LACP/MLAG working.
* Network interfaces are named `ethX` as OS level but OSPF and ISIS need `etX` names. This must also be tweaked.

## Arista cEOS with containerlab
[Containerlab](https://containerlab.dev) is specially made for running network devices containers.
* [Quickstart](https://containerlab.dev/quickstart/)
* [Arista cEOS](https://containerlab.dev/manual/kinds/ceos/)

The cEOS image with default paramters (non-persistent */mnt/flash*) is needed for this setup.
```
sudo docker import --change 'VOLUME /mnt/flash/' cEOS64-lab-4.33.1F.tar.xz ceos64lab:4.33.1F
```


## Arista cEOS with GNS3
**NOTE**: after messing several hours with GNS3 and cEOS, it works but this is far from the perfect lab solution:
* Daring to change the hostname through GNS3 GUI doesn't work and will just throw you an error message every 20 seconds.
* Some default configuration must be applied to new cEOS nodes and there is no straightforward way to automate that with GNS3.
* The web interface is clunky therefore the "heavy" client is mandatory for an optimal experience.

**This isn't the best way to go.**

### Installation

Installation works perfectly by following the [documentation](https://docs.gns3.com/docs/getting-started/installation/remote-server).
```
curl https://raw.githubusercontent.com/GNS3/gns3-server/master/scripts/remote-install.sh > gns3-remote-install.sh
bash gns3-remote-install.sh --with-iou --with-i386-repository
```
### HTTP GUI
There is a HTTP user interface on port 3080 but it doesn't work very well with Firefox or Brave:
* The `xdg-open` actions need to access Auxiliary console are a nightmare to configure, at least on Gnome Desktop. [This](https://github.com/GNS3/gns3-webclient-pack) may help but I did not test it.
* Drawed lines and links cannot be selected. 

For now, it's a no go !

### Heavy client
Package `gns3-client` must be installed on Fedora.

Terminator can be used as console application with the following command line (title does not work on tabs):
```
/usr/bin/terminator --new-tab -T "{name}" -x telnet {host} {port}
```

### cEOS setup for GNS3
Arista documentation:

* [cEOS-lab in GNS3](https://arista.my.site.com/AristaCommunity/s/article/ceos-lab-in-gns3)
* [vEOS/cEOS GNS3 Labs](https://arista.my.site.com/AristaCommunity/s/article/veos-ceos-gns3-labs)

Import the docker image in persistent */mnt/flash* volume and create a template in GNS3.
```
sudo docker import --change 'VOLUME /mnt/flash/' cEOS64-lab-4.33.1F.tar.xz ceos64lab-persistent:4.33.1F
```

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

Then you can add a device to your topology and start it. To enter in the shell you need to open the auxiliary console and run `FastCli` command, unless the Management0 interface is available.

System MAC and interface naming scheme have to be tweaked but it can only be done manually in GNS3 as far as I've seen. As I want my lab setups to be as automated as possible, the GNS3 experience stops here.
