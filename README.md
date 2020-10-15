# Arch with KDE Plasma Installation Guide (MBR & UEFI)

- We are going to install Arch and KDE Plasma packages required for a minimal system up & running.
- Grab the latest Arch Image ISO from https://www.archlinux.org/download/ and write it to an USB Stick.
- After the image is done writing, it's time to boot the into Arch Live ISO. First thing to do after you land onto Live ISO terminal is to:

## Getting started

### Check for Internet Connectivity
```
ping -t 4 google.com
```
- If you are using Wi-Fi, then use `wifi-menu` to connect to your local network
- If this is successful then we will head to next step

### Update system clock
```
timedatectl set-ntp true
```

## Disk Partitioning & Mounting (MBR)
We are going to make two partitions on our HDD, `1. SWAP & 2. ROOT` using `cfdisk`.
```
cfdisk /dev/sd*
```
- "*" = disk drive, find your by running `lsblk`
- SWAP Partition should double the size of RAM available in your system
- We will be using one partition for our ROOT, boot & home

### Format the Partition, Make SWAP & Mount ROOT
##### ROOT
```
mkfs.ext4 /dev/sd*2
```
##### SWAP
```
mkswap /dev/sd*1
swapon /dev/sd*1
```
#### Mount ROOT
```
mount /dev/sd*2 /mnt
```

## Disk Partitioning (UEFI)
Coming soon...


## Base System Installation

### Update Mirrors using Reflector
```
reflector --country County1,Country2 --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
Replace Country1 & Country2 with country names near to you or with the one you're living in. Refer to https://wiki.archlinux.org/index.php/reflector for more info.

### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware nano intel-ucode reflector
```
Replace `linux` with linux-hardened, linux-lts or linux-zen to install the kernel of your choice.
Replace `nano` with editor of your choice (vim or vi).
Replace `intel-ucode` with `amd-ucode` if you are using an AMD Processor.

### Generate yor fstab (use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/index.php/UUID) or labels, respectively): 
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors. 

## Chroot
```
arch-chroot /mnt
```

### Set Time & Date
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```
Replace `Region` & `City` with your region and city. Refer to https://wiki.archlinux.org/index.php/installation_guide#Time_zone.

## Set Language (en_US.UTF-8 as default Language)
If you want to set your language, please read 

#### Edit locale.gen
```
nano /etc/locale.gen
```
Uncomment the below line from
```
#en_US.UTF-8 UTF-8
```
To
```
en_US.UTF-8 UTF-8
```
- save & exit

Generate Locale
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
save & exit

### Install & Enable NetworkManager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

### Set ROOT Password
```
passwd
```

### GRUB Bootloader
```
pacman -S grub
grub-install --target=i386-pc /dev/sd*
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

### Add user
```
useradd -mG wheel username
```
Replace `username` with your username of choice.

## Set User Password
```
passwd username
```

### Set Wheel Group to use Sudo Command
```
EDITOR=nano visudo
```

#### Find and uncomment the below line
```
%wheel ALL=(ALL) ALL
```
save & exit

### Exit from ROOT
```
exit
```

## Login as USER

### Check for updates
```
sudo pacman -Syu
```

### XOrg & GPU Drivers
```
sudo pacman -S xorg xf86-video-xxx
```

### Enable Multilib Repo


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
plasma-desktop | Minimal Plasma DE installation
konsole | KDE Terminal
dolphin | KDE default File Manager
ark | Archiving Tool
kwrite | Text Editor
kcalc | Scientific Calculator
spectacle | KDE screenshot capture utility
ksysguard | KDE System Task Monitor
krunner | KDE Quick drop-down search menu
kscreen | KDE Display Setting Manager
partitionmanager | KDE Disk & Partion Manager

### My Required Applications
```
sudo pacman -S firefox qbittorrent wget github neofetch zsh
```

### Install YAY
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
### Final Reboot
```
reboot
```
