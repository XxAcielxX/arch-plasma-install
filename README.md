# Arch with KDE Plasma Installation Guide (MBR & UEFI)

We are going to install Arch and KDE Plasma packages required for a minimal system up & running.

Grab the latest Arch Image ISO from https://www.archlinux.org/download/ and write it to an USB Stick.

After the image is done writing, it's time to boot the into Arch Live ISO. First thing to do after you land onto Live ISO terminal is to:

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
- Replace Country1 & Country2 with country names near to you or with the one you're living in. Refer to https://wiki.archlinux.org/index.php/reflector for more info

### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware nano intel-ucode reflector
```
- Replace `linux` with linux-hardened, linux-lts or linux-zen to install the kernel of your choice
- Replace `nano` with editor of your choice (vim or vi)
- Replace `intel-ucode` with amd-ucode if you are using AMD Processor
