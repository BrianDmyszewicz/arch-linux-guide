# Arch Linux Guide

Tested and up to date on the 2021/11/01 ISO

This is a comprehensive and exhaustive Arch Linux step-by-step installation and configuration guide as well as a basic user guide. It is meant for everyone willing to give arch a try but not being able to grasp the multitude of options and concepts described on archwiki. It aims to provide sane defaults and cover system optimization and maintenance, all in a single, easy to follow file, including appropriate commands.

Conventions used in this guide are as follows:
- Commands marked with: $
- Comments marked with: #
- Names or values to input: "..." 
- Optional names/values: (...)
- Used to separate commands/inputs: ;
- Optional steps marked with cursive
- sdY used to refer to a usb drive (eg. sdc)
- sdX used to refer to hard drive (eg. sda, nvme0)

## Step 0: Create a bootable USB 

#### On Linux or Mac:

Download the latest ISO from https://archlinux.org/download/

To verify the image, download the PGP signature, place it in the ISO directory and run: (optional)
```
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-"ISO version"-x86_64.iso.sig
```

List all devices to determine the usb drive directory (eg. sdb, sdc) by looking at size:
```
$ lsblk
```

Copy the ISO to the USB drive directory sdY (replace sdY by whatever your drive name is)
```
$ sudo cp "path to iso"/archlinux-"ISO version"-x86_64.iso /dev/sdY
```
      
#### On Windows: 

Download the latest ISO from https://archlinux.org/download/

Use Rufus, format to FAT32, burn the arch ISO (www.rufus.ie)

#### Change BIOS settings and boot into Arch live environment:

