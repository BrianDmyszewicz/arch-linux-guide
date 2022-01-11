# Arch Linux Guide

#### Tested and up to date on the 2021/11/01 ISO

This is a comprehensive and exhaustive Arch Linux step-by-step installation and configuration guide as well as a basic user guide. It is meant for everyone willing to give arch a try but not being able to grasp the multitude of options and concepts described on archwiki. It aims to provide sane defaults and cover system optimization and maintenance, all in a single, easy to follow file, including appropriate commands. It is highly recommended that you're familiar with general unix system structure and commands before following this guide, you can learn about those from my other guides.

#### Conventions used in this guide are as follows:

Commands can be split into multiple lines, they're marked with: $  
Comments inbetween or after commands are marked with: #  
Used to separate commands/inputs on a single line: ;  
_**Names**_ or _**values**_ for you to input marked with _**bold italic**_  
Optional names or values: (...)  
sdY used to refer to a usb drive (eg. sdc)  
sdX used to refer to hard drive (eg. sda, nvme0)  

## Create a bootable USB on Linux:  

A bootable USB is a USB drive which carries an operating system and can be booted (meaning loaded/started) by a computer, much like a regular hard drive. In order to create a bootable USB, we must obtain the operating system and "install it" on said USB drive. This can be done using any software or through command line, as presented below. 

#### Download the latest ISO  
you can get it from https://archlinux.org/download/

#### Verify the image *(optional)*  
download the PGP signature, place it in the ISO directory and run:  
<pre>
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-<b><i>ISO version</i></b>-x86_64.iso.sig
</pre>

#### Determine the usb drive directory  
by listing all block devices and looking at their size, the usb drive might be called sdb, sdc etc. I will refer to it as sdY
<pre>
$ lsblk
</pre>

#### Copy the ISO to the USB drive directory sdY:  
replace sdY by whatever your drive name is, as displayed by lsblk
<pre>
$ sudo cp "path to iso"/archlinux-"ISO version"-x86_64.iso /dev/sdY
</pre>

## Create a bootable USB on Windows:

#### Download the latest ISO  
You can get it from https://archlinux.org/download/

#### Burn the Arch ISO onto the USB drive  
You can use a tool called Rufus (www.rufus.ie) or any other tool you want, make sure to format the USB drive to FAT32.

## Change BIOS settings and boot into Arch live environment:  

#### Insert the USB  

#### Enter BIOS on startup:  
Do this by mashing a specific key on startup, which key to press depends on the motherboard, most commonly F1, F2, F7 Esc or Del.

#### Use UEFI boot mode:  
There are 2 methods of booting a computer - the legacy mode using BIOS firmware or UEFI mode. UEFI is the default on all modern motherboards but in case you see this setting, make sure to select it, the bootloader used in this guide requires it.

#### Disable secure boot:  
Secure boot requires all firmware to be digitally signed to reduce chance of malware. Since your firmware will come from the kernel and official repositories, this is not required and can in fact cause many issues. To disable secure boot on some motherboards you might have to clear/delete secure boot keys. It's easily reversible so don't worry.

#### Change the boot priority:
Your BIOS will normally boot into your hard drive by default. You must change its boot priority to make sure that the USB is booted instead. Set the boot priority of your Arch USB drive to be the highest.  

#### Save changes and exit:  
The computer should reboot into Arch Linux, if it doesn't, make sure you performed the above steps correctly.  

#### In case of any issues: *(optional)*  
Disable fast boot - make sure to reenable it after the installation  


## Initial ISO setup

The Arch Linux ISO is a functional Arch system in form of a Live Environment, installed on the USB drive. It comes with the tools required to install Arch on your machine but also tools that allow you to access any Linux machine and troubleshoot it in case anything ever goes wrong, this will be covered in the recovery section. The following steps will make sure the Live Environment is running properly and has access to the internet.  

#### Verify UEFI boot mode:
This command should return a lot of EFI variables if you're using the UEFI boot mode. If it doesn't, return to BIOS settings.  
<pre>
$ ls /sys/firmware/efi/efivars
</pre>

#### Check network interface:
Your wired connection will be called something like eth0, enp3s0, enp3s10 etc. If you don't have a wired connection, connect to wifi.
<pre>
$ ip link
</pre>

