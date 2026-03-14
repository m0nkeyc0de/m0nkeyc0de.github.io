---
title: "OTA WLAN packet capture"
date: 2026-03-12
slug: wlan_capture
type: posts
draft: false
categories:
  - Linux
  - Wi-Fi
tags:
  - linux
  - wifi
  - wlan
---
Capturing raw WLAN frames Over-The-Air isn't the most straightforwad thing to do. This articles paves the way for achieving that easily on Linux with cheap hardware. Cherry on the cake : full 6GHz spectrum is supported (up to Wi-Fi 6E).

# Capture adapter
The capture adapter must have drivers that support **monitor mode**. The [Linux Kernel documentation](https://wireless.docs.kernel.org/en/latest/en/users/drivers.html) and the [USB-WiFi project](https://github.com/morrownr/USB-WiFi) provide valuable information.

The [**Alfa Network AWUS036AXML**](https://www.alfa.com.tw/products/awus036axml) adapter, based on a MediaTek MT7921AUN chipset, works pretty well.

# Enabling monitor mode
In the monitor mode, all the 802.11 frames and their headers can be captured (in opposition to the "normal" mode where the driver strips all that information).

To reliably enable monitor mode, use the `airmon-ng` command from the [aircrack-ng suite](https://www.aircrack-ng.org/). Be aware that NetworkManager may ruin your efforts as it may compete (and probably win) for the control of the card.

```bash
# Calm NetworkManager down
nmcli device set wlan1 managed no

# Start monitor mode
airmon-ng start wlan1
```

Now you can see all the 802.11 frames on `wlan1mon` with Wireshark or tcpdump.

# Selecting the capture channel
First check the active regulatory domain and available frequencies with `iw reg get`. If it doesn't match your country, try to set the regulatory domain with `iw reg set CH`, if your adapter isn't *self-managed* (good luck for that if you have an Intel card...).

For channels in the 2.4GHz or 5GHz band, you can chose channel by number.
```
iw dev wlan1mon set channel 36
```

For 6GHz channels, you'll have to set the center frequency (don't ask me why).
```
iw dev wlan1mon set freq 6135
```

You'll find a list of channels with their center frequency on [Wikipedia](https://en.wikipedia.org/wiki/List_of_WLAN_channels).

Note: My attempts to run monitor mode in 6GHz with an Intel BE200 were defeated by the firmware, which is unable to keep the current country code in monitor mode. As a result, the adapter falls back to the frequencies allowed worldwide, which excludes the entire 6GHz spectrum.
