---
title: "Certified Wireless Network Administrator (CWNA-109)"
date: 2026-10-21
lastmod: 2026-10-21
slug: cwna-109
type: posts
draft: false
categories:
  - Networking
tags:
  - wlan
  - cwnp
  - certification
---
Notes for the CWNP Certified Wireless Network Administrator certification.

## 802.11 Standards

### Most common PHYs

| Amendment | PHY | Bands | Modulation | Max Bandwith | Max Rate | Max Streams |
| --- | --- | --- | --- | --- | --- | --- | 
| 802.11-1997 | DSSS | 2.4 GHz | DSSS | 22 MHz | 2 Mbps | 1 stream |
| 802.11b-1999 | HR/DSSS | 2.4 GHz | CCK,DSSS | 22 MHz | 11Mbps | 1 stream |
| 802.11a-1999 | OFDM | 5 GHz | OFDM | 20 MHz | 54 Mbps | 1 stream |
| 802.11g-2003 | ERP | 2.4 GHz | OFDM | 20 MHz | 54 Mbps | 1 stream |
| 802.11n-2009 (Wi-Fi 4)| HT | 2.4/5 GHz | MIMO OFDM (64-QAM) | 40 MHz | 600 Mbps | 4 streams |
| 802.11ac-2013 (Wi-Fi 5) | VHT | 5 GHz | DL MU-MIMO OFDM (256-QAM) | 160 MHz | 6933 Mbps | 8 streams |
| 802.11ax-2021 (Wi.Fi 6/6E) | HE | 2.4/5/6 GHz | UL/DL MU-MIMO OFDMA (1024-QAM) | 160 MHz | 9602 Mbps | 8 streams |
| 802.11be-2024 (Wi-Fi 7) | EHT | 2.4/5/6 GHz | UL/DL MU-MIMO OFDMA (4096-QAM) | 320 MHz | 23 Gbps | 8 streams |
| 802.11bn-20.. (Wi-Fi 8) | UHR | 2.4/5/6 GHz | ... | ... | ...  | ... |

Notes for 802.11n:
* Typically no channel 146 (introduced in 802.11ac)
* 4 spatials streams in the standard, but only 3x3:3 in reality
* Frame Aggregation (A-MSDU, A-MPDU)
* TxBF introduction
* Falls back to 54 Mbps if WPA is used

Notes for 802.11ac:
* ONLY in 5 GHz band (other bands may have b/g/n)
* TxBF standardized to an option
* TxOP (transmit opportunity) with EDCA queues
* Falls back to 54 Mbps if WPA is used

### Less common PHYs
| PHY | Name | Bands | Modulation | Max Bandwith | Max Rate | Max Streams |
| --- | --- | --- | --- | --- | --- | --- | 
| 802.11af | TVHT | 54-790 MHz | MIMO OFDM | 32 MHz | 568 Mbps | 4 streams |
| 802.11ah | S1G | 868 MHz (EU) | MIMO OFDM | 16 MHz | 346 Mbps | 4 streams |
| 802.11ad | DMG | 60 GHz | Single carrier | 2160 MHz | 6.7 Gbps | N/A |

### Non-PHY

| Name | What it's about |
| --- | --- |
| 802.11c-1998 | Updates 802.11D bridging for 802.11 operations |
| 802.11d-2001 | Specifications for regulatory domains |
| 802.11h-2003 | TPC & DFS |
| 802.11i-2004 | Robust Security Network with CCMP/AES, TKIP, DOT1X or PSK |
| 802.11e-2005 | QoS |
| 802.11k-2008 | TPC outside 5 GHz band |
| 802.11r-2008 | Fast Roaming for BSS transitions |
| 802.11w-2009 | Protected Management Frames |
| 802.11s-2011 | Interoperable mesh ESS |
| 802.11u-2011 | Interworking with WiMAX |
| 802.11ae-2012 | Prioritization of management frames |

## 802.11 Frames

### PHY Preamble

| Acronym | Definition |
| --- | --- |
| L | Legacy (non-HT, before 802.11n) |
| STF | Short Training Field |
| LTF | Long Training Field |
| SIG | Signal |
| HT | High Troughput (802.11n) |
| VHT | Very High Troughput (802.11ac) |
| GF | Greenfield |
| HE | High Efficiency (802.11ax) |

| PPDU type | PLCP Header |
| --- | --- |
| Non-HT (legacy) | `L-STF + L-LTF + L-SIG` |
| Mixed format | `L-STF + L-LTF + L-SIG` + HT-SIG + **HT-STF** + `HT-LTF + ... + HT-LTF + (E)HT-LTF + ... + (E)HT-LTF` |
| HT Greenfield | **HT-GF-STF + HT-LTF1** + HT-SIG + `HT-LTF + ... + HT-LTF + (E)HT-LTF + ... + (E)HT-LTF` |
| VHT | `L-STF + L-LTF + L-SIG` + VHT-SIG-A + VHT-STF + VHT-LTF + VHT-SIG-B |
| HE SingleUser | `L-STF + L-LTF + L-SIG` + `RL-SIG + HE-SIG-A(8us)` + `HE-STF(4us) + HE-LTF + ... + HE-LTF` |
| HE MultiUser | `L-STF + L-LTF + L-SIG` + `RL-SIG + HE-SIG-A(8us)` + HE-SIG-B + `HE-STF(4us) + HE-LTF + ... + HE-LTF` |
| HE ER SU | `L-STF + L-LTF + L-SIG` + `RL-SIG + HE-SIG-A(16us)` + `HE-STF(4us) + HE-LTF + ... + HE-LTF`|
| HE TB | `L-STF + L-LTF + L-SIG` + `RL-SIG + HE-SIG-A(8us)` + `HE-STF(8us) + HE-LTF + ... + HE-LTF` | 

