# Arch Linux Installation Steps

*First edition 2018-09-06.*

*Updated 2019-03-06: Migrated to markdown and changed some wording.*

Installing Arch Linux is way more complex than other Linux distros due to the absence of a GUI installer. However, it has also made Arch Linux more customizable than any other distros. Here's what I did when installing Arch Linux with an `archiso` on a Hyper-V Gen 2 VM.

## Preparations

`archiso` only contains a very limited subset of Arch Linux. Thus, installation requires Internet connection to download packages. Check your network first:
```bash
# ip addr
# ping archlinux.org
```

Use `timedatectl(1)` to ensure the system clock is accurate:
```bash
# timedatectl set-ntp true
# timedatectl status
```

## Partitioning

Use `gdisk` to create a **GUID Partition Table (GPT)** and necessary partitions.
```bash
# gdisk /dev/sda
    o
    n
    n
    n
    w
```
Then verify the partitions created:
```bash
# lsblk -f
```
Format the partitions. For SSDs and VHDs, enable `discard` with `-d` option.
```bash
# mkswap /dev/sda3
# swapon -d /dev/sda3
# mkfs.fat -F 32 /dev/sda1
# mkfs.btrfs /dev/sda2
# lsblk -f
```

## Mount the partitions
Note that for VHDs and SSDs, it is recommended to mount with discard option.
```bash
# mount -o discard /dev/sda2 /mnt
# mkdir /mnt/efi
# mount -o discard /dev/sda1 /mnt/efi
```
## Select mirrors for `pacman`
Back up the existing `/etc/pacman.d/mirrorlist`:
```bash
# cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```
Then get a mirror list from the official [Pacman Mirrorlist Generator](https://www.archlinux.org/mirrorlist/).

## Install the `base` package group
Use the `pacstrap` script to install the `base` package group:
```bash
# pacstrap /mnt /base
```
This group does not include all tools from the live installation, such as `btrfs-progs` or specific wireless firmware.

## Configurations

### fstab
Generate an `fstab` file (use `-U` or `-L` to define by UUID or labels, respectively):
```bash
# genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors. **You may want to add discard options for each partition.**

### chroot
Change root into the newly-installed system to make further changes:
```bash
# arch-chroot /mnt
```

### Time Zones
Set the time zone:
```bash
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
Run `hwclock(8)` to generate `/etc/adjtime`:
```bash
# hwclock --systohc
```
This command assumes the hardware clock is set to UTC. See [Time#Time standard](https://wiki.archlinux.org/index.php/Time#Time_standard) for details.

### Localization
Uncomment `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, and generate them with:
```bash
# locale-gen
```
Set the `LANG` variable in `locale.conf(5)` accordingly, for example:

`/etc/locale.conf`
```bash
LANG=en_US.UTF-8
```

### Networking
Create the `hostname` file:

`/etc/hostname`
```bash
VM-Arch-Desktop
```
Add matching entries to `hosts(5)`:

`/etc/hosts`
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	VM-Arch-Desktop.localdomain	VM-Arch-Desktop
```
If the system has a permanent IP address, it should be used instead of `127.0.1.1`.
Complete the [network configuration](https://wiki.archlinux.org/index.php/Network_configuration) for the newly installed environment.

### Root Password
Set the root password:
```bash
passwd
```

### Boot loader
A Linux-capable boot loader must be installed in order to boot Arch Linux. See [Category:Boot loaders](https://wiki.archlinux.org/index.php/Category:Boot_loaders) for available choices. Here we install `GRUB` and make a configuration file.
```bash
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```
We have finally finished the installation. Exit the chroot environment and reboot the system.
```bash
# exit
# reboot
```

## Post Installation

Enable DHCP client first to connect to the Internet:
```bash
# systemctl enable –-now dhcpcd
```
Install btrfs utilities:
```bash
# pacman -S btrfs-progs
```
