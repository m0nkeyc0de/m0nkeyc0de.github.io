---
title: "Firmwares reverse engineering"
date: 2026-11-10
lastmod: 2026-11-10
slug: firmware-reverse-engineering
type: posts
draft: false
categories:
  - Security
tags:
  - mithril
  - moria
  - hashcat
---
What is really inside your devices firmwares ?

## Where to find firmware files ?
There are mainly two ways to get a firmware file:
* Download it from your vendor website (quick and easy way)
* Off-chip extraction (very involved way, far beyond the scope of this article)

## Files extraction
With [moria](https://github.com/nmatt0/moria), which is basically a `binwalk` on steroids, you should be able to extract all the files contained in it.

```bash
moria -e YOUR_FIRMWARE_FILE.bin
```

## Static analysis
Once the firmware extracted, you can run a static analysis on it with [mithril](https://github.com/nmatt0/mithril).

```bash
# Fetch CVE DB
mithril --fetch-db

# Let your extracted firmware be analyzed
mithril YOUR_FIRMWARE_FILE.bin.extracted
```

## What next ?
Once the static analysis done, the real fun begins !

### Check for weak passwords with hashcat
The static analysis will certainly list `shadow` files, which may contain hashed passwords. Are they any good ?

```bash
# Get the rockyou.txt password list
wget https://github.com/RykerWilder/rockyou.txt/raw/refs/heads/main/rockyou.txt.zip
unzip rockyou.txt.zip

# List hashes in the shadow file
hashcat YOUR_FIRMWARE_FILE.extracted/path/to/etc/shadow

# You'll get a list of password modes. MD5 is not great...
#      # | Name                                                       | Category
#  ======+============================================================+======================================
#    500 | md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)                  | Operating System
#   1500 | descrypt, DES (Unix), Traditional DES                      | Operating System

# Test MD5-crypt hashes for known passwords from rockyou.txt
hashcat -m 500 --force YOUR_FIRMWARE_FILE.extracted/path/to/etc/shadow rockyou.txt
```
