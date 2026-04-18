---
title: "Android Debug Bridge tricks"
date: 2024-09-22
lastmod: 2026-04-18
slug: android_adb
type: posts
draft: false
categories:
  - Android
tags:
  - android
  - debug
  - adb
---
Useful ADB commands

## Processes and apps
List installed applications
```bash
adb shell pm list packages
```

Get application PID
```bash
adb shell pidof -s com.application.name
```

Logs for given PID
```bash
adb logcat --pid=8064
```

## Screenshots
```bash
adb exec-out screencap -p > "$(date).png"
```

## ADB as root
LineageOS provides `Rooted Debugging` in Developper Options.
![Rooted Debugging](/adb/android_rooted_debugging.png)

Once enabled you can start ADB in root mode.

```bash
adb root
# ADB restarts
adb shell
```

## Bluetooth packet capture
Enable `Bluetooth HCI snoop log` in Developper Options. 
![Bluetooth HCI Snoop](android_bluetooth_hci_log_snoop.png).

It generates a log file that can directly be opened with Wireshark.

If you can start ADB as root:
```bash
adb pull /data/misc/bluetooth/logs/btsnoop_hci.log
```

If you can't start ADB as root, you need to generate the bugreport archive (takes time).
```bash
adb bugreport bugreport.zip
```
The file is in the `FS/data/misc/bluetooth/logs` folder of the archive.
