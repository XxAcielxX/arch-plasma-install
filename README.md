[<img src="https://img.shields.io/badge/BTC-1USXozdnotw3u5XkzooUadw7NrdyV752V-E66000?labelColor=353535&style=for-the-badge&logo=btc"/>](https://acielgaming.cb.id)

(Works with Arch ISO Image build as of: 2024.01.*)

# Arch Linux with KDE Plasma Installation Guide (UEFI & MBR)

Hello everyone, This is my guide for installing minimal Arch Linux with KDE Plasma Desktop Environment. In this guide we will go step by step on how I install my Arch System and set everything up from scratch for a stable & healthy OS.
</br>

## Table of Contents
 - [**Let's Begin**](#lets-begin)
 - [**Connect to the Internet**](#connect-to-the-internet)
 - [**Disk Partitioning**](#preparing-the-disk-for-system)
 - [**Base System Installation**](#base-system-installation)
   - [Update Mirrors](#update-mirrors-using-reflector)
   - [Base System](https://github.com/XxAcielxX/arch-plasma-install#install-base-system)
   - [Generate fstab](#generate-fstab)
 - [**Chroot**](#chroot)
   - [Date & Time](#set-time--date)
   - [Language](#set-language)
   - [Hostname & Hosts](#set-hostname)
   - [Network Manager](#install--enable-networkmanager)
   - [ROOT Password](#set-root-password)
   - [Installing the Bootloader](#installing-bootloader)
     - [GRUB](#grub)
     - [SystemD-Boot](#systemd-boot)
     - [Making and editing config files (UEFI ONLY)](#making-config-file)
 - [**Boot Freshly Installed System**](#now-boot-into-your-freshly-installed-arch-system)
   - [Connect to the internet (again)](#reconnect-to-the-internet)
   - [Add User](#add-new-user)
   - [Sudo Command](#allow-wheel-group-to-use-sudo-commands)
 - [**User Login**](#reconnect-to-the-internet)
   - [Display Server & GPU Drivers](#xorg--gpu-drivers)
   - [Multilib Repository (32bit)](#multilib)
   - [Display Manager (SDDM)](#install--enable-sddm)
   - [Desktop Environment (KDE Plasma)](#kde-plasma--applications)
   - [Audio Utilities & Bluetooth](#audio-utilities--bluetooth)
   - [Misc Applications](https://#my-required-applications)
 - [**The Conclusion**](#the-conclusion)
 - [**Extras (optional)**](#extras)
   - [Yay](#install-yay)
   - [Alternative Shells](#alternative-shells)
   - [Change SHELL](#changing-your-shell)
   - [PipeWire](#pipewire)
   - [EasyEffects](#easyeffects)
   - [Clam AntiVirus](#clamav)
 - [**Theming & Customisations**](#theming--customisations)
    - [Oh My Zsh & Powerlevel10k Theme](#install-oh-my-zsh)
    - [Kvantum Manager](#kvantum-manager)
 - [**Maintenance, Performance Tuning & Monitoring**](maintenance-performance-tuning--monitoring)
   - [Paccache](#paccache)
   - [Cockpit](#install-cockpit)
 - [**Changelog**](#changelog)
</br>

## Let's begin! <a name="lets-begin"></a>

- Grab the latest built ISO Image from **[Arch Linux Download](https://www.archlinux.org/download/)** and write it to an empty USB Stick.
- After the image is done writing, restart your computer and hold one of the following keys: Del, F12, F9, F7, Option
- Your computer will then prompt you to select a bootable device
- Select the bootable USB stick and your computer should show a range of options
- Select "Arch Linux Install medium" and wait to be booted into the ArchISO

If your computer doesn't recognise the USB stick or throws an error when trying to boot into it, you likely has Secure Boot on.\
Go into your BIOS settings and disable Secure Boot.

> Tip: Hit CTRL+L to quickly clear the screen

## Connect to the internet <a name="connect-to-the-internet"></a>
Firstly, use the command:
```
iwctl
```

To see which networks stations you have installed, use the command:
```
device list
```

Select a station from the ones listed and power it on by using the command:
```
device [selected station] set-property Powered on
```

Use the command above to turn on its corresponding adapter, only replacing "device" with "adapter"
Then, you may either scan for networks or connect through WPS.

### WPS

Use the following command:
```
wsc [selected station] push-button
```

And push the WPS button at the back of your router. This may take a minute or two to complete.
Once the WPS LED stops flashing, your computer has been connected to the internet!

### Regular method

Use the following command to scan for all of the access points you can currently connect to:
```
station [selected station] scan
```

Then, to display the networks, use the following command:
```
station [selected station] get-networks
```

Select an access point from the list provided and connect to it by using the following command:
```
station [selected station] connect [SSID]
```

IWCTL will prompt you to enter the access point's passphrase. Enter it and you should be connected to the internet soon after.

### Load Keymaps (for non US-ENG Keyboard Users only)
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
ping -c 4 archlinux.org
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

### Disk Partitioning
We are going to make two partitions on our HDD, `EFI BOOT & ROOT` using `gdisk`.
- **IMPORTANT**:  Do not make a `/boot` partition if you are installing on an MBR system
- If you have a brand new HDD or if no partition table is found, then create GPT Partition Table by pressing `g`.
```
gdisk /dev/[disk name] # If you are on an EFI system
fdisk /dev/[disk name] # If you are on an MBR system
```
- [disk name] = device to partition, find yours by running `lsblk`, this shows all the mountpoints and partitions of a disk.
- We will be using separate partitions for our `/`, `/boot`, `/swap` & `/home`.
- Firstly, we will initialise the disk by using the commands below:

If you are on an EFI system:
```
x - Expert command
z - "Zap" the disk
y - Blank our MBR (Fully initialises the disk)
```

If you are on an MBR system:
```
q - To quit
sfdisk --delete /dev/[disk name]
```
Then, run gdisk or fdisk again.

```
n = New Partition
simply press enter = 1st Partition
simply press enter = As First Sector
+1G = As Last sector (BOOT Partition Size)
ef00 = EFI Partition Type

n = New Partition
simply press enter = 2nd Partition
simply press enter = As First Sector
+16G = As Last sector (SWAP size, or double your RAM, whichever is smaller)
8200 = Linux Swap

n = New Partition
simply press enter = 3rd Partition
simply press enter = As First Sector
+40G = As Last sector [ROOT Partition Size (you may use 20GiB if you have a small hard drive)]
8300 or simply press enter = Linux filesystem

n = New Parition
simply press enter = 4th Partition
simply press enter = As first sector
simply press enter = As last sector [HOME parition size (takes up remaining hard drive space)]
8300 or simply press enter = Linux filesystem

w = write & exit
```

It is ABSOLUTELY recommended to make a home partitition, for both security and convenience if you do decide to distro-hop\
**IMPORTANT**: From now on, your disk will be referred to as sdx, with x being the letter representing your drive.

### Format Partitions
```
mkfs.fat -F32 /dev/sdx1
mkswap /dev/sdx2
mkfs.btrfs /dev/sdx3 # Add -f if your system tells you another filesystem like ext4 is already present
mkfs.btrfs /dev/sdx4
```

Turn on swap memory
```
swapon /dev/sdx2
```

### Mount Remaining Partitions
```
mount /dev/sdx3 /mnt 
mount --mkdir /dev/sdx1 /mnt/boot/efi
mount --mkdir /dev/sdx4 /mnt/home
```

## Base System Installation

### Update Mirrors using Reflector (optional but recommended for faster download speeds, slow download speeds can time out) <a name="update-mirrors-using-reflector"></a>
```
reflector -c County1 -c Country2 -a 12 -p https --sort rate --save /etc/pacman.d/mirrorlist
```
Replace `Country1` & `Country2` with countries near to you or with the one you're living in. Refer to **[Reflector](https://wiki.archlinux.org/index.php/reflector)** for more info.

Example:
```
reflector -c 'United States' -a 12 -p https --sort rate --save /etc/pacman.d/mirrorlist
```

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

### Set Time & Date
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```
Replace `Region` & `City` according to your Time zone. To see what timezones are available, use the following commands:
```
ls /usr/share/zoneinfo/
```
and
```
ls /usr/share/zoneinfo/[Region]
```
An example of this would be:
```
/usr/share/zoneinfo/Europe/London
```

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

## Installling the bootloader <a name="installing-bootloader"></a>

The bootloader is what manages the boot process, and is the PID 0 of your Arch system.\
For MBR systems, we will install GRUB and for UEFI system, we will install SystemD-Boot

### Installing GRUB (MBR) <a name="grub"></a>

"Targets" are CPU architechtures. These are important for grub to know so it can handle the boot proess correctly.\
Find your CPU architechture from [this site](https://renenyffenegger.ch/notes/Linux/shell/commands/grub-install#grub-install-target) and specify that as the target

```
pacman -S grub
grub-install /dev/[disk name] # You don't need to specify a target because the default is i386-pc
grub-mkconfig -o /boot/grub/grub.cfg
```

### Install EFI Boot manager and SystemD-Boot (UEFI) <a name="systemd-boot"></a>
```
pacman -S efibootmgr
bootctl --path=/boot/efi install
```

#### Creating config files <a name="making-config-file"></a>

Open and edit /boot/loader/loader.conf
```
nano /boot/efi/loader/loader.conf
```
Comment out the line beginning with ```default``` by putting a hashtag at the beginning of the line.

And add this line to the bottom of the file
```
default arch-*
```

Once that's done, type:
```
nano /boot/efi/loader/entries/arch.conf
```

And define parameters as follows:
```
title    Arch Linux
linux    /vmlinuz-linux
initrd   /initramfs-linux.img
options root /dev/sdx3 rw
```

(Keeping in mind that sdx refers to the drive you want to install Arch Linux onto)

Save by hitting Ctrl+O, Enter, then Ctrl+X.

**:warning: - Did you follow the above steps? That section is MANDATORY** 

### Final Step
```
exit
umount -a
reboot
```
</br>

## Now boot into your freshly installed Arch system

### Login as "root" and enter root password when prompted

### Add new User
```
useradd -mG wheel [username]
```
Replace `[username]` with your username of choice.

### Set User Password
```
passwd [username]
```

Repeat the above process as many times as you want, depending on the amount of users you want to add to your system.\
If you do not want a user to use sudo commands, use the below command instead:

```
useradd -m [username]
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

### Logout of "root"
```
exit
```

## Login as USER and let's connect to the internet again! <a name="reconnect-to-the-internet"></a>

Since we're now using NetworkManager instead of iwd, our connection settings have been lost (and the connection process is slightly different)\
Firstly, to take a look at what network stations you have installed on your computer, use the command:
```
nmcli device
```
Then, we turn on wifi by using the command:
```
nmcli radio wifi on
```
And we list local access points by using the command:
```
nmcli device wifi list
```
Select one of the access points listed and connect to it by running the following command:
```
nmcli device wifi connect [Access Point SSID] password [Access Point Password]
```

You don't need to check for updates as Arch will have already downloaded the latest version of Arch Linux

You can stop here if you want to do a server installation or have a desktop-less Arch system for any other reason.

### Xorg & GPU Drivers
```
sudo pacman -S xorg [xf86-video-your gpu type]
```
- For Nvidia GPUs, type `nvidia` & `nvidia-settings`. For more info/old GPUs, refer to [Arch Wiki - Nvidia](https://wiki.archlinux.org/index.php/NVIDIA).
- For newer AMD GPUs, type `xf86-video-amdgpu`.
- For legacy Radeon GPUs like HD 7xxx Series & below, type `xf86-video-ati`.
- For dedicated Intel Graphics, type `xf86-video-intel`.

### Enable Multilib Repo (optional but absolutely recommended) <a name="multilib"></a>
multilib contains 32-bit software and libraries that can be used to run and build 32-bit applications on 64-bit installs (e.g. [Wine](https://www.winehq.org/), [Steam](https://store.steampowered.com/), etc).

Edit `/etc/pacman.conf` & uncomment the below two lines.
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

#### MESA Libraries (32bit) (optional but highly recommmended)
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
Packages         | Description
---------------- | ------------------------------------
plasma           | KDE Plasma Desktop Environment.
konsole          | KDE Terminal.
dolphin          | KDE File Manager.
ark              | Archiving Tool.
kwrite           | Text Editor.
kcalc            | Scientific Calculator.
spectacle        | KDE screenshot capture utility.
krunner          | KDE Quick drop-down desktop search.
partitionmanager | KDE Disk & Partion Manager.

### Audio Utilities & Bluetooth (optional but recommended) <a name="audio-utilities--bluetooth"></a>
```
sudo pacman -S alsa-utils bluez bluez-utils
```
Packages    | Description
----------- | -----------------------------------------
alsa-utils  | This contains (among other utilities) the `alsamixer` and `amixer` utilities.
bluez       | Provides the Bluetooth protocol stack.
bluez-utils | Provides the `bluetoothctl` utility.

#### Enable Bluetooth Service
```
sudo systemctl enable bluetooth.service
```


### Apps I would personally recommend installing but aren't required
You can install all the following packages or only the one you want.
```
sudo pacman -S firefox openssh qbittorrent audacious wget screen git neofetch
```
Packages | Description
--------- | ----------
firefox | Mozilla Firefox Web Browser.
openssh | Secure Shell access server.
qbittorrent | Qt-based BitTorrent Client.
audacious | Qt-based music player.
wget\* | Wget is a free utility for non-interactive download of files from the Web. 
screen | Is a full-screen window manager that multiplexes a physical terminal between several processes, typically interactive shells.
git\*| Github command-line utility tools. (needed to access the AUR) 
fastfetch | Fastfetch is a command-line system information tool, that is the sucessor to NeoFetch.
cups\*| Printer service

> \* - These are some of the more important packages, which a lot of programs tend to use. They're optional but it is highly recommended to install both of them.

### Enable OpenSSH daemon and CUPS printer service
```
sudo systemctl enable sshd.service
sudo systemctl enable --now cups.service
```

### Final Reboot
```
reboot
```
</br>

### The Conclusion <a name="the-conclusion"></a>
Now everything is installed and after the final `reboot`, you will land in the SDDM greeter. You can continue reading for some steps to further improve your experience. I recommend you to install `yay` & `paccache`.
- Yay will provide your packages from AUR (Arch User Repository), which are not available in the official repos.
- Paccache can be used clean pacman cached packages either manually or in an automated way.
</br>

## Extras (optional but worth a read) <a name="extras"></a>

### Install [Yay](https://github.com/Jguer/yay)
Yet Another Yogurt - An AUR Helper.
A lot of programs written for Arch can be founded in the AUR, but be careful of what you download from there.
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Install [NuShell](https://www.nushell.sh) <a name="alternative-shells"></a>
NuShell is a powerful shell that has really helpful debug statements and is overall a solid shell environment.
```
yay -S nushell
```

### Install [Zsh](https://wiki.archlinux.org/index.php/zsh/)
Zsh is a powerful shell that operates as both an interactive shell and as a scripting language interpreter. It's a preferred shell environment by many.
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

### Set NuShell or Zsh as our SHELL
For an example, we will set Zsh as default SHELL which we installed in the last step:
```
chsh -s /usr/bin/zsh # To set Zsh as the default SHELL
chsh -s /usr/bin/nu # To set NuShell as the default SHELL
```
For the changes to apply, you will have Logout and Log back in or better do `reboot`.

## PipeWire
[PipeWire](https://wiki.archlinux.org/title/PipeWire) is a new low-level multimedia framework. It aims to offer capture and playback for both audio and video with minimal latency and support for PulseAudio, JACK, ALSA and GStreamer-based applications.
#### Install
```
sudo pacman -S pipewire wireplumber pipewire-audio pipewire-alsa pipewire-pulse
```

## EasyEffects
[EasyEffects](https://wiki.archlinux.org/title/PipeWire#EasyEffects) (former PulseEffects) is a GTK utility which provides a large array of audio effects and filters to individual application output streams and microphone input streams. Notable effects include an input/output equalizer, output loudness equalization and bass enhancement, and input de-esser and noise reduction plug-in.
Install
```
sudo pacman -S easyeffects
# or
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
sudo pacman -S kvantum
```

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

 - **2024-06-25**
   - A lot of changes, improvements, and updates. Special thanks to our biggest contributor @[Exvix](https://github.com/Exvix).

#### Full and complete changelog, [click here](https://github.com/XxAcielxX/arch-plasma-install/blob/main/CHANGELOG.md).
