# Arch Linux Guide

Tested and up to date on the 2021/11/01 ISO

This is a comprehensive and (soon to be) exhaustive Arch Linux step-by-step installation and configuration guide as well as a basic user guide. It is meant for everyone willing to give arch a try but not being able to grasp the multitude of options and concepts described on archwiki. It aims to provide sane defaults and cover system optimization and maintenance, all in a single, easy to follow file, including appropriate commands.

In the future, it will be expanded into an installation script and a post-installation setup script to automate all the mentioned steps. Understanding of those steps is nonetheless crucial.

The configuration covered in the guide includes but is not limited to:
* GUID/GPT partition scheme with separate home and swap partitions
* ext4 file system
* systemd-boot bootloader
* mirrors refresh automation
* GPU drivers for all common GPU configurations
* Vulkan graphics
* hardware video acceleration
* an AUR helper (paru or yay)
* Gnome desktop configuration

Planned additions:
* KDE desktop environments configurations
* openCL and CUDA configurations
* creating virtual partitions with lvm
* OPTIMUS hybrid graphics
* performance optimizations
* optimizing battery life on laptops
* silent and fast boot
* zsh customization
* pipewire audio
* drivers for old Radeon GPUs
* configuration of a WM-based desktop

Conventions used in this guide are as follows:
* Commands marked with: $
* Comments marked with: #
* Names or values to input: "..." 
* All optional names/values: (...)

## Stage 0: Create a bootable USB and change BIOS settings

## Stage 1: Initial ISO setup

## Stage 2: Partition, format, mount, install

## Stage 3: Essential system configuration

## Stage 4: Installing GPU drivers

## Stage 5: Booting into the system, additional configuration

## Stage 6: Installing a Desktop Environment, Display Manager, enabling services

## Stage 7: Performance optimizations

## Stage 8: Power optimizations (for laptops)

## Stage 9: Essentials of system management & Troubleshooting
