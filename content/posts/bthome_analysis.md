---
title: "BTHome security analysis"
date: 2026-06-30
lastmod: 2026-06-30
slug: bthome-security
type: posts
draft: true
categories:
  - Security
tags:
  - ble
  - shelly
  - nrf52840
---
Is my [garage door control with Shelly hardware](/posts/shellygarage) reasonably safe ? Let's find it out !

## Stuff needed
What you will need to reproduce this project:

* A [Shelly1 Gen3 relay](https://www.shelly.com/fr/products/shelly-1-gen3?variant=56273482547549) with firmware version 2.0.0
* A [Shelly BLU Button](https://www.shelly.com/fr/products/shelly-blu-button-tough-1-zb-ivory) with firmware version 1.0.23 and the [Shelly BLE Debug app](https://play.google.com/store/apps/details?id=cloud.shelly.bledebug&hl=en-US) to configure it.
* A [Nordic Semiconductors nRF52840 Dongle](https://www.nordicsemi.com/Products/Development-hardware/nrf52840-dongle) to [capture BLE frames](/posts/bluetooth-capture)
* A [seeed studio XIAO-nRF52840-Plus](https://www.seeedstudio.com/Seeed-Studio-XIAO-nRF52840-Plus-p-6359.html) board to [generate BLE advertisements](/posts/ble-advertisements)
* The [nRF Connect for Mobile app](https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile) to quickly control the generated payloads

## Understanding BTHome protocol
The [BTHome protocol](https://bthome.io/) is a well documented open standard everyone can use (well done Allterco !). 

Here is the BLE payload (without headers), thus the BTHome service data for the BLU Button.

| Field | Bytes | Description | Example values |
| --- | --- | --- | --- |
| UUID | 2 | BTHome messages start with `0xFCD2`. Bytes are reversed. | `0xD2FC` must be used to generate payloads. |
| BTHome device information | 1 | Flags indicating device capabilities. | `0x44` for unencrypted payloads, `0x41` when using encryption. |
| Object type | 1 | Indicates what value follows in next byte(s) | `0x00` for packet ID, `0x01` for battery level, `0x3A` for a button event |
| Object data | 1-2 | Data length depends on the object type | `0x01` for a button (`0x3A`) press, `0x64` for battery (`0x01`) at 100% |
| Object type | 1 | Next object type .. | ... |
| Object data | 1-2 | Next object data ... | ... |

Objects are chained together, up to the maximal possible payload size. In this experiment with the seeed studio XIAO-nRF52840-Plus, it wasn't possible to exceed 29 bytes.

Sample button push at 100% battery payloads : `D2FC 44 00EC 0164 3A01` and `D2FC 44 00AB 0164 3A01`

When not using encryption, the `Packet ID (0x00)` object is used to determine if a received packet belongs to the same transmission, as the BLE message is sent several times to ensure proper reception by the receiver. It must be different for each button press. When using encryption, this field is omitted as it becomes useless.

### Encryption

The (optional but strongly recommended) (BTHome encryption)[https://bthome.io/encryption/] is done using AES-CCM, which is a standard.

The encryption key is generated on the remote. It must be retrieved with the Shelly BLE Debug app, after pairing the remote with a smartphone. This encryption key is installed manually on the receiver through its web interface. Key exchange is done out-of-band, which is reasonably safe. The only weak point may be the pairing process between the smartphone running the app and the remote. This risk can be reduced by pairing in an isolated environment.

Replay attacks are avoided by incrementing a counter value, which is included in the nonce. Proper checks against this counter value must be done on the receiving side (counter value must be greather than the last one for the message to be accepted).

The counter is a 32 bits value, which allows 4294967296 payloads before it overflows. If an event is generated every second, it would take 136 years before overflow happens. In the case of a garage door, this may never happen. Nonetheless I'm curious to see what happens when the counter overflows.

The encryption design is solid on paper.

## Simulating the BLU Button

To simulate the BLU Button, I use my [DIY advertiser built with a XIAO-nRF52840-Plus](posts/ble-advertisements). Unencrypted payloads from above were used to confirm that advertisement are working correctly and are straightforward. Let's play with the encrypted payloads. Encryption is done using the [bthome example script](https://github.com/Bluetooth-Devices/bthome-ble/blob/main/src/bthome_ble/bthome_v2_encryption.py).

As there already is a counter with AES-CCM, `Packet ID (0x00)` field must not be included in encrypted data. Only `Battery (0x01)` and `Button (0x3A)` are kept : `0164 3A01` is our cleartext payload.

| Field | Bytes | Description | Example values |
| --- | --- | --- | --- |
| UUID | 2 | BTHome messages start with `0xFCD2`. Bytes are reversed. | `0xD2FC` must be used to generate payloads. |
| BTHome device information | 1 | Flags indicating device capabilities. | `0x41` when using encryption. |
| Ciphertext | Depends on data | Encrypted payload | `0xD890C2B1` |
| Counter | 4 | Little endian (like UUID) | `0x0A000000` |
| MIC | 4 | Message Integrity Check | `0xA522591A` |

Sample encrypted payload with `0x0000000A` in the counter : `D2FC 41 D890C2B1 0A000000 A522591A`

Below is the ordered test sequence.
| Counter value (little endian) | Description | Advertised payload | Result |
| --- | --- | --- | --- |
| `0x0000000A` | Functional test. Must work. | `D2FC41D890C2B10A000000A522591A` | |
| `0x0000000B` | Functional test. Must work | `D2FC419FACE9970B0000000693EFA3` | |
| `0x00000005` | Counter value decreased. Must not work. | `D2FC41F92213B9050000002C3B30EF` | |
| `0x0000000C` | Valid counter value. Must work again. | `D2FC41F08CB92E0C000000A31A1C90` | |
| `0x0000000C` | Replay attack. Must not work. | `D2FC41F08CB92E0C000000A31A1C90` | |
| `0xF0000000` | Big counter jump forward. Must work. | `D2FC41557712F8000000F04A101E93` | |
| `0xFFFFFFFF` | Biggest counter value possible. Must work. | `D2FC41CEB140D8FFFFFFFF6343DC0C` | |
| `0x00000000` | Counter overflow. Does it still work ? | `D2FC4104A875E5000000002E36D095` | |


## Evaluation of the actual security

Tests were done with encryption enabled.

| Control point | Explanation | Result |
| --- | --- | --- |
| Standard encryption | Used encryption protocols are well-recognized standards. No home-brew encryption algorithms are used. | Pass : AES-CCM is used |
| Key exchange security | Is the key exchange secure ? | Pass : out-of-band key exchange can be considered as safe, if done properly |
| Unencrypted payload | Receiving an unencrypted payload must not trigger the output relay | Pass : unencrypted frames are ignored when encryption is enabled. |
| Replay attack mitigation | Replaying a frame must not trigger the output relay | |


## Final thoughts
