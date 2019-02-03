# Arch Linux Installation Steps
Installing Arch Linux is way more complex than other Linux distros due to the absence of a GUI installer. However, it has also made Arch Linux more customizable than any other distros. Here's what I did when installing Arch Linux with an archiso on a Hyper-V Gen 2 VM.

## Preparations

archiso only contains a very limited subset of Arch Linux. Thus, installation requires Internet connection to download packages.
Check your IP address with:

```shell
# timedatectl set-ntp true
# timedatectl status
```

## Partitioning

Use `gdisk` to create a **GUID Partition Table (GPT)** and necessary partitions.