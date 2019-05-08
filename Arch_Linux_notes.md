# Arch Linux Notes

*First edition 2019-05-06.*

## 1. AUR
`makepkg --asroot` has been dropped a few years ago. This means you can't build as root. It's recommended to build as a normal user. If you don't have any other user, there is a method to build as `nobody`.

1. Ensure the `base-devel` package group is installed in full (`pacman -S --needed base-devel`).
2. Get `PKGBUILD` files: `git clone $AUR_Link` and verify them.
3. If you have a normal user, simply run `makepkg -si`. Otherwise, install needed dependencies, and do this:
```shell
mkdir /home/build
chgrp nobody /home/build
chmod g+ws /home/build
setfacl -m u::rwx,g::rwx /home/build
setfacl -d --set u::rwx,g::rwx,o::- /home/build
sudo -u nobody makepkg --skipinteg
pacman -U $package_name.tar.xz
```
Use `--skipinteg` to skip PGP key verification.

### References
* [Arch User Repository - ArchWiki](https://wiki.archlinux.org/index.php/Arch_User_Repository)
* [Replacing “makepkg –asroot” | Allan McRae](http://allanmcrae.com/2015/01/replacing-makepkg-asroot/)