# Arch Linux Installation Steps

An opinionated installation guide for Arch Linux.

## 1. Pre-installation

### 1.1. Checklist

Make sure the live system is connected to the internet and has the right system time:

```bash
ip addr
networkctl status
ping archlinux.org
timedatectl status
```

### 1.2. Partitioning

Use `gdisk` to create a **GUID Partition Table (GPT)** and necessary partitions.

```console
# gdisk /dev/sda
    n
    n
    p
    w
```

Then verify the partitions created:

```bash
lsblk -f
```

Format and mount the partitions:

```bash
mkfs.fat -F 32 -n ESP /dev/nvme0n1p1
mkfs.btrfs -L archlinux /dev/nvme0n1p2
mount -o noatime,compress=zstd,ssd,discard /dev/nvme0n1p2 /mnt
mkdir /mnt/efi /mnt/boot
mount -o noatime,discard /dev/nvme0n1p1 /mnt/efi
mount -o noatime,discard /dev/nvme0n1p1 /mnt/boot
```

Create subvolumes for common directories.

```bash
cd /mnt/
btrfs sub create etc
btrfs sub create home
btrfs sub create opt
btrfs sub create root
btrfs sub create srv
btrfs sub create usr
btrfs sub create var
btrfs sub create var/cache
btrfs sub create var/lib
btrfs sub create var/log
```

## 2. Installation

### 2.1. Select Mirrors

Check and clean up `/etc/pacman.d/mirrorlist`:

```bash
nano /etc/pacman.d/mirrorlist
```

### 2.2. Install Bootstrap Packages

Use the `pacstrap` script to install bootstrap packages:

```bash
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs nano htop sudo tmux man-db man-pages texinfo
```

## 3. Configuration

### 3.1. fstab

Generate an `fstab` file (use `-U` or `-L` to define by UUID or labels, respectively):

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors.

### 3.2. chroot

Change root into the newly-installed system to make further changes:

```bash
arch-chroot /mnt
```

### 3.3. Time Zones

Set the time zone:

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run `hwclock(8)` to generate `/etc/adjtime`:

```bash
hwclock --systohc
```

This command assumes the hardware clock is set to UTC. See [Time#Time standard](https://wiki.archlinux.org/index.php/Time#Time_standard) for details.

### 3.4. Localization

Uncomment `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, and generate them with:

```console
# locale-gen
```

Set the `LANG` variable in `locale.conf(5)` accordingly, for example:

`/etc/locale.conf`

```
LANG=en_US.UTF-8
```

### 3.5. Networking

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

### 3.6. Users and permissions

Set the root password:

```console
# passwd
```

Create a new user for yourself:

```console
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

### 3.7. Additional Packages


For desktops and laptops, install a desktop environment and additional packages. Here's a list of my opinionated package selection:

```bash
# TODO: Actually go through my pacman.log to compile this list of packages.
pacman -Syu --needed
```

### 3.8. Boot loader

- Choose [systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot) to boot from the kernel in ESP.
- Choose [GRUB](https://wiki.archlinux.org/index.php/GRUB) to boot from anywhere.
- Choose [Syslinux](https://wiki.archlinux.org/index.php/Syslinux) if you installed to a [partitionless btrfs disk](https://wiki.archlinux.org/index.php/Btrfs#Partitionless_Btrfs_disk).

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

## Post-installation

Configure a firewall. Install your favorite [AUR helper](https://wiki.archlinux.org/index.php/AUR_helpers).
