# Arch Linux Guide

#### Tested and up to date on the 2021/11/01 ISO

This is a comprehensive and exhaustive Arch Linux step-by-step installation and configuration guide as well as a basic user guide. It is meant for everyone willing to give arch a try but not being able to grasp the multitude of options and concepts described on archwiki. It aims to provide sane defaults and cover system optimization and maintenance, all in a single, easy to follow file, including appropriate commands.

#### Conventions used in this guide are as follows:

- Commands marked with: $
- Comments marked with: #
- Names or values to input: "..." 
- Optional names/values: (...)
- Used to separate commands/inputs: ;
- Optional steps marked with cursive
- sdY used to refer to a usb drive (eg. sdc)
- sdX used to refer to hard drive (eg. sda, nvme0)

## Create a bootable USB 
      
#### On Windows: 

Download the latest ISO from https://archlinux.org/download/

Use Rufus, format to FAT32, burn the arch ISO (www.rufus.ie)

#### On Linux or Mac:

Download the latest ISO from https://archlinux.org/download/

*(optional)* Verify the image:  
by downloading the PGP signature, placing it in the ISO directory and running:  
```
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-"ISO version"-x86_64.iso.sig
```

List all blocks/devices to determine the usb drive directory:  
easiest by looking at size, the usb drive might be called sdb, sdc etc. 
I will refer to the u it as sdY
```
$ lsblk
```

Copy the ISO to the USB drive directory sdY:  
replace sdY by whatever your drive name is, as displayed by lsblk
```
$ sudo cp "path to iso"/archlinux-"ISO version"-x86_64.iso /dev/sdY
```

## Change BIOS settings and boot into Arch live environment:

Insert the USB

Enter BIOS on startup  
by mashing F2, F7, F11 etc. depending on the motherboard

Disable secure boot  
clear/delete secure boot keys if necessary, it's reversible

*(optional)* Disable fast boot  
only do it if you encounter issues, reenable it after install

Change the boot priority of the USB drive to be the highest

Save changes and exit  
should reboot into arch

## Initial ISO setup

#### Verify (uefi) boot mode:
```
$ ls /sys/firmware/efi/efivars
```

#### Check network interface:
```
$ ip link
```

#### Connect to wifi: 
```
$ iwctl
$ device list                                   # find wireless interface/device
$ station "Device" scan                         # eg. station wlan0 scan
$ station "Device" get-networks
$ station "Device" connect "Network"
$ "network's wifi password"
# wait a couple seconds and quit iwctl using ctrl+d
```

#### Verify connection:
```		
$ ping -c 5 archlinux.org
```

#### In case of wifi connection issues:
```
$ rfkill unblock all
```

#### Update system clock:
```
$ timedatectl set-ntp true
$ timedatectl status
```

## Partition, format, mount, install

#### Partition disks:  
separate home and swap partitions are optional, replace sdX with your drive name eg. sda or nvme0
```
$ lsblk                                         # identify your hard drive (eg. sda) by its size
$ hdparm -i /dev/sdX                            # optionally inspect the drive to make sure
$ sgdisk -Z /dev/sdX                            # CAUTION! this zaps/wipes the drive
$ cgdisk /dev/sdX
      New; default; "..."MiB; ef00; boot        # 1024MiB is recommended
      New; default; "..."GiB; 8200; swap        # for hibernation 2xRam
      New; default; "..."GiB; default, root     # at least 20, safest 40
      New; default; default; default; home
      Write; Quit;
```

#### Format partitions and setup Swap:  
instead of sdX1-4, use appropriate partition names, like sda1-4, nvme0p1-p4 or nvme0n1p1-p4
```
$ lsblk                                         # identify your partition names
$ mkfs.fat -F32 /dev/sdX1                       # formats boot as fat32
$ mkswap /dev/sdX2                              # makes swap on swap partition
$ swapon /dev/sdX2                              # enables swap
$ mkfs.ext4 /dev/sdX3                           # formats root to ext4
$ mkfs.ext4 /dev/sdX4                           # formats home to ext4
```

#### Mounting folders for installation:  
once again, use your own partition names
```
$ mount /dev/sda3 /mnt                          # mount root
$ mkdir /mnt/boot                               # create boot directory
$ mkdir /mnt/home                               # create home directory
$ mount /dev/sda1 /mnt/boot                     # mount boot partition at the boot directory
$ mount /dev/sda4 /mnt/home                     # mount home partition at the home directory
```

