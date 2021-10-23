# Arch Linux Installation Steps

*First edition 2018-09-06.*

*Updated 2019-03-06: Migrated to markdown and other minor edits.*

*Updated 2021-01-26: Keep it up-to-date.*

*Updated 2021-10-23: btrfs subvolume creation and user password.*

An opinionated installation guide for Arch Linux.

## Preparations

Make sure you are connected to the internet.

```console
# ip addr
# networkctl status
# ping archlinux.org
```

Use `timedatectl(1)` to ensure the system clock is accurate:
```console
# timedatectl set-ntp true
# timedatectl status
```

## Partitioning

Use `gdisk` to create a **GUID Partition Table (GPT)** and necessary partitions.

```console
# gdisk /dev/sda
    n
    n
    p
    w
```

Then verify the partitions created:

```console
# lsblk -f
```

Format the partitions. For SSDs and VHDs, enable `discard` with `-d` option.

```console
# mkfs.fat -F 32 -n ESP /dev/nvme0n1p1
# mkfs.btrfs -L "Arch Linux" /dev/nvme0n1p2
# lsblk -f
```

## Mount the partitions

Note that for VHDs and SSDs, it is recommended to mount with discard option.

```console
# mount -o noatime,compress=zstd,ssd,discard /dev/nvme0n1p2 /mnt
# mkdir /mnt/efi /mnt/boot
# mount -o noatime,discard /dev/nvme0n1p1 /mnt/efi
# mount -o noatime,discard /dev/nvme0n1p1 /mnt/boot
```

## Create btrfs subvolumes

Create subvolumes for common directories.

```console
# cd /mnt/
# btrfs sub create etc
# btrfs sub create home
# btrfs sub create opt
# btrfs sub create root
# btrfs sub create srv
# btrfs sub create usr
# btrfs sub create var
# btrfs sub create var/cache
# btrfs sub create var/lib
# btrfs sub create var/log
```

## Select mirrors for `pacman`

Back up the existing `/etc/pacman.d/mirrorlist`:

```console
# mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

Then get a mirror list from the official [Pacman Mirrorlist Generator](https://www.archlinux.org/mirrorlist/).

## Install the `base` package group

Use the `pacstrap` script to install necessary packages:

```console
# pacstrap /mnt base base-devel linux linux-firmware btrfs-progs nano htop sudo tmux man-db man-pages texinfo
```

For desktops and laptops, install a desktop environment.

## Configurations

### fstab

Generate an `fstab` file (use `-U` or `-L` to define by UUID or labels, respectively):

```console
# genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors. **You may want to add discard options for each partition.**

### chroot

Change root into the newly-installed system to make further changes:
```console
# arch-chroot /mnt
```

### Time Zones
Set the time zone:

```console
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run `hwclock(8)` to generate `/etc/adjtime`:

```console
# hwclock --systohc
```

This command assumes the hardware clock is set to UTC. See [Time#Time standard](https://wiki.archlinux.org/index.php/Time#Time_standard) for details.

### Localization

Uncomment `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, and generate them with:

```console
# locale-gen
```

Set the `LANG` variable in `locale.conf(5)` accordingly, for example:

`/etc/locale.conf`
```console
LANG=en_US.UTF-8
```

### Networking

Create the `hostname` file:

`/etc/hostname`
```console
desktop.domain.name
```

Add matching entries to `hosts(5)`:

`/etc/hosts`
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 desktop.domain.name.localdomain desktop.domain.name
```

For desktops and laptops, enable `NetworkManager`. For headless servers, enable `systemd-networkd`. Enable `systemd-resolved`.

```console
# systemctl enable NetworkManager systemd-resolved
```

If the system has a permanent IP address, it should be used instead of `127.0.1.1`.
Complete the [network configuration](https://wiki.archlinux.org/index.php/Network_configuration) for the newly installed environment.

### Users and permissions

Set the root password:

```console
# passwd
```

Create a new user for yourself:

```console
# touch /etc/subuid /etc/subgid
# useradd -mc "Full Name" -G wheel -s /usr/bin/fish <username>
# passwd <username>
```

You might want to take the chance to convert your home directory to a btrfs subvolume.

Configure `/etc/sudoers` to allow users in the `wheel` group to use `sudo`:

```console
# EDITOR=nano visudo
```

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

### Boot loader

Choose [systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot) to boot from the kernel in ESP. Choose [GRUB](https://wiki.archlinux.org/index.php/GRUB) to boot from anywhere. Choose [Syslinux](https://wiki.archlinux.org/index.php/Syslinux) if you installed to a [partitionless btrfs disk](https://wiki.archlinux.org/index.php/Btrfs#Partitionless_Btrfs_disk).

To install `GRUB`:

```console
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

Exit the chroot environment and reboot.

```console
# exit
# reboot
```

## Post Installation

Configure a firewall. Install your favorite [AUR helper](https://wiki.archlinux.org/index.php/AUR_helpers).
