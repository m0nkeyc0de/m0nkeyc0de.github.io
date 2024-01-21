---
title: "Wiping storage devices"
date: 2024-01-21T11:04:16+01:00
slug: storage_wipe
type: posts
draft: false
categories:
  - Security
tags:
  - storage
  - linux
---
Before giving or throwing devices away, you should better wipe storage drives properly. Otherwise your data may be recovered and who knows what happens next...

## Expeditive solution
If you can't wipe your drive properly, have some fun and **destroy** it physically.

## Vendor provided solutions
Some device vendors (HP for example) provide a built-in **Secure Erase** solution in the BIOS/UEFI.

## DIY solutions

### Hard Disk Drives
All sectors of the magnetic drive must overwritten several times to ensure data is properly wiped out. The Linux **shred** command does the job well by default (3 random iterations).

```
shred /dev/sdX --force --verbose
```

### Flash drives and memory cards
USB flash drives and memory cards only need one pass with zeroes to be wiped-out. The Linux command **shred** can do it well.

```
shred /dev/sdX --force --verbose --iterations=0 --zeroes
```

### Solid State Drives
Wiping SSDs is a little bit more tricky than other types of drives. They have a TRIM function which writes new data to the less used free sectors. Therefore you can't just use shred as it may not overwrite the entire drive (and also reducing its lifespan). ATA commands must be sent using **hdparm** as explained in this [excellent article](https://www.thomas-krenn.com/en/wiki/Perform_a_SSD_Secure_Erase).

```
SSD=/dev/sdX
USERMASTER=u
PASS=Eins

# 1a. NOT frozen
# Note: SSD is "frozen" if the SATA port is not hot swappable
# Check that "not frozen" is present in Security section
hdparm -I $SSD

# 1b. If it's frozen, try to supsend the machine (or make it sleep)
systemctl suspend
hdparm -I $SSD

# 2. Set the user password
# "Security level high" is added to the Security section output
hdparm --user-master $USERMASTER --security-set-pass $PASS $SSD

# 3. Secure Erase
time hdparm --user-master $USERMASTER --security-erase $PASS $SSD

# 4. Checking
# "Security level high" should be away from Security section
hdparm -I $SSD
```

## Android Phones
If the data is automatically encrypted (PIN code necessary to start applications), a factory reset will clean the encryption keys and make your data inaccessible.

If you have an old device without data encryption, a factory reset may not be sufficient. You may try to **dd** zeroes on **/data** using **adb** (my best guess so far).
