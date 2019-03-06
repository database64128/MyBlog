# Some Notes About Fedora Workstation

*Updated 2019-03-05: Added `firewalld` for Fedora Workstation 29.*

Some notes regarding day-to-day usage of Fedora Workstation.

## 1. SELinux

As of 2018-11-18, on Fedora Workstation 29, `SELinux` is enabled and running in enforced mode. This will cause issue if you change the listening port of `sshd.service`, resulting in failure of starting the service.

Ironically, `SELinux` is intended to enhance the security level of Linux. Instead of making Linux safer, it prevents normal security procedures from taking place.

If you like how `SELinux` behaves, there is a workaround by adding an allowed port in `SELinux`'s policy.

```shell
# semanage port –a –t ssh_port_t –p tcp 10022
```
`-a, --add = Add a OBJECT record NAME`
`-t, --type = SELinux Type for the object`
`-p, --proto = Protocol for the specified port (tcp|udp).`

If you find SELinux quite annoying and useless, just like me, disabling SELinux is not much of a hustle.

Configure `SELINUX=disabled` in `/etc/selinux/config`:
```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#       targeted - Targeted processes are protected,
#       mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
Reboot your system. After the reboot, confirm that the `getenforce` command returns `Disabled`:
```shell
# getenforce
Disabled
```
The status of `SELinux` can also be checked with `sestatus`. You might get something like:
```shell
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             refpolicy
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              disabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

## 2. `yum` is depreciated
`yum` is depreciated. Though `yum` is still installed by default on Fedora Workstation 29, some of its functions may not work. Instead always use `dnf`. See `man yum2dnf` for more details.
## 3. `firewalld` uses `nftables` as the default backend
Starting from Fedora Workstation 29, with the release of `firewalld 0.6.0`, `firewalld` uses the new `nftables` as the new default backend. This change will come with CentOS 8 too.

Refer to [Configuring `firewalld`](Configuring_firewalld.md) for more information on how to configure `firewalld`.