#### Configuring mirrors on the installation device:  
*(optional but recommended for faster installation)*
```
$ nano /etc/xdg/reflector/reflector.conf        # configure reflector - a mirror refreshing tool
      --save /etc/pacman.d/mirrorlist
      --country "Country1","Country2..."
      --protocol https
      --sort rate
      --latest 5
$ systemctl start reflector.service             # start a service that refreshes mirrors
```

#### Installation of system and base utilities:  
*(optionally you can install lts or custom kernel and headers or a different terminal text editor)*
```
$ pacstrap -i /mnt base base-devel linux linux-headers linux-firmware nano git
```

#### Making an fstab file (file system table) which mounts partitions into filesystem on boot:
```
$ genfstab -U /mnt >> /mnt/etc/fstab
$ cat /mnt/etc/fstab
```

## Essential system configuration

#### Change root to load into the new system:
```
$ arch-chroot /mnt
```

#### Set time zone:
```
$ hwclock --systohc (--utc)                     # use hardware clock
$ timedatectl list-timezones
$ timedatectl set-timezone "Region"/"City"      # this links /usr/share/zoneinfo/"Region"/"City" to /etc/localtime ?
$ timedatectl set-ntp true
$ timedatectl status
# systemctl enable systemd-timesyncd		
      # synchs clock on boot, should be enabled by set-ntp true
```

#### Mirror refresh automation:
```
$ pacman -Syu reflector
$ nano /etc/xdg/reflector/reflector.conf
      --save /etc/pacman.d/mirrorlist
      --country "Country1","Country2..."
      --protocol https
      --sort rate
      --latest 5
$ systemctl enable reflector.timer              # refreshes once a week
```

#### Hostname configuration:
```
$ hostnamectl set-hostname "Hostname"           # eg. ArchLinux
$ nano /etc/hosts
      127.0.0.1   localhost
      ::1         localhost
      127.0.1.1   "Hostname"
```

#### Set system locale:
```
$ nano /etc/locale.gen
      # uncomment selected locale
      # eg. en_US.UTF-8 for default American English
      # eg. en_IE.UTF-8 for English with EU formats and metric units
$ locale-gen
$ echo LANG="Selected Locale" > /etc/locale.conf
$ export LANG="Selected Locale"                 # sets locale for all processes for current shell
```

#### Network configuration:
```
$ pacman -S networkmanager
$ systemctl enable NetworkManager.service
# after booting into the system, use $ nmtui or $ nmcli to set up wifi
```

#### Enable multilib - 32 bit app support:  
*(optional, recommended)*
```
$ nano /etc/pacman.conf
      # Uncomment 2 lines: [multilib] and Include =...
$ pacman -Syu
```

#### For LVM, system encryption or RAID:  
*(skip this, only here as reference)*
```
$ nano /etc/mkinitspcio.conf                    # modify initramfs config
$ mkinitcpio -P                                 # recreate image using config
$ mkinitcpio -p linux                           # regenerate using linux defaults
# image must be specified in boot loader config file
```

#### Set root password:
```
$ passwd
```

#### Create a user with administrative privileges, create user password:
```
$ useradd -m -g users -G wheel "user"
$ passwd "user"
```

#### Associate wheel group with sudo:
```
$ EDITOR=nano visudo                            # safely edit /etc/sudoers file
      # Uncomment %wheel ALL=(ALL) ALL
      # Optionally, if you want to require roots instead of users password, add:
            Defaults rootpw
```

#### Install and configure boot loader:  
*(systemd-boot used here, alternatively grub can be used)*
```
$ bootctl install
$ pacman -S "Cpu"-ucode                         # intel-ucode or amd-ucode, depending on your CPU
$ nano /boot/loader/entries/"Entry".conf        # recommended name arch.conf to avoid confusion
      title "Title"                             # eg. title ArchLinux
      linux /vmlinuz-linux
      initrd /"Cpu"-ucode.img                   # intel-ucode.img / amd-ucode.img
      initrd /initramfs-linux.img
$ echo "options root=UUID=$(blkid -s UUID -o value /dev/sdX3) rw" >> /boot/loader/entries/"Entry".conf
      # this copies the UUID of your root partition and adds it to the loader
```

#### Create a pacman hook to automatically update the bootloader:  
*(optional but recommended)*
```
$ nano /etc/pacman.d/hooks/100-systemd-boot.hook
      [Trigger]
      Type = Package
      Operation = Upgrade
      Target = systemd

      [Action]
      Description = Updating systemd-boot
      When = PostTransaction
      Exec = /usr/bin/bootctl update
```

## Installing GPU drivers

#### Identify the GPU:
```
$ lspci -k | grep -A 2 -E "(VGA|3D)"
```

***

