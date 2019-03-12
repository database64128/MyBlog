# Gnome Keyring and SSH Keys
_Written on 2019-03-10._

For the past few months, I've been using OpenSSH client on Windows. The OpenSSH client integrated in Git Bash is good enough to manage my Linux VMs and VPSes. But then, there is the need to connect to an OpenSSH server on Linux. I followed the same procedures, making sure access privilages are properly set. But the system keeps asking for user password when an SSH client or Git attempts to use a private key. After a quick search, a simple `ssh-add $privatekey` solved my problem.

## Cause
> [GNOME Keyring](https://wiki.gnome.org/Projects/GnomeKeyring) is "a collection of components in GNOME that store secrets, passwords, keys, certificates and make them available to applications."
On systems using `Gnome`, `Gnome Keyring` is installed as a part of the `gnome` group. Gnome Keyring includes an SSH agent which integrates with the `gnome-keyring` and user login for its passwords. It controls access to SSH private keys.

Private keys with corresponding public keys should be automatically loaded. Otherwise, use `ssh-add` to load the private key.

## Reference
* [Projects/GnomeKeyring/Ssh - GNOME Wiki!](https://wiki.gnome.org/Projects/GnomeKeyring/Ssh)
* [GNOME/Keyring - ArchWiki](https://wiki.archlinux.org/index.php/GNOME/Keyring)