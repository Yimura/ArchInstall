# Arch Install

## Table of contents

## Pre-chroot

Set the keyboard layout for this session:
```sh
loadkeys be-latin1
```

Update system clock:
```sh
timedatectl set-ntp true
```

Setup partitioning:
```sh
fdisk -l

fdisk /dev/nvme0n1

# the following commands are ran inside fdisk
# delete existing partitions with
d
=> 1,2,3,...

# create new partition for EFI of 1GiB
n
=> default = 1
=> default
=> 1G

# create a swap partition
n
=> default = 2
=> default
=> 32G # take the side of your RAM or 

# partition for our Linux system
n
=> default = 3
=> default
=> default

# Set the type of our first partition to EFI
t
=> 1

# Write changes
w
```

Format Partitions:
```bash
mkfs.ext /dev/nvme0n1p3

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
vim /etc/pacman.conf

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

Localization:
```bash
vim /etc/locale.gen


# uncomment accordingly
en_US.UTF-8
nl_BE.UTF-8
```

```bash
vim /etc/locale.conf

LANG=en_US.UTF-8
```

Set tty console default:
```bash
vim /etc/vconsole.conf

KEYMAP=be-latin1
```

Set hostname:
```bash
vim /etc/hostname

sapphire
```

Set our root password:
```
passwd
```

