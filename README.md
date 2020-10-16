# Arch with KDE Plasma Installation Guide (UEFI & MBR)

Hello everyone, This is my guide for installing minimal Arch Linux with KDE Plasma Desktop Environment. In this guide we will go step by step on how I install my Arch System and set everything up from scratch for a stable & healthy OS.

## Table of Contents
  * [Let's Begin](#lets-begin)
  * [Disk Partitioning](#preparing-the-disk-for-system)
    * [UEFI System](#for-uefi-system)
    * [MBR System](#for-mbr-system)
  * [Base System Installation](#base-system-installation)
  * [Update Mirrors](#update-mirrors-using-reflector)
  * [Base System](https://github.com/XxAcielxX/arch-plasma-install#install-base-system)
  * [Generate fstab](#generate-yor-fstab-use--u-or--l-to-define-by-uuid-or-labels-respectively)
  * [Chroot](#chroot)
    * [Swapfile (UEFI only)](#create-swapfile-uefi-only)
    * [Date & Time](#set-time--date)
    * [Language](#set-language)
    * [Hostname & Hosts](#set-hostname)
    * [Network Manager](#install--enable-networkmanager) 
    * [ROOT Password](#set-root-password) 
    * [GRUB Bootloader](#install-grub-bootloader) 
      * [UEFI System](#for-uefi-system-1) 
      * [MBR System](#for-mbr-system-1)
  * [Boot Freshly Installed System](#unplug-the-usb-stick-and-boot-into-your-freshly-installed-arch-system)
    * [Add User](#add-new-user) 
    * [Sudo Command](#set-wheel-group-to-use-sudo-command) 
  * [User Login](#login-as-user)
    * [Display Drivers & GPU Drivers](#xorg--gpu-drivers)
    * [Multilib Repository (32bit)](#enable-multilib-repo-optional)
    * [Display Manager (SDDM)](#install--enable-sddm)
    * [Desktop Environment (KDE Plasma)](#kde-plasma--applications)
    * [Misc Applications](https://#my-required-applications)
  * [Extras](#extras)
    * [Yay](#install-yay)
  * [Theming & Customisations](theming--customisations)
  * [Maintenance & Performance Tuning](maintenance--performance-tuning)
    * [Paccache](#paccache)

## Let's begin

- Grab the latest Arch Image ISO from https://www.archlinux.org/download/ and write it to an USB Stick.
- After the image is done writing, it's time to boot the into Arch Live ISO. First thing to do after you land onto Live ISO terminal.

### Check for Internet Connectivity
```
ping -t 4 google.com
```
- If you are connected through Ethernet, then your Internet will be working out of the box.
- If you are using Wi-Fi, then use `wifi-menu` to connect to your local network.
- If this step is successful then we will head to next one.

### Update system clock
```
timedatectl set-ntp true
```
## Preparing the Disk for System

### \*** WARNING ***</br>
>> Be extremely careful when managing your disks, incase you delete your precious data then DON'T blame me.</br>
>> Disk Partition (use UEFI or MBR, go according to your system)

## For UEFI System

### Disk Partitioning (UEFI)
We are going to make two partitions on our HDD, `1. EFI BOOT & 2. ROOT` using `gdisk`.
```
gdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk` and replace in all the below instances.
- If you have a brand new HDD then create GPT Partition Table by pressing `g`.
- We will be using one partition for our `/`, `/boot` & `/home`. 

```
n = New Partition
1 = 1st Partition 
+512M = BOOT Partition Size
ef00 = EFI Partition Type

n = New Partition again
2 = 2nd Partition 
simply press enter = ROOT Partition Size (using the remaining space left)
8300 or simply press enter = EXT4 ROOT Partition Type
```
### Format Partitions (UEFI)
```
mkfs.fat -F32 /dev/[efi partition name]
mkfs.ext4 /dev/[root partiton name] 
```

### Mount Partitions (UEFI)
```
mount /dev/[root partition name] /mnt
mkdir /mnt/boot/efi
mount /dev/[efi partition name] /mnt/boot/efi
```
## For MBR System

### Disk Partitioning (MBR)
We are going to make two partitions on our HDD, `1. SWAP & 2. ROOT` using `cfdisk`.
```
cfdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk` and replace in all the below instances.
- If you have a brand new HDD then create MSDOS Partition Table by selecting `msdos`.
- SWAP Partition should double the size of RAM available in your system.
- We will be using one partition for our `/`, `/boot` & `/home`.

### Format the Partition, Make SWAP & Mount ROOT (MBR)
##### Format ROOT Partition as EXT4
```
mkfs.ext4 /dev/[root partition name]
```
##### Make & Turn SWAP Partition on (MBR)
```
mkswap /dev/[swap partition name]
swapon /dev/[swap partition name]
```
#### Mount ROOT Partition (MBR)
```
mount /dev/[root partition name] /mnt
```

## Base System Installation

### Update Mirrors using Reflector
```
reflector --country County1,Country2 --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
Replace Country1 & Country2 with country names near to you or with the one you're living in. Refer to https://wiki.archlinux.org/index.php/reflector for more info.

### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano intel-ucode reflector
```
- Replace `linux` with *linux-hardened*, *linux-lts* or *linux-zen* to install the kernel of your choice.
- Replace `linux-headers` with Kernel type type of your choice respectively (e.g if you installed `linux-zen` then you will need `linux-zen-headers`).
- Replace `nano` with editor of your choice (i.e `vim` or `vi`).
- Replace `intel-ucode` with `amd-ucode` if you are using an AMD Processor.

### Generate yor fstab (use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/index.php/UUID) or labels, respectively): 
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors. 

## Chroot

```
arch-chroot /mnt
```
### Create Swapfile (UEFI only)
```
dd if=/dev/zero of=/swapfile bs=1G count=2 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```
Replace the above 2 in `count=2` with 2 x RAM Size. (e.g it you have 8GB, then 2x8 = 16 use `count=16`).

### Add Swapfile entery in your `/etc/fstab` file (UEFI only) 
```
/swapfile none swap defaults 0 0
```
Insert the above line at the bottom of `/etc/fstab`.

### Set Time & Date
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```
Replace `Region` & `City` according to your Time-Zone . Refer to https://wiki.archlinux.org/index.php/installation_guide#Time_zone.

## Set Language
We will use `en_US.UTF-8` as our default language here but, if you want to set your language, please read 

#### Edit locale.gen
```
nano /etc/locale.gen
```
Uncomment the below line
```
#en_US.UTF-8 UTF-8
```
save & exit.

### Generate Locale
```
locale-gen
```

### Add LANG to locale.conf
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## Set Hostname

```
echo arch > /etc/hostname
```
Replace `arch` with hostname of your choice.

### Set Hosts
```
nano /etc/hosts
```
#### add these lines to it
```
127.0.0.1    localhost
::1          localhost
127. 0.1.1   arch.localdomain arch
```
Replace `arch` with hostname of your choice.
save & exit.

### Install & Enable NetworkManager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

### Set ROOT Password
```
passwd
```

### Install GRUB Bootloader
```
pacman -S grub
```

#### For UEFI System
```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

#### For MBR System
```
grub-install --target=i386-pc /dev/[disk name]
```

### Create Grub configuration file
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Final Step
```
exit 
umount -a
reboot
```

## Unplug the USB Stick and boot into your freshly installed Arch system

### Login as ROOT

### Add new User
```
useradd -mG wheel [username]
```
Replace `[username]` with your username of choice.

### Set User Password
```
passwd [username]
```

### Set Wheel Group to use Sudo Command
```
EDITOR=nano visudo
```

#### Find and uncomment the below line
```
#%wheel ALL=(ALL) ALL
```
save & exit

### Logout ROOT
```
exit
```

## Login as USER

### Check for updates
```
sudo pacman -Syu
```

### xorg & GPU Drivers
```
sudo pacman -S xorg xf86-video-[your gpu type]
```
For Nvidia GPUs, type `xf86-video-nvidia`.</br>
For newer AMD GPUs, type `xf86-video-amdgpu`.</br>
For legacy Radeon GPUs like HD 7xxx Series & below, type `xf86-video-ati`.</br>

### Enable Multilib Repo (optional)
multilib contains 32-bit software and libraries that can be used to run and build 32-bit applications on 64-bit installs (e.g. [Wine](https://wine.hq), [Steam](https://store.steampowered.com/), etc).

Edit `/etc/pacman.conf` & uncomment the below section.
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

### Install & Enable SDDM
```
sudo pacman -S sddm
sudo systemctl enable sddm
```

### KDE Plasma & Applications
```
sudo pacman -S plasma-desktop konsole dolphin ark kwrite kcalc spectacle ksysguard krunner kscreen partitionmanager
```
Packages | Description
------------ | -------------
plasma-desktop | Minimal Plasma DE installation.
konsole | KDE Terminal.
dolphin | KDE default File Manager.
ark | Archiving Tool.
kwrite | Text Editor.
kcalc | Scientific Calculator.
spectacle | KDE screenshot capture utility.
ksysguard | KDE System Task Monitor.
krunner | KDE Quick drop-down desktop search.
kscreen | KDE Display Setting Manager.
partitionmanager | KDE Disk & Partion Manager.

### My Required Applications
```
sudo pacman -S firefox qbittorrent wget git neofetch zsh
```
Packages | Description
------------ | -------------
firefox | Mozilla Firefox Web Browser.
qbittorrent | A BitTorrent Client based on QT.
wget | Wget is a free utility for non-interactive download of files from the Web.
git | Github Utility Tools.
neofetch | Neofetch is a command-line system information tool.
zsh | The Z shell (Zsh) is a Unix shell that can be used as an interactive login shell and as a command interpreter for shell scripting.

### Final Reboot
```
reboot
```

## Extras

### Install [Yay](https://github.com/Jguer/yay)
Yet Another Yogurt - An AUR Helper.
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## Theming & Customisations
**[Section coming soon...]**

## Maintenance & Performance Tuning

### Paccache
Pacman Cache Cleaner.

Install
```
sudo pacman -S pacman-contrib
```

To manually clean pacman cache, run
```
paccache -rk1
```
Where, *k* indicates to keep "num" of each package in the cache.

#### To automate paccache process

Create a file in `/etc/pacman.d/hooks`
```
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
```

Add the following lines in it
```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk1
```
