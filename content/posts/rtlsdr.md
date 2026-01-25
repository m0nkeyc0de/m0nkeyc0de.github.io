---
title: "Software Defined Radio with RTL2832U"
date: 2026-01-25
slug: rtlsdr
type: posts
draft: false
tags:
  - sdr
---
I'v got a cheap blue DVB-T+FM+DAB dongle to play with SDR.

## The hardware
![Blue dongle](/rtlsdr/dongle.png)

One chip on the PCB is marked `RTL2832U`.
![RTL2832U](/rtlsdr/dongle_rtl2832u.png)

The other one is marked `R820T2`. It matches what written on the case.
![R820T2](/rtlsdr/dongle_r820t2.png)

According to USB data it's a *RTL2838* but according to [Osmocom wiki](https://osmocom.org/projects/rtl-sdr/wiki), RTL2838 chip do not exist.
```
Bus 003 Device 014: ID 0bda:2838 Realtek Semiconductor Corp. RTL2838 DVB-T
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 [unknown]
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x0bda Realtek Semiconductor Corp.
  idProduct          0x2838 RTL2838 DVB-T
  bcdDevice            1.00
  iManufacturer           1 Realtek
  iProduct                2 RTL2838UHIDIR
  iSerial                 3 00000001
  bNumConfigurations      1
```

The tuner is physically a Rafael Micro 820T2, which provides the frequency range of 24 MHz to 1766 MHz according to Oscom's Wiki.

## Getting it to work on Linux Mint 22
Some RTL SDR libs exist in the official repos. Let's install them.
```
apt install rtl-sdr librtlsdr2 librtlsdr-dev
```

Osmocom software and GNU radio blocks are also there.
```
apt install osmo-sdr gnuradio gr-osmosdr
```

`rtl_test` output
```
Found 1 device(s):
  0:  Realtek, RTL2838UHIDIR, SN: 00000001

Using device 0: Generic RTL2832U OEM
Detached kernel driver
Found Rafael Micro R820T tuner
Supported gain values (29): 0.0 0.9 1.4 2.7 3.7 7.7 8.7 12.5 14.4 15.7 16.6 19.7 20.7 22.9 25.4 28.0 29.7 32.8 33.8 36.4 37.2 38.6 40.2 42.1 43.4 43.9 44.5 48.0 49.6 
```

Checking for a known FM radio signal presence with `osmocom_fft --samp-rate=2.554M --center-freq=100M --waterfall`:
![Osmocom FM waterfall](/rtlsdr/osmocom_fft_fm_waterfall.png)


Testing FM reception with [GNU Radio](https://wiki.gnuradio.org/index.php?title=RTL-SDR_FM_Receiver):
![FM FFT](/rtlsdr/gnuradio_fft.png)

It works !
