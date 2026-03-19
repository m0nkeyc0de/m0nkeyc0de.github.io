---
title: "WLAN access point with hostapd"
date: 2026-03-19
slug: wlanaphostapd
type: posts
draft: false
categories:
  - Wi-Fi
tags:
  - wlan
  - linux
  - hostapd
---
With **hostapd** you can quickly and easily run an access point. Sometimes it can be useful to diffuse a temporary SSID and/or analyze the traffic from a specific device with Wireshark. Inspiration is taken from [Matt Brown's mitmrouter project](https://github.com/nmatt0/mitmrouter) which configures the entire network stack manually. Here we leverage **NetworkManager** instead.

# Hardware considerations
As usual when messing around with WLAN, adapter support is hit-or-miss:

* [**Alfa Network AWUS036AXML**](https://www.alfa.com.tw/products/awus036axml) adapter, based on a MediaTek MT7921AUN chipset works well.
* TP-Link Archer T3U adapter, based on a Realtek RTl8812BU chipset did not send any beacons out. Not great for an access point.
* Intel AX200 PCIe card works, but you end up with the international regulatory domain (see [the monitor mode article](wlan_capture.md) for more details).

# Uplink connectivity
Connected devices need at least to receive an IP address. They may also connect to internet. We're going to create `bridge0` interface with NetworkManager and apply connection sharing on it.

```bash
nmcli connection add type bridge \
    ifname bridge0 \
    connection.id bridge0 \
    connection.autoconnect yes \
    ipv4.method shared \
    ipv4.addresses 172.31.255.1/24 \
    ipv6.method disabled \
    bridge.stp no \
    bridge.multicast-snooping no
    
nmcli con up bridge0
```

# Access point
Create a `hostapd.conf` file with following content.

```
interface=wlan0
bridge=bridge0
ssid=IOT
country_code=CH
hw_mode=g
channel=6
wpa=2
wpa_passphrase=ConnectingIOT
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
ieee80211n=1
#ieee80211w=1
```

Avoid NetworkManager to interfer with your chosen AP interface.
```bash
$ nmcli device set wlan0 managed no
```

Start `hostapd`.
```bash
sudo hostapd hostapd.conf
```

Verify that your SSID is visible.
```bash
nmcli dev wifi
```