#### Connect to wifi: *(optional)*  
Iwctl is a tool built into the Arch ISO that allows you to connect to a wireless network. This tool isn't included by default on a clean Arch Linux installation so you'll only really have to use it now.
<pre>
$ iwctl
$ device list                                   # find wireless interface/device eg. wlan0
$ station "Device" scan                         # eg. station wlan0 scan
$ station "Device" get-networks                 # should display all network names
$ station "Device" connect "Network"
$ "network's wifi password"
# wait a couple seconds and quit iwctl using ctrl+d
</pre>

If you encounter any issues, you can use rfkill to enable all wireless interfaces.  
<pre>
$ rfkill unblock all
</pre>

#### Verify connection:
If a connection is established, you'll see 5 packets transmitted and received with time higher than 0.  
<pre>
$ ping -c 5 archlinux.org
</pre>

#### Update system clock:
This enables systemd-timesyncd, a systemd component which synchronizes system time to a remote server, using a simple NTP protocol. The status command should show that the NTP service is active and system clock is synchronized.  
<pre>
$ timedatectl set-ntp true
$ timedatectl status
</pre>

## Partition, format, mount, install
Right now you're still using the Arch Live Environment on the USB drive. To install Arch on your hard drive, you must first partition it. The partitions must then be formatted depending on their function. Lastly, the formatted partitions created on the hard drive must be mounted on the currently used live usb drive, in order for the installation to be possible.

#### Partition disks:  
Arch only requires a boot partition, used to initially boot into the system, and a root partition, where the entire system lives. It is highly recommended, however, to keep your user files (home directory) on a separate partition. A swap partition is also recommended, the system can use it as "emergency RAM" to prevent freezes and crashes. To create all 4 using recommended names and sizes, follow the steps below.  

Replace sdX with your drive name eg. sda, nvme0, nvme0n1 etc, as identified by the output of lsblk.
<pre>
$ lsblk                                         # identify your hard drive by looking at its size
$ hdparm -i /dev/sdX                            # inspect the drive to make sure it's the right one
$ sgdisk -Z /dev/sdX                            # caution! this will completely wipe the drive
$ cgdisk /dev/sdX
      New; default; "..."MiB; ef00; boot        # 1024MiB is recommended
      New; default; "..."GiB; 8200; swap        # twice your Ram if you want hibernation
      New; default; "..."GiB; default, root     # at least 20 but i recommend 40
      New; default; default; default; home
      Write; Quit;
</pre>

#### Format partitions and setup Swap:  
It's recommended and simplest to format the root partition to ext4. The same is recommended for the home partition. Boot partition is an exception, it's a special EFI system partition and must use the FAT32 format. The swap partition is not formatted, instead swap is "made" and enabled.

Instead of sdX1-4, use appropriate partition names, like sda1-4, nvme0p1-p4 or nvme0n1p1-p4
<pre>
$ lsblk                                         # identify your partition names
$ mkfs.fat -F32 /dev/sdX1                       # formats boot as fat32
$ mkswap /dev/sdX2                              # makes swap on swap partition
$ swapon /dev/sdX2                              # enables swap
$ mkfs.ext4 /dev/sdX3                           # formats root to ext4
$ mkfs.ext4 /dev/sdX4                           # formats home to ext4
</pre>

#### Mounting partitions:  
First mount the root directory at /mnt, a mount point directory of the live arch environment. Then make home and boot directories under the /mnt directory in order to then mount their corresponding partitions there. If you understand unix directory structure, this should make sense, as home and boot are subdirectories of root. Once the hard drive partitions are properly mounted on the live usb drive, the installation can be performed.  

Once again, use your own partition names instead of sdX1-4
<pre>
$ mount /dev/sdX3 /mnt                          # mount root
$ mkdir /mnt/boot                               # create boot directory
$ mkdir /mnt/home                               # create home directory
$ mount /dev/sdX1 /mnt/boot                     # mount boot partition at the boot directory
$ mount /dev/sdX4 /mnt/home                     # mount home partition at the home directory
</pre>

#### Configuring mirrors on the installation device:
A mirror is a server that provides an exact copy of repositories that you'll use to download and install the system, software and any updates. Mirrors might at times experience downtime or be out of date and your local mirrors will provide a much faster download speed so it's worth configuring and updating them.  