### General format
Format of the **MPDU** that will be transmitted to the Physical Layer (where it name changes to PSDU).
```
  ◄──────────────────────────MPDU/PSDU───────────────────────────► 
  ┌─────────────────────────┌─────────────────┌──────────────────┐ 
  │ MAC Header              │ Payload (MSDU)  │ Footer           │ 
  │ 802.11 Information      │ User Data       │ Error Correction │ 
  └─────────────────────────└─────────────────└──────────────────┘ 
```

Several MSDU can be aggregated in in the same MPDU if they share the same MAC Header and are then called **A-MSDU**.
```
 ◄─────────────────────────────────────────────────────PPDU──────────────────────────────────────────────────► 
           ◄─────────────────────────────────────────────MPDU────────────────────────────────────────────────► 
 ┌─────────┌─────────────────────────┐─────────────────┌──────────────────┌─────────────────┌────────────────┐ 
 │ PLCP    │ MAC Header              │ Payload (A-MSDU)│ Payload (A-MSDU) │Payload (A-MSDU) │Footer          │ 
 │ Header  │ 802.11 Information      │ User Data       │ User Data        │User Data        │Error Correction│ 
 └─────────└─────────────────────────┘─────────────────└──────────────────└─────────────────└────────────────┘ 
```

Several MPDU can be aggregated in the same by the frame before beeing sent on the air by the PHY layer. They are then called **A-MPDU**. Each MPDU has its own MAC Header.
```
  ◄──────────────────────────────────────────────PPDU───────────────────────────────────────────────►  
 ┌────────┌───────────────────────┌────────────────────────┌───────────────────────┌─────────────────┐ 
 │ PLCP   │ A-MPDU                │ A-MPDU                 │ A-MPDU                │ Footer          │ 
 │ Header │ MAC Header + User Data│ MAC Header + User Data │ MAC Header + User Data│ Error Correction│ 
 └────────└───────────────────────└────────────────────────└───────────────────────└─────────────────┘ 
```

### MAC Header
Detail of the **MAC Header** (field sizes are in bytes).
```
 ┌────────────┌──────────┌──────┌─────────┌─────────┌──────────────┌─────────┌─────────┌────────┐  
 │Frame       │Duration/ │Addr1 │Addr2    │Addr3    │Sequence      │Addr4    │QoS Ctrl │HT Ctrl │  
 │Control (2) │ID (2)    │(6)   │(0 or 6) │(0 or 6) │Ctrl (0 or 2) │(0 or 6) │(0 or 2) │(0 or 2)│  
 └────────────└──────────└──────└─────────└─────────└──────────────└─────────└─────────└────────┘  
```

Detail of the MAC Header **Frame Control** field (sizes are in bits).
```
 ┌────────────┌─────┌────────┌──────┌────────┌───────────────┌──────┌───────────┌──────────┌──────────┌──────────┐
 │Protocol    │Type │Subtype │To DS │From DS │More Fragments │Retry │Power Mgmt │More Data │Protected │+HTC/Order│
 │Version (2) │(2)  │(2)     │(1)   │(2)     │(1)            │(1)   │(1)        │(1)       │Frame (1) │(1)       │
 └────────────└─────└────────└──────└────────└───────────────└──────└───────────└──────────└──────────└──────────┘
```

Notes about fields
* Protocol Version : always `00`.
* Power management : STA sets it to `1` if it sleeps after this frame.
* More Data : I have more data for you after this frame, do not sleep.

| Management Frame (Type `00`) | Subtype |
| --- | --- |
| Beacon | `1000` |
| Association Request | `0000` |
| Association Response | `0001` |
| Authentication | `1011` |
| Deauthentication | `1100` |
| Action | `1101` |
| Action No Ack | `1110` |

| Control Frame (Type `01`) | Subtype |
| --- | --- |
| Control Wrapper | `0111` |
| Block ACK request | `1000` |
| Block ACK | `1001` |
| PS-Poll | `1010` |
| RTS | `1011` |
| CTS | `1100` |
| AKC | `1101` |

| Data Frame (Type `10`) | Subtype |
| --- | --- |
| Standard | `0000` |
| Null | `0100` |
| QoS | `1000` |
| Qos Null | `1110` |

| To DS | From DS | Signification |
| --- | --- | --- |
| `0` | `0` | IBSS (STA to STA), BSS (direct STA to STA), Management or Control Frames |
| `0` | `1` | STA to DS (frame will leave the WLAN infrastructure) |
| `1` | `0` | DS to STA (frame coming from outside the WLAN infrastructure), Mesh (3 addresses format) |
| `1` | `1` | Mesh (4 addresses format) |
