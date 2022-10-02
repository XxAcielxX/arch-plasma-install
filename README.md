[<img src="https://img.shields.io/badge/BTC-3Hqd4mameE1GzDNrfj2V9KAWaL7sUxJtA8-E66000?labelColor=353535&style=for-the-badge&logo=btc"/>](https://en.cryptobadges.io/donate/3Hqd4mameE1GzDNrfj2V9KAWaL7sUxJtA8)

(Works with Arch ISO Image build as of: 2022.10.01)

# Arch Linux with KDE Plasma Installation Guide (UEFI & MBR)

Hello everyone, This is my guide for installing minimal Arch Linux with KDE Plasma Desktop Environment. In this guide we will go step by step on how I install my Arch System and set everything up from scratch for a stable & healthy OS.
</br>

## Table of Contents
  * [**Let's Begin**](#lets-begin)
  * [**Disk Partitioning**](#preparing-the-disk-for-system)
    * [UEFI System](#for-uefi-system)
    * [MBR System](#for-mbr-system)
  * [**Base System Installation**](#base-system-installation)
    * [Update Mirrors](#update-mirrors-using-reflector)
    * [Base System](https://github.com/XxAcielxX/arch-plasma-install#install-base-system)
    * [Generate fstab](#generate-fstab)
  * [**Chroot**](#chroot)
    * [Swapfile (UEFI only)](#create-swapfile-uefi-only)
    * [Date & Time](#set-time--date)
    * [Language](#set-language)
    * [Hostname & Hosts](#set-hostname)
    * [Network Manager](#install--enable-networkmanager) 
    * [ROOT Password](#set-root-password) 
    * [GRUB Bootloader](#install-grub-bootloader) 
      * [UEFI System](#for-uefi-system-1) 
      * [MBR System](#for-mbr-system-1)
  * [**Boot Freshly Installed System**](#now-boot-into-your-freshly-installed-arch-system)
    * [Add User](#add-new-user) 
    * [Sudo Command](#allow-wheel-group-to-use-sudo-commands) 
  * [**User Login**](#login-as-user)
    * [Display Server & GPU Drivers](#xorg--gpu-drivers)
    * [Multilib Repository (32bit)](#enable-multilib-repo-optional)
    * [Display Manager (SDDM)](#install--enable-sddm)
    * [Desktop Environment (KDE Plasma)](#kde-plasma--applications)
    * [Audio Utilities & Bluetooth](#audio-utilities--bluetooth)
    * [Misc Applications](https://#my-required-applications)
  * [**The Conclusion**](#the-conclusion) 
  * [**Extras (optional)**](#extras-optional)
    * [Yay](#install-yay)
    * [Zsh](#install-zsh)
    * [Change SHELL](#changing-your-shell)
    * [PipeWire](#pipewire)
    * [EasyEffects](#easyeffects)
    * [Clam AntiVirus](#clamav)
    * [Printer Service](#printer-service)
  * [**Theming & Customisations**](#theming--customisations)
     * [Oh My Zsh & Powerlevel10k Theme](#install-oh-my-zsh)
     * [Kvantum Manager](#kvantum-manager)
  * [**Maintenance, Performance Tuning & Monitoring**](maintenance-performance-tuning--monitoring)
    * [Paccache](#paccache)
    * [Cockpit](#install-cockpit)
  * [**Changelog**](#changelog)
</br>

## Let's begin
- Grab the latest built ISO Image from **[Arch Linux Download](https://www.archlinux.org/download/)** and write it to an empty USB Stick.
- After the image is done writing, it's time to boot into the Arch Live Environment. First thing you do is:

### Load Keymaps (for non US ENG Keyboard Users only)
For a list of all the available keymaps, use the command:
```
localectl list-keymaps
```

To search for a keymap, use the following command, replacing `[search_term]` with the code for your language, country, or layout:
```
localectl list-keymaps | grep -i [search_term]
```

### Now Loadkeys
```
loadkeys [keymap]
```

### Check for Internet Connectivity
```
ping -c 4 google.com
```
- If you are connected through Ethernet, then your Internet will be working out of the box.
- If you are using Wi-Fi, then use `wifi-menu` to connect to your local network.
- If this step is successful then we will head to next one.

### Update system clock
```
timedatectl set-ntp true
```
</br>

## Preparing the Disk for System

> :warning: Be extremely careful when managing your disks, incase you delete your precious data then DON'T blame me.
Disk partitioning type (use UEFI or MBR, go according to your system).

## For UEFI System

### Disk Partitioning (UEFI)
We are going to make two partitions on our HDD, `EFI BOOT & ROOT` using `gdisk`.
- If you have a brand new HDD or if no partition table is found, then create GPT Partition Table by pressing `g`.
```
gdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk`.
- We will be using one partition for our `/`, `/boot` & `/home`. 

```
n = New Partition
simply press enter = 1st Partition
simply press enter = As First Sector
+512M = As Last sector (BOOT Partition Size)
ef00 = EFI Partition Type

n = New Partition again
simply press enter = 2nd Partition
simply press enter = As First Sector 
simply press enter = As Last sector [ROOT Partition Size (using the remaining disk space left)]
8300 or simply press enter = EXT4 ROOT Partition Type

w = write & exit
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
We are going to make two partitions on our HDD, `SWAP & ROOT` using `cfdisk`.
- If you have a brand new HDD or if no partition table is found, then create MSDos Partition Table by selecting `msdos`.
```
cfdisk /dev/[disk name]
```
- [disk name] = device to partition, find yours by running `lsblk`.
- SWAP Partition should double the size of RAM available in your system. Not applicable on 16GB or more RAM.
- We will be using one partition for our `/`, `/boot` & `/home`.

### Format the Partition, Make SWAP & Mount ROOT (MBR)
#### Format ROOT Partition as EXT4
```
mkfs.ext4 /dev/[root partition name]
```
#### Make & Turn SWAP Partition on (MBR)
```
mkswap /dev/[swap partition name]
swapon /dev/[swap partition name]
```
#### Mount ROOT Partition (MBR)
```
mount /dev/[root partition name] /mnt
```
</br>

## Base System Installation

### Update Mirrors using Reflector
```
reflector -c County1 -c Country2 -a 12 -p https --sort rate --save /etc/pacman.d/mirrorlist
```
Replace `Country1` & `Country2` with countries near to you or with the one you're living in. Refer to **[Reflector](https://wiki.archlinux.org/index.php/reflector)** for more info.

### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano intel-ucode reflector mtools dosfstools
```
- Replace `linux` with *linux-hardened*, *linux-lts* or *linux-zen* to install the kernel of your choice.
- Replace `linux-headers` with Kernel type type of your choice respectively (e.g if you installed `linux-zen` then you will need `linux-zen-headers`).
- Replace `nano` with editor of your choice (i.e `vim` or `vi`).
- Replace `intel-ucode` with `amd-ucode` if you are using an AMD Processor.

### Generate fstab
(use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/index.php/UUID) or labels, respectively)
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors. 
</br>

## Chroot

```
arch-chroot /mnt
```
### Create Swapfile (UEFI only)
Replace the below 4096 in `count=4096` with double the amount of RAM installed your system. Not applicable on 16GB or more RAM.
```
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

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
Replace `Region` & `City` according to your Time zone. Refer to **[Time zone](https://wiki.archlinux.org/index.php/installation_guide#Time_zone)** more info.

## Set Language
We will use `en_US.UTF-8` here but, if you want to set your language, replace `en_US.UTF-8` with yours in all below instances.

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

### Add Keymaps to vconsole
For keyboard users with non US Eng only. Replace `[keymap]` with yours.
```
echo "KEYMAP=[keymap]" > /etc/vconsole.conf
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
127.0.1.1    arch.localdomain arch
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
</br>

## Now boot into your freshly installed Arch system

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

### Allow Wheel Group to use Sudo Commands
```
EDITOR=nano visudo
```

#### Find and uncomment the below line
```
#%wheel ALL=(ALL) ALL
```
save & exit.

### Logout ROOT
```
exit
```

## Login as USER

### Check for updates
```
sudo pacman -Syu
```

### Xorg & GPU Drivers
```
sudo pacman -S xorg [xf86-video-your gpu type]
```
- For Nvidia GPUs, type `nvidia` & `nvidia-settings`. For more info/old GPUs, refer to [Arch Wiki - Nvidia](https://wiki.archlinux.org/index.php/NVIDIA).
- For newer AMD GPUs, type `xf86-video-amdgpu`.
- For legacy Radeon GPUs like HD 7xxx Series & below, type `xf86-video-ati`.
- For dedicated Intel Graphics, type `xf86-video-intel`.

### Enable Multilib Repo (optional)
multilib contains 32-bit software and libraries that can be used to run and build 32-bit applications on 64-bit installs (e.g. [Wine](https://www.winehq.org/), [Steam](https://store.steampowered.com/), etc).

Edit `/etc/pacman.conf` & uncomment the below two lines.
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

#### MESA Libraries (32bit)
This package is required by Steam if you play games using Vulkan Backend.
```
sudo pacman -S lib32-mesa
```

### Install & Enable SDDM
```
sudo pacman -S sddm
sudo systemctl enable sddm
```

### KDE Plasma & Applications
```
sudo pacman -S plasma konsole dolphin ark kwrite kcalc spectacle krunner partitionmanager packagekit-qt5
```
Packages | Description
--------- | ----------
plasma | KDE Plasma Desktop Environment.
konsole | KDE Terminal.
dolphin | KDE File Manager.
ark | Archiving Tool.
kwrite | Text Editor.
kcalc | Scientific Calculator.
spectacle | KDE screenshot capture utility.
krunner | KDE Quick drop-down desktop search.
partitionmanager | KDE Disk & Partion Manager.

### Audio Utilities & Bluetooth
```
sudo pacman -S alsa-utils bluez bluez-utils
```
Packages | Description
--------- | ----------
alsa-utils | This contains (among other utilities) the `alsamixer` and `amixer` utilities.
bluez | Provides the Bluetooth protocol stack.
bluez-utils | Provides the `bluetoothctl` utility.

#### Enable Bluetooth Service
```
sudo systemctl enable bluetooth.service
```

### My Required Applications
You can install all the following packages or only the one you want.
```
sudo pacman -S firefox openssh qbittorrent audacious wget screen git neofetch
```
Packages | Description
--------- | ----------
firefox | Mozilla Firefox Web Browser.
openssh | Secure Shell access server.
qbittorrent | A BitTorrent Client based on Qt.
audacious | Qt based music player. 
wget | Wget is a free utility for non-interactive download of files from the Web.
screen | Is a full-screen window manager that multiplexes a physical terminal between several processes, typically interactive shells.
git | Github command-line utility tools.
neofetch | Neofetch is a command-line system information tool.

### Enable OpenSSH daemon
```
sudo systemctl enable sshd.service
```

### Final Reboot
```
reboot
```
</br>

### The Conclusion
Now everything is installed and after the final `reboot`, you will land in you GUI Login Screen ready to use your system. You can also do some extra steps mentioned below to further improve your experience. I'll recommend you to install `yay` & `paccache`.
- Yay will provide your packages from AUR (Arch User Repository), which are not available in the Official Repo.
- Paccache can be used clean pacman cached packages either manually or in an automated way.
</br>

## Extras (optional)

### Install [Yay](https://github.com/Jguer/yay)
Yet Another Yogurt - An AUR Helper.
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Install [Zsh](https://wiki.archlinux.org/index.php/zsh/) 
Zsh is a powerful shell that operates as both an interactive shell and as a scripting language interpreter.
```
sudo pacman -S zsh zsh-completions
```
Read *[here](#install-oh-my-zsh)* for customisation & theming for Zsh. Read below how to change your SHELL.

### Changing your SHELL
First check your current SHELL by running:
```
echo $SHELL
```

#### To get list of all available/installed SHELLs:
```
chsh -l
```

### Set Zsh as our SHELL
For an example, we will set Zsh as default SHELL which we installed in the last step:
```
chsh -s /usr/bin/zsh
```
For the changes to apply, you will have Logout and Log back in or better do `reboot`.

## PipeWire
[PipeWire](https://wiki.archlinux.org/title/PipeWire) is a new low-level multimedia framework. It aims to offer capture and playback for both audio and video with minimal latency and support for PulseAudio, JACK, ALSA and GStreamer-based applications. 
#### Install
```
sudo pacman -S pipewire
```

## EasyEffects
[EasyEffects](https://wiki.archlinux.org/title/PipeWire#EasyEffects) (former PulseEffects) is a GTK utility which provides a large array of audio effects and filters to individual application output streams and microphone input streams. Notable effects include an input/output equalizer, output loudness equalization and bass enhancement, and input de-esser and noise reduction plug-in.
Install
```
sudo pacman -S easyeffects
or
yay -S easyeffects-git
```
> This will also install pipewire-pulse and replace PulseAudio with PipeWire.

## ClamAV
[Clam AntiVirus](https://wiki.archlinux.org/index.php/ClamAV) is an open source (GPL) anti-virus toolkit for UNIX. It provides a number of utilities including a flexible and scalable multi-threaded daemon, a command line scanner and advanced tool for automatic database updates.
#### 1. Install
```
sudo pacman -S clamav
```

#### 2. Update Signatures/Database (must do)
```
sudo freshclam
```

#### 3. Enable & start services
```
sudo systemctl enable --now clamav-freshclam.service
sudo systemctl enable --now clamav-daemon.service
```

#### 4a. ClamTK (optional)
GUI for ClamAV
```
sudo pacman -S clamtk
```

#### 4b. KDE Dolphin File Manager Plugin (optional)
Download the latest `master zip` from [ClanTK-KDE Gitlab](https://gitlab.com/dave_m/clamtk-kde) & extract it your `~/Downloads` folder. Now open a terminal from within the extracted folder & run:
```
sudo cp clamtk-kde.desktop /usr/share/kservices5/ServiceMenus/
```

### Printer Service
```
sudo pacman -S cups
```

#### Enable CUPS (Printer) Service
```
sudo systemctl enable --now cups.service
```
</br>

## Theming & Customisations

### Install [Oh My Zsh](https://ohmyz.sh/) 
Oh My Zsh is an open source, community-driven framework for managing your Zsh configuration.
```
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
My favourite theme is Powerlevel10k (follow below for installation).
- You can visit [here](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) to download theme of your choice.

### Get [Powerlevel10k](https://github.com/romkatv/powerlevel10k/) Theme for Oh My Zsh
This is the theme I'll install to spice up my terminal experience.
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

#### Get the recommended fonts
We will be using ***Yay*** to install the below two packages as one of them is only available from AUR.
```
yay -S ttf-dejavu ttf-meslo-nerd-font-powerlevel10k
```
Also set your Konsole Terminal font to `MesloGS-NF-Regular`.

#### Set Powerlevel10k as your Zsh Theme
```
nano ~/.zshrc
```
Find the line starting with `ZSH_THEME="...."` and replace the theme name so the line should now look like this `ZSH_THEME="powerlevel10k/powerlevel10k"` Now do `source ~/.zshrc`.
#### Configuration
> ***For new users***, on the first run, Powerlevel10k configuration wizard will ask you a few questions and configure your prompt. If it doesn't trigger automatically, type `p10k configure`. Configuration wizard creates `~/.p10k.zsh` based on your preferences. Additional prompt customization can be done by editing this file. It has plenty of comments to help you navigate through configuration options.

## Kvantum Manager
[Kvantum](https://github.com/tsujan/Kvantum) is a SVG-based theme engine for Qt, tuned to KDE and LXQt, with an emphasis on elegance, usability and practicality.

### Install through Yay (git version)
```
yay -S kvantum-qt5-git
```

***Or***

### Install through Pacman
```
sudo pacman -S kvantum-qt5
```

</br>

## Maintenance, Performance Tuning & Monitoring

### Paccache
Pacman Cache Cleaner.

Install
```
sudo pacman -S pacman-contrib
```

To manually clean pacman cache, run
```
sudo paccache -rk
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
Exec = /usr/bin/paccache -rk
```
save & exit.

### Install [Cockpit](https://cockpit-project.org/)
A systemd web based user interface for Linux servers, Workstations and even Desktops. Can be used to monitor your system stats, performance and perform various settings including updating of your system.
```
sudo pacman -S cockpit
```

##### Enable Cockpit
```
sudo systemctl enable --now cockpit.socket
```
Now open your browser and point to it `your-machine-ip:9000` and login with ***root*** to get full access.

</br>

## Changelog


  * **2022-10-02**
    * Updated guide compatibility for `2022-10-01` iso image.
  * **2022-08-05**
    * Updated guide compatibility for `2022-08-01` iso image.
  * **2022-07-02**
    * Updated guide compatibility for `2022-07-01` iso image.
  * **2022-04-14**
    * Updated guide compatibility for `2022-04-01` iso image.
  * **2022-01-18**
    * Updated guide compatibility for `2022-01-01` iso image.
  * **2021-10-19**
    * Updated `EasyEffects` with new package name, formerly known as PulseEffects.
  * **2021-10-05**
    * Updated guide compatibility for `2021-10-01` iso image.
  * **2021-09-09**
    * Updated guide compatibility for `2021-09-01` iso image.
  * **2021-08-01**
    * Updated guide compatibility for `2021-08-01` iso image.
  * **2021-06-14**
    * Updated guide compatibility for `2021-06-01` iso image.
  * **2021-05-12**
    * Updated guide compatibility for `2021-05-01` iso image.
    * Added `PipeWire` & `PulseEffects` packages section under *Extras*.
  * **2021-04-05**
    * Updated guide compatibility for `2021-04-01` iso image.
  * **2021-02-22**
    * Updated guide compatibility for `2021-02-01` iso image.
  * **2020-11-22**
    * Added `CUPS` Printer Service under *Extras* Section.
  * **2020-11-08**
    * Updated guide compatibility for `2020-11-01` iso image.
    * Added `ClamAV` AntiVirus/Security for protecting our Arch system.
  * **2020-10-24**
    * Edited `Zsh` & `Oh-My-Zsh` Sections and added further details.
    * Added `Change SHELL` Section.
  * **2020-10-22**
    * Added `Kvantum Manager` under *Theming* Section.
  * **2020-10-18**
    * Added *Audio Utilities & Bluetooth* Section.
    * Added `Zsh` along with `Oh My Zsh` & `Powerlevel10k` theme.
    * Added `cockpit` under *Maintenance, Performance Tuning & Monitoring* Section.
  * **2020-10-17**
    * Added `lib32-mesa` package under *Multilib Repo* Section.
    * Added Discord Badge.
    * Added Donate BTC Badge.
    * Improved Guide text formatting for easier reading.
  * **2020-10-16**
    * Initial guide created.
