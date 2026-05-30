---
title: "Android tricks"
date: 2024-09-22
lastmod: 2026-05-30
slug: android
type: posts
draft: false
categories:
  - Android
tags:
  - android
  - debug
  - adb
  - root
  - bluetooth
  - lineageos
  - wireshark
---
Android tricks using ADB

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

## Remote control
With [SCRCPY](https://scrcpy.org/) you just need ADB (USB or WLAN) to remote-control your Android device from your PC.
