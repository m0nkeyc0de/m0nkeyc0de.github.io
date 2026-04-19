---
title: "Windows 11 IoT in QEMU"
date: 2026-03-08
slug: qemu_w11
type: posts
draft: false
categories:
  - Virtualization
tags:
  - qemu
  - linux
---
Sometimes you absolutely need Windows to run a piece of software. This page explains how to install and run Windows 11 IoT LTSC into QEMU in less than 30 minutes.

## Windows 11 IoT LTSC
This is the best version of Windows so far as it contains no bloat and doesn't require TPM. The catch is that [Microsoft doesn't want you to use it](https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/deployment/volume-license?pivots=windows11). 

Nothing is perfect and here are the non-technical limitations (Microsoft is what it is...) so far:
* You can't officially buy a license for it.
* Will only work for 90 days without beeing activated (then it'll reboot every hour).
* The online account creation also needs to by bypassed.

The ISO file can be downloaded from [here](https://www.microsoft.com/en-us/evalcenter/download-windows-11-iot-enterprise-ltsc-eval).

Here is the way I used to [bypass Microsoft Account](https://pureinfotech.com/bypass-microsoft-account-setup-windows-11/) dark pattern:
* Start the terminal with `SHIFT+F10` on the Region Settings page
* Execute `reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\OOBE /v BypassNRO /t REG_DWORD /d 1 /f`
* Reboot with `shutdown /r /t 0`
* Start the terminal with `SHIFT+F10` on the Region Settings page.
* Disconnect from network with `ipconfig /release`
* Click on `I don't have internet` when it appears.

## Bridge network
If you need to access internet or other network ressources, you'll need a NAT-ed bridge.

In **NetworkManager**, create a bridge named `bridge0` and select `Share with other computers` in the IPv4/IPv6 sections. Bring this connection up.

## QEMU
Windows 11 IoT can be started in QEMU [with following command](https://computernewb.com/wiki/QEMU/Guests/Windows_10) (as no TPM is needed, the Windows 10 way works fine):
```bash
# Sudo is required for bridge nic
sudo qemu-system-x86_64 \
  -M q35,usb=on,acpi=on,hpet=off \
  -m 6G \
  -cpu host,hv_relaxed,hv_frequencies,hv_vpindex,hv_ipi,hv_tlbflush,hv_spinlocks=0x1fff,hv_synic,hv_runtime,hv_time,hv_stimer,hv_vapic \
  -smp cores=4 \
  -accel kvm \
  -drive file=win11.qcow2 \
  -device usb-tablet \
  -device VGA,vgamem_mb=256 \
  -monitor stdio \
  -nic bridge,model=e1000,br=bridge0
```

Steps for installation:
* Create the disk QCOW image with `qemu-img create -f qcow2 win11.qcow2 40G`
* Add the ISO image as cdrom to qemu command line `-cdrom ~/Downloads/26100.1742.240906-0331.ge_release_svc_refresh_CLIENT_IOT_LTSC_EVAL_x64FRE_en-us.iso`
