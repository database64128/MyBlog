# SMB in Linux

*First edition 2019-05-04.*

*2020-01-23: added permissions in global settings.*

## SMB server - `samba`
Install the `samba` package. `samba` doesn't come with a configuration file. You can get an example from [here](https://git.samba.org/samba.git/?p=samba.git;a=blob_plain;f=examples/smb.conf.default;hb=HEAD). Then modify entries as you like. Add your SMB share.

1. The default configuration sets `log file` to a non-writable location, which will cause errors. Use `logging = systemd` instead.
2. If required; the `workgroup` specified in the `[global]` section has to match the Windows workgroup (default `WORKGROUP`).
3. There are some optional optimizations provided in the following example.
```
[global]
   server min protocol = SMB3
   smb encrypt = desired
   server multi channel support = yes
   use sendfile = yes
   aio read size = 1
   aio write size = 1

   ;inherit owner = unix only ; Inherit ownership of the parent directory for new files and directories
   ;inherit permissions = yes ; Inherit permissions of the parent directory for new files and directories
   create mask = 0644
   directory mask = 2755
   force create mode = 0644
   force directory mode = 2755

[homes]
# my root share - created 2019-05-03
[root]
   comment = root / directory
   path = /
   valid users = root
   public = no
   writable = yes
   printable = no
```

`samba` uses Linux user accounts but with a different set of passwords. Set password for a user:
```shell
# smbpasswd -a samba_user
```

Configure your firewall to allow traffic for `samba`:
```shell
# firewall-cmd --permanent --zone=home --add-service samba samba-dc samba-client
# firewall-cmd --reload
```
Now start the service:
```shell
# systemctl enable --now smb.service nmb.service
```
Now the SMB server should be up and discoverable in your local network.

Note that previously `smbd.service` and `nmbd.servive` has been renamed with 'd' dropped. Don't use `samba.service` as they are completely different services.

## SMB client - `smbclient`
`smbclient` is an FTP-like SMB client. Install and use `smbclient` to query, browse or mount any SMB share.

Note that you can't mount the "root" of an SMB share in Linux. This is because there is no such "root" for an SMB share. The "root" you see is actually the result of list query. See reference 4.

### Manual Mounting
Put your credentials under `/etc/samba/credentials/` like this:
```shell
/etc/samba/credentials/share
-----------------------------
username=myuser
password=mypass
```
Make sure they have proper permissions:
```shell
# chown -hR root:root /etc/samba/credentials
# chmod 700 /etc/samba/credentials
# chmod 600 /etc/samba/credentials/share
```
Then mount it:
```shell
# mount -t cifs //192.168.50.18/Users /mnt/myshare --verbose -o credentials=/etc/samba/credentials/share,workgroup=DESKTOP-BMGO2R3,iocharset=utf8
mount.cifs kernel mount options: ip=192.168.50.18,unc=\\192.168.50.18\Users,user=username,domain=DESKTOP-BMGO2R3,pass=********
```
Note that the `domain` option must be set to the SMB server's name. Otherwise mounting may fail. See reference 3.

To unmount the share, use:
```shell
# umount mount_point
```

### Automatic Mounting
Create a systemd unit and start it. The unit name must match your mount point. Otherwise mounting may fail.

```
/etc/systemd/system/mnt-myshare.mount
--------------------------------------
[Unit]
Description=Mount Share at boot

[Mount]
What=//server/share
Where=/mnt/myshare
Options=x-systemd.automount,credentials=/etc/samba/credentials/share,iocharset=utf8,rw
Type=cifs
TimeoutSec=30
ForceUnmount=true

[Install]
WantedBy=multi-user.target
```

## References
* [Samba - ArchWiki](https://wiki.archlinux.org/index.php/Samba)
* [smb.conf(5) — Arch manual pages](https://jlk.fjfi.cvut.cz/arch/manpages/man/smb.conf.5)
* [cifs, smb - Can't mount (permission denied) or navigate shared folder - Ask Ubuntu](https://askubuntu.com/questions/767422/cifs-smb-cant-mount-permission-denied-or-navigate-shared-folder)
* [How to mount the "root" file system of a Windows Samba Share with cifs? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/189049/how-to-mount-the-root-file-system-of-a-windows-samba-share-with-cifs)