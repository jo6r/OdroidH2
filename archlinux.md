# Archlinux on Odroid H2

Official Installation guide
* https://wiki.archlinux.org/title/Installation_guide

Inspiration
* https://github.com/raven2cz/geek-room/tree/main/arch-install-luks-btrfs

Simple tutorial how to install arch on Odroid H2

## Installation

### Connect to the internet

```shell
ip link
ping archlinux.org
```

### Time Date Settings

```shell
timedatectl set-ntp true
```

### Disk Partitioning

| Number | 	Type             | 	Size                       |
|--------|-------------------|-----------------------------|
| 1      | 	EFI              | 	512 Mb                     |
| 2      | 	Linux Filesystem | 	all of the remaining space |

```shell
fdisk -l
fdisk /dev/nvme0n1
```

```
Command (m for help): g
Created a new GPT disklabel (GUID: 89F14EEB-1A94-4A13-B2C4-3040C1667C68).

Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-976773134, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-976773134, default 976773119): +512M

Created a new partition 1 of type 'Linux filesystem' and of size 512 MiB.

---

Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.

---

Command (m for help): n
Partition number (2-128, default 2):
First sector (1050624-976773134, default 1050624):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-976773134, default 976773119):

Created a new partition 2 of type 'Linux filesystem' and of size 465.3 GiB.

---

Command (m for help): p
Disk /dev/nvme0n1: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: CT500P1SSD8
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 89F14EEB-1A94-4A13-B2C4-3040C1667C68

Device           Start       End   Sectors   Size Type
/dev/nvme0n1p1    2048   1050623   1048576   512M EFI System
/dev/nvme0n1p2 1050624 976773119 975722496 465.3G Linux filesystem

---

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### Disk formatting

```shell
mkfs.fat -F 32 /dev/nvme0n1p1
mkfs.btrfs /dev/nvme0n1p2
```

### Disk mounting

```shell
mount /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@snapshots

btrfs subvolume list /mnt

umount /mnt

mount -o compress=zstd,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/home
mount -o compress=zstd,subvol=@home /dev/nvme0n1p2 /mnt/home
mkdir -p /mnt/var/cache
mount -o compress=zstd,subvol=@cache /dev/nvme0n1p2 /mnt/var/cache
mkdir -p /mnt/var/log
mount -o compress=zstd,subvol=@log /dev/nvme0n1p2 /mnt/var/log
mkdir -p /mnt/tmp
mount -o compress=zstd,subvol=@tmp /dev/nvme0n1p2 /mnt/tmp
mkdir -p /mnt/snapshots
mount -o compress=zstd,subvol=@snapshots /dev/nvme0n1p2 /mnt/snapshots

lsblk


# efi
mkdir -p /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

### Packages installation

```shell
pacstrap -K /mnt base linux-lts linux-firmware intel-ucode dhcpcd git btrfs-progs grub efibootmgr openssh sudo less nano htop wget rsync util-linux cronie lm_sensors
```

### Fstab

```shell
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

### Chroot

```shell
arch-chroot /mnt
```

### Set up the time zone

```shell
ln -sf /usr/share/zoneinfo/Europe/Prague /etc/localtime
hwclock --systohc
```

### Localization

```shell
# uncomment en_US.UTF-8 UTF-8
nano /etc/locale.gen
locale-gen
```

### Network configuration

```shell
nano /etc/hostname
```

```
odroid
```

```shell
nano /etc/hosts
```

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 odroid
```

### Root and users

```shell
# Set up the root password
passwd
# add user
useradd -m -G wheel jo6r
passwd jo6r

EDITOR=nano visudo
# uncomment %wheel ALL=(ALL:ALL) ALL
```

### Services

```shell
systemctl enable dhcpcd
systemctl enable cronie
systemctl enable sshd
systemctl enable fstrim.timer
```

### GRUB

```shell
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Unmount everything and reboot

```shell
# Exit from chroot
exit
umount -R /mnt
reboot
```

## Post-installation

### SSH

```shell
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
rm ~/.ssh/id_ed25519.pub
rm ~/.ssh/id_ed25519
chmod 600 ~/.ssh/authorized_keys
```

```shell
nano /etc/ssh/sshd_config

Port 22222
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
```

### How To Enable Wake On LAN

https://wiki.odroid.com/odroid-h2/application_note/wake_on_lan

### Nano as default editor

https://wiki.archlinux.org/title/Nano#Replacing_vi_with_nano

```shell
nano ~/.bash_profile
```

```
export VISUAL=nano
export EDITOR=nano
```

### Another disks

```shell
mkdir -p /media/data /media/k3s
chmod a=rwx /media/data /media/k3s
lsblk -f
sudo nano /etc/fstab
```

```
# /dev/sda1 Samsung SSD 850 / 232.89 GiB
UUID=7b66f327-13ed-4084-a2dd-76f8290d378e       /media/k3s   ext4    defaults,discard        0 2

# /dev/sdb1  Samsung SSD 870 / 1.82 TiB
UUID=cc76f943-04cb-400f-953c-c0c15327dbfe       /media/data     ext4     defaults,discard        0 2
```

### Sensors

https://wiki.archlinux.org/title/lm_sensors

```shell
sudo sensors-detect
```