Reflector is a tool built into the Arch ISO which can be used to refresh your mirrors, you can configure it as follows:  
<pre>
$ nano /etc/xdg/reflector/reflector.conf        # use nano to edit the reflector configuration file
      --save /etc/pacman.d/mirrorlist           # tells reflector where to save the mirror list
      --country "Country1","Country2..."        # this is optional, sorting latest by rate is often sufficient
      --protocol https                          # also optional but technically more secure
      --sort rate                               # sorts mirrors by download speed
      --latest 5                                # only saves 5 most up to date mirrors
</pre>

Next you can use systemd to start the reflector service, which refreshes your mirrors. Services are just programs/applications that usually run in the background. You can start services through systemd, using the systemctl command. A started service will perform once. 
<pre>
$ systemctl start reflector.service             # starts a systemd service that refreshes mirrors
</pre>

#### Installation of system and base utilities:  
Pacstrap is a tool built into the arch ISO that allows for an easy Arch installation. It can also install some additional packages that you will need to set up the system. I recommend installing nano as a terminal text editor, git in order to install software from the Arch User Repository and reflector, in order to automate refreshing mirrors. (reflector and nano were present on the Arch ISO but aren't installed by default on a clean Arch system)  
<pre>
$ pacstrap -i /mnt base base-devel linux linux-headers linux-firmware nano git reflector
</pre>

#### Making a file system table:
A file system table or an fstab file is stored in /etc/fstab. It contains rules that tell systemd where to mount partitions or remote file systems on boot. For instance, it will make sure that your home partition is always mounted in the home directory under root. Fstab can be generated using genfstab and its output can be sent to the desired location. Remember that despite already installing Arch on the hard drive, we're still using the Live Environment, therefore we must save fstab at /mnt/etc/fstab, as we use the temporary mount point /mnt as our root directory.
<pre>
$ genfstab -U /mnt >> /mnt/etc/fstab            # -U option defines partitions by their UUID
$ cat /mnt/etc/fstab                            # displays the content of the fstab file
</pre>

## Essential system configuration  

Now Arch Linux is technically already installed on your hard drive, but it's not ready to boot just yet. Additional configuration must first be performed, most importantly creating a user and installing a bootloader.

#### Change root to load into the new system:
You are still using the Live Environment, to continue configuration, you must start using the new system installed on your hard drive. Since the system is not able to boot yet, this can be done using the arch-chroot (change root) tool, included in the Arch ISO. Chroot changes the working root directory to that of a new system, mounted on the hard drive and allows us to access the new system.  
<pre>
$ arch-chroot /mnt
</pre>

#### Set time zone:
All available timezones are stored at /usr/share/zoneinfo/"Region"/"City". The system time is decided by /etc/localtime where a selected timezone can be symlinked to using ln -sf. Systemd tool timedatectl makes it even simpler. Lastly, hardware clock should
<pre>
$ timedatectl list-timezones                    # list all available timezones
$ timedatectl set-timezone "Region"/"City"      # set a selected timezone eg. Europe/Amsterdam
$ hwclock --systohc                             # set the hardware clock from the system clock
$ timedatectl                                   # display time date information
</pre>

I need to test this section, I'll delete this code block if timesyncd is enabled by default.
<pre>
$ timedatectl set-ntp true
$ timedatectl status
</pre>

#### Mirror refresh automation:
Since you're using a new Arch system now and not the live USB, you need to configure reflector again. You can use the following configuration, same as before:
<pre>
$ nano /etc/xdg/reflector/reflector.conf        # use nano to edit the reflector configuration file
      --save /etc/pacman.d/mirrorlist           # tells reflector where to save the mirror list
      --country "Country1","Country2..."        # this is optional, sorting latest by rate is often sufficient
      --protocol https                          # also optional but technically more secure
      --sort rate                               # sorts mirrors by download speed
      --latest 5                                # only saves 5 most up to date mirrors
</pre>

Enable a reflector.timer service that will refresh your mirrors automatically once a week, to make sure they're always fast and up to date. Services can be enabled using systemd's systemctl command. Most services will run on every startup when enabled.
<pre>
$ systemctl enable reflector.timer              # enables weekly automatic mirror refresh service
</pre>

#### Hostname configuration:
Hostname is a unique name used to identify a machine on a network, it's stored in /etc/hostname and can be set with systemd using hostnamectl.
<pre>
$ hostnamectl set-hostname "Hostname"           # eg. hostnamectl set-hostname brians-arch-laptop
$ nano /etc/hosts
      127.0.0.1   localhost
      ::1         localhost
      127.0.1.1   "Hostname"
</pre>

Optionally, a "pretty hostname" can be set as well. To do this, edit the /etc/machine-info file:  
<pre>
$ nano /etc/machine-info
      PRETTY_HOSTNAME=""your pretty hostname""    # eg. PRETTY_HOSTNAME="Brian's Arch Laptop"
</pre>

#### Set system locale:
<pre>
$ nano /etc/locale.gen
      # uncomment selected locale
      # eg. en_US.UTF-8 for default American English
      # eg. en_IE.UTF-8 for English with EU formats and metric units
$ locale-gen
$ echo LANG="Selected Locale" > /etc/locale.conf
$ export LANG="Selected Locale"                 # sets locale for all processes for current shell
</pre>

#### Network configuration:
<pre>
$ pacman -S networkmanager
$ systemctl enable NetworkManager.service
# after booting into the system, use $ nmtui or $ nmcli to set up wifi
</pre>

#### Enable multilib - 32 bit app support:  
*(optional, recommended)*
<pre>
$ nano /etc/pacman.conf
      # Uncomment 2 lines: [multilib] and Include =...
$ pacman -Syu
</pre>

#### For LVM, system encryption or RAID:  
*(skip this, only here as reference)*
<pre>
$ nano /etc/mkinitspcio.conf                    # modify initramfs config
$ mkinitcpio -P                                 # recreate image using config
$ mkinitcpio -p linux                           # regenerate using linux defaults
# image must be specified in boot loader config file
</pre>

#### Set root password:
<pre>
$ passwd
</pre>

#### Create a user with administrative privileges, create user password:
<pre>
$ useradd -m -g users -G wheel "user"
$ passwd "user"
</pre>

#### Associate wheel group with sudo:
<pre>
$ EDITOR=nano visudo                            # safely edit /etc/sudoers file
      # Uncomment %wheel ALL=(ALL) ALL
      # Optionally, if you want to require roots instead of users password, add:
            Defaults rootpw
</pre>

#### Install and configure boot loader:  
*(systemd-boot used here, alternatively grub can be used)*
<pre>
$ bootctl install
$ pacman -S "Cpu"-ucode                         # intel-ucode or amd-ucode, depending on your CPU
$ nano /boot/loader/entries/"Entry".conf        # recommended name arch.conf to avoid confusion
      title "Title"                             # eg. title ArchLinux
      linux /vmlinuz-linux
      initrd /"Cpu"-ucode.img                   # intel-ucode.img / amd-ucode.img
      initrd /initramfs-linux.img
$ echo "options root=UUID=$(blkid -s UUID -o value /dev/sdX3) rw" >> /boot/loader/entries/"Entry".conf
      # this copies the UUID of your root partition and adds it to the loader
</pre>

#### Create a pacman hook to automatically update the bootloader:  
*(optional but recommended)*
<pre>
$ nano /etc/pacman.d/hooks/100-systemd-boot.hook
      [Trigger]
      Type = Package
      Operation = Upgrade
      Target = systemd

      [Action]
      Description = Updating systemd-boot
      When = PostTransaction
      Exec = /usr/bin/bootctl update
</pre>

## Installing GPU drivers

#### Identify the GPU:
<pre>
$ lspci -k | grep -A 2 -E "(VGA|3D)"
</pre>

***

#### - Intel integrated graphics:
<pre>
$ pacman -S xf86-video-intel                                # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
            vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            vulkan-intel lib32-vulkan-intel                 # Vulkan drivers
</pre>

#### - AMD modern graphics:  
for GCN3/Radeon Rx 300 and newer cards
<pre>
$ pacman -S xf86-video-amdgpu                               # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
            vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            vulkan-radeon lib32-vulkan-radeon               # Vulkan drivers
</pre>

#### - AMD legacy graphics:  
for GCN2/Radeon Rx 200 and older cards  
*(more details on https://wiki.archlinux.org/title/ATI)*
<pre>
$ pacman -S xf86-video-ati                                  # DDX driver and 2D acceleration
            mesa lib32-mesa                                 # DRI driver for 3D acc / OpenGL
</pre>

#### - AMD Radeon HD 7000 and RX200 generations:  
*https://wiki.archlinux.org/title/AMDGPU#Experimental*  
*(section needs expanding)*

#### - Nvidia:  

Install only ONE of the following 3 packages, depending on your kernel:***
<pre>
$ pacman -S nvidia                                          # for standard linux kernel
            nvidia-lts                                      # for the linux-lts kernel
            nvidia-dkms                                     # other kernels eg. linux-zen
</pre>

Install all of the following packages:  
*(providing OpenGL, Vulkan, OpenCL support and configuration tools)*
<pre>
$ pacman -S vulkan-icd-loader lib32-vulkan-icd-loader       # Vulkan
            nvidia-utils lib32-nvidia-utils                 # OpenGL, Vulkan
            opencl-nvidia                                   # OpenCL support
            xorg-server-devel                               # Provides nvidia-xconfig
            nvidia-settings                                 # Nvidia settings menu

$ nvidia-xconfig                                            # optional auto-configuration for xorg
$ nvidia-settings                                           # settings GUI, use after installation
</pre>

Enable DRM (direct rendering manager):  
by adding a drm kernel mode setting in your boot loader entry
<pre>
$ nano /boot/loader/entries/"Entry".conf
      # set kernel parameter: nvidia-drm.modeset=1 after options root=(...) rw
      # eg. options root="UUID=361cn91740c73-2x9mm871c2dsa" rw nvidia-drm.modeset=1
</pre>

Ensure early loading of the drivers:  
*(optional, recommended?)*
<pre>
$ nano /etc/mkinitcpio.conf
      MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
</pre>

Automatically update initramfs after updating kernel or driver:  
*(required when using early loading or custom kernels)*
<pre>
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
</pre>

#### - PRIME:  
hybrid graphics on laptops - Intel/AMD+Nvidia/AMD  
check arch wiki, if you have a laptop with a modern dedicated gpu  
*(this section needs expanding)*

***

#### Vulkan verification:  
*(optional, recommended)*
<pre>
$ ls /usr/share/vulkan/icd.d/                   # shows installed vulkan implementations
$ pacman -S vulkan-tools
$ vulkaninfo                                    # should show info about your GPU
</pre>

## Hardware video acceleration  

#### - Intel integrated graphics:  
pick one or none depending on your CPU
<pre>
$ pacman -S intel-media-driver                  # 5th gen and newer intel CPUs
          libva-intel-driver                    # 3rd and 4th gen intel CPUs
</pre>

#### - AMD:  
install both unless running an ancient GPU
<pre>
$ pacman -S libva-mesa-driver                   # R600 (Radeon HD 2000) and newer
          mesa-vdpau                            # R300 (Radeon 9000) and newer
</pre>

#### - Nvidia:  
hardware accelerated video decoding is provided by nvidia-utils  
for encoding with NVENC, create this rule
<pre>
$ nano /etc/udev/rules.d/70-nvidia.rules        
      ACTION=="add", DEVPATH=="/bus/pci/drivers/nvidia", 
      RUN+="/usr/bin/nvidia-modprobe -c0 -u"
</pre>

#### Verify hardware video acceleration:  
*(Optional, untested)*
<pre>
$ pacman -S libva-utils	vdpauinfo
$ vainfo                                        # verify VA-API settings
$ vdpauinfo                                     # verify VDPAU settings
$ pacman -S intel-gpu-tools                     # for intel GPUs
$ intel-gpu-top                                 # play video and monitor gpu usage
$ pacman -S radeontop                           # for AMD GPUs
$ radeontop                                     # play video and monitor gpu usage
</pre>

## Booting into the system, additional configuration

## Installing a Desktop Environment, Display Manager, enabling services

## Performance optimizations

## Power optimizations (for laptops)

## Essentials of system management & Troubleshooting
