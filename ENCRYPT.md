#laptopsetup

## Pre Chroot

### Syncronise clock
```bash
timedatectl set-ntp true
```

### Prepping the disk

Scan for available hard disks:
```bash
fdisk -l
```

Setup partitions on disk, one for boot the other for our LVM partition:
```bash
gdisk /dev/nvme0n1

# delete partition
d
# select the number of each partition separately
# "1", "2", "3", ...

# create new "boot" partition
n
# take start of first sector @ 2048
# set end of first partition to +512M

# create new "system" partition
n
# take default for start of partition, should be end of previous partition
# take default again for end of partition, should be whole disk

# set partition type
t
# Select 1 for first partition
# type "ef00" to set partition 1 to EFI

# set partition type
t
# select 2 for second partition
# type "8e00" to set partition 2 to LVM

# write our changes
w
```

Setup encryption for disk:
```bash
crytpsetup luksFormat /dev/nvme0n1p2
```
This step will wipe your whole disk and setup an encrypted partition with a password of your choice.

```bash
cryptsetup open /dev/nvme0n1p2 cryptlvm
```

```bash
pvcreate /dev/mapper/cryptlvm
```

`MyVolGroup` in the below command can be anything, as long as you're consistent throughout the guide you should be fine. 
```bash
vgcreate lvm /dev/mapper/cryptlvm
```

Setup all the partitions:
```bash
lvcreate -L 8G lvm -n swap
# lvcreate -L 300G lvm -n home
lvcreate -l 100%FREE lvm -n root
```

Format partitions:
```bash
mkfs.ext4 /dev/lvm/root
# mkfs.ext4 /dev/lvm/home
mkswap /dev/lvm/swap
```

Mount partitions
```bash
mount /dev/lvm/root /mnt
# mount --mkdir /dev/lvm/home /mnt/home

swapon /dev/lvm/swap
```

Create boot partition:
```bash
mkfs.fat -F32 /dev/nvme0n1p1

mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

Go into `/etc/pacman.conf` and change the below setting to improve download speeds:
```bash
# make sure it isn't commented out
ParallelDownloads = 16
```

Install our base system to `/mnt`:
```bash
pacstrap /mnt base linux linux-firmware base-devel lvm2
```
- `base-devel` : group of packages usually used by developers
- `lvm2` : required for decrypting our root partition
- `sof-firmware` : not in the list but possibly required for Lenovo laptops

Generate fstab file:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Jump into our chroot:
```bash
arch-chroot /mnt
```
### In chroot
Set timezone:
```bash
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
```

Sync hardware clock:
```bash
hwclock -uw
```

Setting the locale:
```bash
nano /etc/locale.gen

en_GB.UTF-8
```

```bash
nano /etc/locale.conf

LANG=en_GB.UTF-8
```

Hostname:
```bash
nano /etc/hostname
```

Set root password:
```bash
passwd
```

```bash
useradd -m printer

passwd printer
```

### Modify mkinitcpio
```bash
nano /etc/mkinitcpio.conf

# Find HOOKS() and modify so it contains "encrypt" and "lvm" between "block" and "filesystem"
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

Apply changes to the kernel:
```bash
mkinitcpio -p linux
```
### Boot loader install
```bash
bootctl install
```

Setup bootctl entry:
```bash
echo -e "title\tArch Linux (Crypt LVM)\nlinux\t/vmlinuz-linux\ninitrd\t/initramfs-linux.img\noptions\tcryptdevice=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p2):cryptlvm root=/dev/lvm/root" > /boot/loader/entries/arch-lvm.conf
```

### Install GUI
```bash
pacman -S xorg-server plasma-meta konsole dolphin flatpak
```
#### Issues with outdated certificates / gpg / pgp issues
```bash
# try to update archlinux-keyring
pacman -S archlinux-keyring

# if that doesn't work try the following commands on top of that
pacman-key --init
pacman-key --populate
pacman -S <your_package>
```

Enable GUI services to start on boot
```bash
systemctl enable sddm NetworkManager
```

```bash
pacman -S sudo

groupadd sudo
usermod -aG sudo myusername
nano /etc/sudoers
# => uncomment users in sudo group can use sudo
```
