# Configuring xrdp to run a Gnome session with Xorg

Xrdp should work out of box with the default configuration of working as a proxy to a Xvnc session. However, this usually results in degraded performance. Our goal is to run a xrdp session natively with the Xorg server.

## Steps

The following steps are conducted on Fedora Workstation 29. They should work on other distros too.

### 1. Installing `xrdp`
Specifically, make sure the `xorgxrdp` package is installed.

### 2. Configuring `xrdp`
Edit `/etc/xrdp/xrdp.ini` to set `Xorg` as the autorun session. Set security settings to use `tls` and add a TLS certificate.

*Note: As of 2018-11-18, RDP client on Windows 10 1809 Build 17763.107 supports up to TLS 1.2.

`/etc/xrdp/xrdp.ini`
```shell
; You may want to change listening port
port=13389

; Set TLS as the security layer and add certificate
security-layer=tls

; Set Xorg as the default session
autorun=Xorg
```

Next, set gnome as the session to start when connected with `xrdp`. Note that `~/.Xinitrc` is not needed.

`/etc/sysconfig/desktop`
```shell
PREFERRED=gnome-session
unset DBUS_SESSION_BUS_ADDRESS; gnome-session
```
Dont forget the other config file:
`/etc/X11/Xwrapper.config`
```shell
allowed_users=anybody
```

### 3. Get set and start the service!
`xrdp` should be ready for use. Start the service, or restart the service if it is already running. Reloading `xrdp` is not available.
```shell
# systemctl enable --now xrdp
```
Or, 
```shell
# systemctl restart xrdp
```