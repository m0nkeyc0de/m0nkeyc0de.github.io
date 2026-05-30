---
title: "Bluetooth/BLE traffic capture"
date: 2026-05-30
lastmod: 2026-05-30
slug: bluetooth-capture
type: posts
draft: false
categories:
  - Networking
tags:
  - android
  - adb
  - root
  - bluetooth
  - wireshark
  - nrf52840
  - ble
---
Tips and tricks on how to capture Bluetooth and BLE packets.

## Bluetooth and BLE PCAP file with Android 
This is the easiest way to capture Bluetooth/BLE traffic if the communication only occurs between your phone and a device.

Enable `Bluetooth HCI snoop log` in Developper Options. 
![Bluetooth HCI Snoop](/adb/android_bluetooth_hci_log_snoop.png)

It generates a file that can directly be opened with Wireshark.

If you can start [ADB as root](/posts/android):
```bash
adb pull /data/misc/bluetooth/logs/btsnoop_hci.log
```

If you can't start ADB as root, you need to generate the bugreport archive (takes time).
```bash
adb bugreport bugreport.zip
```
The file is in the `FS/data/misc/bluetooth/logs` folder of the archive.

For BLE traffic, use the `btatt` Wireshark display filter.

## BLE Advertisements with nRF Connect for Mobile
If you don't need/want a full blown PCAP file and just look at BLE Advertisements payloads, [nRF Connect for Mobile](https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile) is a good choice.

![nRFConnect for Mobile](/ble/nrfconnect_for_mobile.png)

## BLE capture with nRF52840 Dongle
The [Nordic Semiconductors nRF52840 Dongle](https://www.nordicsemi.com/Products/Development-hardware/nrf52840-dongle) can be used as [sniffer for Bluetooth LE](https://www.nordicsemi.com/Products/Development-tools/nRF-Sniffer-for-Bluetooth-LE) directly in Wireshark once programmed properly with [nrfutil](https://www.nordicsemi.com/Products/Development-tools/nRF-Util/Download).

```bash
nrfutil install completion
nrfutil install device
nrfutil install nrf5sdk-tools
nrfutil install ble-sniffer

# Flash the dongle
nrfutil device program --serial-number FBF37Exxxxxx--firmware ~/.nrfutil/share/nrfutil-ble-sniffer/firmware/sniffer_nrf52840dongle_nrf52840_4.1.1.zip

# Wireshark stuff
# https://docs.nordicsemi.com/bundle/nrfutil/page/guides/installing_wireshark.html
mkdir -p ~/.local/lib/wireshark/extcap
nrfutil ble-sniffer bootstrap
# A capture device like /dev/ttyACM0-4.6 (nRF Sniffer for Bluetooth LE) should be present
# There is a BLE toolbar where filtering can be done
```

You should have a `nRF Sniffer for Bluetooth LE` USB device which can be used in Wireshark.
```
Bus 001 Device 008: ID 1915:522a Nordic Semiconductor ASA nRF Sniffer for Bluetooth LE
Negotiated speed: Full Speed (12Mbps)
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 [unknown]
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x1915 Nordic Semiconductor ASA
  idProduct          0x522a nRF Sniffer for Bluetooth LE
  bcdDevice            2.04
  iManufacturer           1 ZEPHYR
  iProduct                2 nRF Sniffer for Bluetooth LE
  iSerial                 3 xxxxxxxxxxxxxxxxxxx
  bNumConfigurations      1
```

![nRF52840 Wireshark](/ble/wireshark_nrf52840.png)

## BLE capture with Python and bleak
Have a look at [bletools from Matt Brown](https://github.com/nmatt0/bletools).
