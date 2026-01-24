---
title: "Android Debug Bridge tricks"
date: 2024-09-22
slug: android_adb
type: posts
draft: false
categories:
  - Android
tags:
  - android
  - debug
---
ADB tricks

# Commands
List installed applications
```
adb shell pm list packages
```

Get application PID
```
adb shell pidof -s com.application.name
```

Logs for given PID
```
adb logcat --pid=8064
```

Screenshot
```
adb exec-out screencap -p > "$(date).png"
```
