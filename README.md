# Arch Linux Guide

Tested and up to date on the 2021/11/01 ISO

This is a comprehensive and exhaustive Arch Linux step-by-step installation and configuration guide as well as a basic user guide. It is meant for everyone willing to give arch a try but not being able to grasp the multitude of options and concepts described on archwiki. It aims to provide sane defaults and cover system optimization and maintenance, all in a single, easy to follow file, including appropriate commands.

Conventions used in this guide are as follows:
- Commands marked with: $
- Comments marked with: #
- Names or values to input: "..." 
- Optional names/values: (...)
- sdY used to refer to a usb drive (eg. sdc)
- sdX used to refer to hard drive (eg. sda)

## Stage 0: Create a bootable USB and change BIOS settings

### Create a bootable USB on Linux:

###### Download the latest ISO from https://archlinux.org/download/

###### Verify the image: (optional) \
- Download the PGP signature, place it in the ISO directory and run:
`$ gpg --keyserver-options auto-key-retrieve --verify archlinux-"ISO version"-x86_64.iso.sig`

List all devices to determine the usb drive directory (eg. sdb, sdc) by looking at size:
  $ lsblk

Copy the ISO to the USB drive directory sdY (NOT to a specific partition eg. sdY1)
  $ sudo cp "path to iso"/archlinux-"ISO version"-x86_64.iso /dev/sdY
      
### Create a bootabe USB on Windows: 

Download the latest ISO from https://archlinux.org/download/

Use Rufus, format to FAT32, burn the arch ISO (www.rufus.ie)

### Change BIOS settings and boot into Arch live environment

- Insert the USB,
- Enter BIOS on startup (by mashing F2, F7, F11 etc. depending on the motherboard)
- Disable secure boot (clear/delete secure boot keys if necessary, it's reversible)
- Disable fast boot (might not be required, remember to reenable it after install)
- Change the boot priority of the USB drive to be the highest
- Save changes and exit (should reboot into arch)

## Stage 1: Initial ISO setup

## Stage 2: Partition, format, mount, install

## Stage 3: Essential system configuration

## Stage 4: Installing GPU drivers

## Stage 5: Booting into the system, additional configuration

## Stage 6: Installing a Desktop Environment, Display Manager, enabling services

## Stage 7: Performance optimizations

## Stage 8: Power optimizations (for laptops)

## Stage 9: Essentials of system management & Troubleshooting
