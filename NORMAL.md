# Arch Install

## Table of contents

## Pre-chroot

Update system clock:
```sh
timedatectl set-ntp true
```

Setup partitioning:
```sh
fdisk -l
```
Look for the model of disk you want to install the OS on
Write the the disk down. Common names used are /dev/sdX or /dev/nvme0nX. Where X should be replaced with the disk letter

Example used: /dev/nvme0n1. Replace with corresponding disk

```sh
fdisk /dev/nvme0n1

# the following commands are ran inside fdisk
# delete existing partitions with, run the d command until you have no partitions left
d
=> 1,2,3,...

# create new partition for EFI
n
=> default = 1
=> default
# EFI partition can be between 500M and 1 G
=> +800M

# create a swap partition
n
=> default = 2
=> default
=> +32G # take the size of your RAM or half of it

# partition for our Linux system
n
=> default = 3
=> default
=> default

# Set the type of our first partition to EFI
t
=> 1

# Write changes  --IMPORTANT--
w
```

assuming your disk name is /dev/nvme0n1
Format Partitions:
```bash
mkfs.ext4 /dev/nvme0n1p3

mkswap /dev/nvme0n1p2

mkfs.fat -F 32 /dev/nvme0n1p1
```

Mount partitions
```bash
mount /dev/nvme0n1p3 /mnt

mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

swapon /dev/nvme0n1p2
```

Before we can go onto installing our system we'll enable some things that'll make our downloads faster.
```bash
nano /etc/pacman.conf

...
ParallelDownloads 20
...
```

Install base system and kernel
```bash
pacstrap /mnt base linux linux-firmware base-devel
```

Generate fstab for our mounted filesystems
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Enter our very basic install:
```bash
arch-chroot /mnt
```

## Inside our chroot

Set our timezone
```bash
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
```

Sync hardware clock
```bash
hwclock --systohc
```

### Setting Locale

Localization:
```bash
nano /etc/locale.gen

# uncomment accordingly
en_US.UTF-8
nl_BE.UTF-8
```

```bash
nano /etc/locale.conf

LANG=en_US.UTF-8
```

### Hostname

Set hostname:
```bash
nano /etc/hostname

sapphire
```

### Setup Users
Set our root password:
```
passwd
```

Create a normal user DO NOT FORGET -m :
```bash
useradd -m username
```

Set the password for our new user
```bash
passwd username
```

### Installing boot loader

## `OPTION 1: REFIND (NOT RECOMMENDED)`
```bash
pacman -S refind
```

```bash
refind-install
```

Install micro-code patches for your CPU:
```bash
pacman -S amd-ucode
# or
pacman -S intel-ucode
```

Log fstab:
```bash
cat /etc/fstab

=>

# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/nvme0n1p3
UUID=a9d51ea5-3fab-4c85-9e35-18dfe25fad02       /               ext4            rw,relatime     0 1

# /dev/nvme0n1p1
UUID=7A00-2C23          /boot           vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro   0 2

# /dev/nvme0n1p2
UUID=98850be0-6397-408a-8a16-cbae126a9486       none            swap            defaults        0 0
```

Take note of the partition with `UUID=a9d51ea5-3fab-4c85-9e35-18dfe25fad02`, this UUID will be different for your system but make sure you take note of this as we'll need it for our refind_linux.conf.

#### Fixing Refind

```bash
nano /boot/refind_linux.conf
```

Make sure what you find in this file looks about the same as the below content, replace the UUID with the one you took note of previously.
```text
"Boot with standard options"  "root=UUID=a9d51ea5-3fab-4c85-9e35-18dfe25fad02 rw initrd=amd-ucode.img initrd=initramfs-linux.img"
"Boot to single-user mode"    "root=UUID=a9d51ea5-3fab-4c85-9e35-18dfe25fad02 single"
"Boot with minimal options"   "ro root=/dev/nvme0n1p3"
```
Also replace `amd-ucode` with `intel-ucode` if you have an intel CPU.

## `OPTION 2: bootctl (RECOMMENDED)`

```bash
bootctl install
```

### Installing KDE plasma
`important` use plasma-desktop if you want a basic set of utilities like text editor, calculator, file explorer,....
```bash
pacman -S xorg-server plasma-meta
```

Enabling SDDM and NetworkManager (not sure if required but it doesn't hurt to do it anyways)
```bash
systemctl enable sddm NetworkManager
```

### Install a Terminal and file explorer
you can use others, we use these and they work great
flatpak is for software installation using discover
```bash
pacman -S konsole dolphin flatpak
```

### Setup Sudo

```bash
pacman -S sudo

groupadd sudo

usermod -aG sudo username

nano /etc/sudoers
# => uncomment users in sudo group can use sudo
```

### Exit Chroot and Reboot

```bash
exit

reboot
```