#### - Intel integrated graphics:
```
$ pacman -S xf86-video-intel                                # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
            vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            vulkan-intel lib32-vulkan-intel                 # Vulkan drivers
```

#### - AMD modern graphics:  
for GCN3/Radeon Rx 300 and newer cards
```
$ pacman -S xf86-video-amdgpu                               # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
            vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            vulkan-radeon lib32-vulkan-radeon               # Vulkan drivers
```

#### - AMD legacy graphics:  
for GCN2/Radeon Rx 200 and older cards  
*(more details on https://wiki.archlinux.org/title/ATI)*
```
$ pacman -S xf86-video-ati                                  # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
```

#### - AMD Radeon HD 7000 and RX200 generations:  
*https://wiki.archlinux.org/title/AMDGPU#Experimental*  
*(section needs expanding)*

#### - Nvidia:  

Install only ONE of the following 3 packages, depending on your kernel:***
```
$ pacman -S nvidia                                          # for standard linux kernel
            nvidia-lts                                      # for the linux-lts kernel
            nvidia-dkms                                     # other kernels eg. linux-zen
```

Install all of the following packages:  
*(providing OpenGL, Vulkan, OpenCL support and configuration tools)*
```
$ pacman -S vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            nvidia-utils lib32-nvidia-utils                 # OpenGL, Vulkan
            opencl-nvidia                                   # OpenCL support
            xorg-server-devel                               # Provides nvidia-xconfig
            nvidia-settings                                 # Nvidia settings menu

$ nvidia-xconfig                                            # optional auto-configuration for xorg
$ nvidia-settings                                           # settings GUI, use after installation
```

Enable DRM (direct rendering manager):  
by adding a drm kernel mode setting in your boot loader entry
```
$ nano /boot/loader/entries/"Entry".conf
      # set kernel parameter: nvidia-drm.modeset=1 after options root=(...) rw
      # eg. options root="UUID=361cn91740c73-2x9mm871c2dsa" rw nvidia-drm.modeset=1
```

Ensure early loading of the drivers:  
*(optional, recommended?)*
```
$ nano /etc/mkinitcpio.conf
      MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

Automatically update initramfs after updating kernel or driver:  
*(required when using early loading or custom kernels)*
```
$ nano /etc/pacman.d/hooks/nvidia.hook
      [Trigger]
      Operation=Install
      Operation=Upgrade
      Operation=Remove
      Type=Package
      Target=nvidia                             # or nvidia-lts or nvidia-dkms
      Target=linux                              # or linux-lts or other kernel

      [Action]
      Description=Update Nvidia module in initcpio
      Depends=mkinitcpio
      When=PostTransaction
      NeedsTargets
      Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

#### - PRIME:  
hybrid graphics on laptops - Intel/AMD+Nvidia/AMD  
check arch wiki, if you have a laptop with a modern dedicated gpu  
*(this section needs expanding)*

***

#### Vulkan verification:  
*(optional, recommended)*
```
$ ls /usr/share/vulkan/icd.d/                   # shows installed vulkan implementations
$ pacman -S vulkan-tools
$ vulkaninfo                                    # should show info about your GPU
```

## Hardware video acceleration  

#### - Intel integrated graphics:  
pick one or none depending on your CPU
```
$ pacman -S intel-media-driver                  # 5th gen and newer intel CPUs
          libva-intel-driver                    # 3rd and 4th gen intel CPUs
```

#### - AMD:  
install both unless running an ancient GPU
```
$ pacman -S libva-mesa-driver                   # R600 (Radeon HD 2000) and newer
          mesa-vdpau                            # R300 (Radeon 9000) and newer
```

#### - Nvidia:  
hardware accelerated video decoding is provided by nvidia-utils  
for encoding with NVENC, create this rule
```
$ nano /etc/udev/rules.d/70-nvidia.rules        
      ACTION=="add", DEVPATH=="/bus/pci/drivers/nvidia", 
      RUN+="/usr/bin/nvidia-modprobe -c0 -u"
```

#### Verify hardware video acceleration:  
*(Optional, untested)*
```
$ pacman -S libva-utils	vdpauinfo
$ vainfo                                        # verify VA-API settings
$ vdpauinfo                                     # verify VDPAU settings
$ pacman -S intel-gpu-tools                     # for intel GPUs
$ intel-gpu-top                                 # play video and monitor gpu usage
$ pacman -S radeontop                           # for AMD GPUs
$ radeontop                                     # play video and monitor gpu usage
```

## Booting into the system, additional configuration

## Installing a Desktop Environment, Display Manager, enabling services

## Performance optimizations

## Power optimizations (for laptops)

## Essentials of system management & Troubleshooting
