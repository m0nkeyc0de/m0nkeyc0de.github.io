---
title: "BTHome on Home Assistant is insecure"
date: 2026-11-01
lastmod: 2026-11-01
slug: bthome-ha-security
type: posts
draft: false
categories:
  - Security
tags:
  - ble
  - bthome
  - home assistant
---
Is the Home Assistant BTHome implementation done well ? Absolutely not !

# Stuff needed
Same tools as in [BTHome Analysis post](/posts/bthome-security):
* Home Assisant version 2026.10.4
* A [seeed studio XIAO-nRF52840-Plus](https://www.seeedstudio.com/Seeed-Studio-XIAO-nRF52840-Plus-p-6359.html) board to [generate BLE advertisements](/posts/ble-advertisements)
* My [forked version of nmatt0 bletools](https://github.com/m0nkeyc0de/bletools) to watch and decode advertisements live.
* The [bthome example script](https://github.com/Bluetooth-Devices/bthome-ble/blob/main/src/bthome_ble/bthome_v2_encryption.py) to generate the payloads.

# Test sequence

Below is the ordered test sequence.
| Counter value (little endian) | Description | Advertised payload | Result |
| --- | --- | --- | --- |
| `0x0000000A` | Functional test. Must work. | `D2FC41D890C2B10A000000A522591A` | Pass: detected and encryption key added |
| N/A | Unencrypted payload. Must not work. | `D2FC4400AB01643A01` and `D2FC4400EC01643A01` | Pass: button presses not showing up in activity |
| `0x0000000B` | Functional test. Must work | `D2FC419FACE9970B0000000693EFA3` | Pass: button press showing up in activity |
| `0x00000005` | Counter value decreased. Must not work. | `D2FC41F92213B9050000002C3B30EF` | Fail: button press showing up in activity. It shouldn't as counter decreased. |
| `0x0000000C` | Valid counter value. Must work again. | `D2FC41F08CB92E0C000000A31A1C90` | Pass: button press showing up in activity |
| `0x0000000C` | Replay attack. Must not work. | `D2FC41F08CB92E0C000000A31A1C90` | Pass: button presses not showing up in activity. It may be to avoid detecting several times the same advertisement. |
| `0xF0000000` | Big counter jump forward. Must work. | `D2FC41557712F8000000F04A101E93` | Pass: button press showing up in activity |
| `0xFFFFFFFF` | Biggest counter value possible. Must work. | `D2FC41CEB140D8FFFFFFFF6343DC0C` | Pass: button presses not showing up in activity |
| `0x00000000` | Counter overflow. Does it still work ? | `D2FC4104A875E5000000002E36D095` | Button press is showing up in activity |

# Conclusion
From the observed behavior, we can that checks done on counter aren't done well. Home Assistant is therefore exposed to BTHome replay attacks.

Instead of doing the proper check `received_counter_value > last_counter_value` it only verifies that values are not equal (like `received_counter_value != last_counter_value`) to differentiate one BLE advertisement from another.

BTHome events received by Home Assistant 2026.10.4 cannot be fully trusted.