Insert the USB \
Enter BIOS on startup (by mashing F2, F7, F11 etc. depending on the motherboard) \
Disable secure boot (clear/delete secure boot keys if necessary, it's reversible) \
Disable fast boot (might not be required, remember to reenable it after install) \
Change the boot priority of the USB drive to be the highest \
Save changes and exit (should reboot into arch)

## Step 1: Initial ISO setup

Verify (uefi) boot mode:
```
$ ls /sys/firmware/efi/efivars
```

Check network interface:
```
$ ip link
```

Connect to wifi: 
```
$ iwctl
$ device list                                   # find wireless interface/device
$ station "Device" scan                         # eg. station wlan0 scan
$ station "Device" get-networks
$ station "Device" connect "Network"
$ "network's wifi password"
# wait a couple seconds and quit iwctl using ctrl+d
```

Verify connection:
```		
$ ping -c 5 archlinux.org
```

In case of wifi connection issues:
```
$ rfkill unblock all
```

Update system clock:
```
$ timedatectl set-ntp true
$ timedatectl status
```

## Step 2: Partition, format, mount, install

Partition disks: \
_(separate home and swap partitions are optional, replace sdX with your drive name eg. sda or nvme0)_
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

Format partitions and setup Swap: \
_(instead of sdX1-4, use appropriate partition names, like sda1-4, nvme0p1-p4 or nvme0n1p1-p4)_
```
$ lsblk                                         # identify your partition names
$ mkfs.fat -F32 /dev/sdX1                       # formats boot as fat32
$ mkswap /dev/sdX2                              # makes swap on swap partition
$ swapon /dev/sdX2                              # enables swap
$ mkfs.ext4 /dev/sdX3                           # formats root to ext4
$ mkfs.ext4 /dev/sdX4                           # formats home to ext4
```

Mounting folders for installation: \
_(once again, use your own partition names)_
```
$ mount /dev/sda3 /mnt                          # mount root
$ mkdir /mnt/boot                               # create boot directory
$ mkdir /mnt/home                               # create home directory
$ mount /dev/sda1 /mnt/boot                     # mount boot partition at the boot directory
$ mount /dev/sda4 /mnt/home                     # mount home partition at the home directory
```

Configuring mirrors on the installation device: \
_(optional but recommended for faster installation)_
```
$ nano /etc/xdg/reflector/reflector.conf        # configure reflector - a mirror refreshing tool
      --save /etc/pacman.d/mirrorlist
      --country "Country1","Country2..."
      --protocol https
      --sort rate
      --latest 5
$ systemctl start reflector.service             # start a service that refreshes mirrors
```

Installation of system and base utilities: \
_(optionally you can install lts or custom kernel and headers or a different terminal text editor)_
```
$ pacstrap -i /mnt base base-devel linux linux-headers linux-firmware nano git
```

Making an fstab file (file system table) which mounts partitions into filesystem on boot:
```
$ genfstab -U /mnt >> /mnt/etc/fstab
$ cat /mnt/etc/fstab
```

## Step 3: Essential system configuration

Change root to load into the new system:
```
$ arch-chroot /mnt
```

Set time zone:
```
$ hwclock --systohc (--utc)                     # use hardware clock
$ timedatectl list-timezones
$ timedatectl set-timezone "Region"/"City"      # this links /usr/share/zoneinfo/"Region"/"City" to /etc/localtime ?
$ timedatectl set-ntp true
$ timedatectl status
# systemctl enable systemd-timesyncd		
      # synchs clock on boot, should be enabled by set-ntp true
```

Mirror refresh automation:
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

Hostname configuration:
```
$ hostnamectl set-hostname "Hostname"           # eg. ArchLinux
$ nano /etc/hosts
      127.0.0.1   localhost
      ::1         localhost
      127.0.1.1   "Hostname"
```

Set system locale:
```
$ nano /etc/locale.gen
      # uncomment selected locale
      # eg. en_US.UTF-8 for default American English
      # eg. en_IE.UTF-8 for English with EU formats and metric units
$ locale-gen
$ echo LANG="Selected Locale" > /etc/locale.conf
$ export LANG="Selected Locale"                 # sets locale for all processes for current shell
```

Network configuration:
```
$ pacman -S networkmanager
$ systemctl enable NetworkManager.service
# after booting into the system, use $ nmtui or $ nmcli to set up wifi
```

Enable multilib - 32 bit app support: \
_(optional, recommended)_
```
$ nano /etc/pacman.conf
      # Uncomment 2 lines: [multilib] and Include =...
$ pacman -Syu
```

For LVM, system encryption or RAID: \
_(skip this, only here as reference)_
```
$ nano /etc/mkinitspcio.conf                    # modify initramfs config
$ mkinitcpio -P                                 # recreate image using config
$ mkinitcpio -p linux                           # regenerate using linux defaults
# image must be specified in boot loader config file
```

Set root password:
```
$ passwd
```

Create a user with administrative privileges, create user password:
```
$ useradd -m -g users -G wheel "user"
$ passwd "user"
```

Associate wheel group with sudo:
```
$ EDITOR=nano visudo                            # safely edit /etc/sudoers file
      # Uncomment %wheel ALL=(ALL) ALL
      # Optionally, if you want to require roots instead of users password, add:
            Defaults rootpw
```

Install and configure boot loader: \
_(systemd-boot used here, alternatively grub can be used)_
```
$ bootctl install
$ pacman -S "Cpu"-ucode                         # intel-ucode or amd-ucode, depending on your CPU
$ nano /boot/loader/entries/"Entry".conf        # recommended name arch.conf to avoid confusion
      title "Title"                             # eg. title ArchLinux
      linux /vmlinuz-linux
      initrd /"Cpu"-ucode.img                   # intel-ucode.img / amd-ucode.img
      initrd /initramfs-linux.img
$ echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/"Entry".conf
      # this copies the UUID of your root partition and adds it to the loader
      # it's optional but recommended and more reliable than label
```

Create a pacman hook to automatically update the bootloader: \
_(optional but recommended)_
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

## Step 4: Installing GPU drivers

## Step 5: Booting into the system, additional configuration

## Step 6: Installing a Desktop Environment, Display Manager, enabling services

## Step 7: Performance optimizations

## Step 8: Power optimizations (for laptops)

## Step 9: Essentials of system management & Troubleshooting
