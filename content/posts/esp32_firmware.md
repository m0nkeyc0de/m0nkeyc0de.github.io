---
title: "Arduino ESP32 firmware backup"
date: 2026-03-27
lastmod: 2026-04-18
slug: esp32_firmware
type: posts
draft: false
categories:
  - Miscellanous
tags:
  - arduino
  - esp32
  - esptool
---
This article explains how to extract an Arduino firmware to have a backup of it, and hopefully beeing able to restore it one day.

The hardware I'm using is a [M5 Core2](https://docs.m5stack.com/en/core/core2) which my probably broken [Condair Cube Air Quality Monitor](https://www.condair.at/condair-cube) unit with crazy CO2 values is made of. Connecting the it over USB generates following `dmesg` logs:
```
[ 1826.600703] usb 3-6.4.1: new full-speed USB device number 15 using xhci_hcd
[ 1826.740947] usb 3-6.4.1: New USB device found, idVendor=1a86, idProduct=55d4, bcdDevice= 4.43
[ 1826.740951] usb 3-6.4.1: New USB device strings: Mfr=0, Product=2, SerialNumber=3
[ 1826.740952] usb 3-6.4.1: Product: USB Single Serial
[ 1826.740953] usb 3-6.4.1: SerialNumber: xxxxxxxxx
[ 1827.303138] cdc_acm 3-6.4.1:1.0: ttyACM0: USB ACM device
```

## Backup
The entire flash can be read with the [esptool](https://docs.espressif.com/projects/esptool/en/latest/esp32/esptool/basic-commands.html#read-flash-contents-read-flash) command (get it from [Pypi](https://pypi.org/project/esptool/)):
```
$ esptool -p /dev/ttyACM0 -b 460800 read-flash 0 ALL condair_cube_v2.0.bin
esptool v5.2.0
Connected to ESP32 on /dev/ttyACM0:
Chip type:          ESP32-D0WDQ6-V3 (revision v3.0)
Features:           Wi-Fi, BT, Dual Core + LP Core, 240MHz, Vref calibration in eFuse, Coding Scheme None
Crystal frequency:  40MHz
MAC:                08:3a:f2:xx:xx:xx

Stub flasher running.
Changing baud rate to 460800...
Changed.

Detected flash size: 16MB
Configuring flash size...
Read 16777216 bytes from 0x00000000 in 438.7 seconds (306.0 kbit/s) to 'condair_cube_v2.0.bin'.

Hard resetting via RTS pin...
```

## Restore
Execute the following command and you're good to go.
```bash
esptool -p /dev/ttyACM0 -b 460800 write-flash 0 condair_cube_v2.0.bin
```
