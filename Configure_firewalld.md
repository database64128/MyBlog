# Configure `firewalld`

With the release of `firewalld 0.6.0`, `firewalld` uses the new `nftables` as the new default backend. This change will come with CentOS 8 too.

`firewalld` divides different network interfaces into `zones`. It applies `public` zone to a new network interface by default. You can add allowed services and ports to any zone. It's recommended to set up your own service configs instead of adding allowed ports.

Changes made by `firewall-cmd` are by default temporary. They will be lost after restarting or reloading. To make a permanent change, add a `--permanent` parameter. Then use `firewall-cmd --reload` to load permanent configurations. Or use `firewall-cmd --runtime-to-permanent` to make current session's configuration permanent. `firewall-offline-cmd` can be used when `firewalld` is not running. It makes permanent changes.

## General Configuration Procedures

### 1. Make your own services.
Pre-defined servers are located in `/usr/lib/firewalld/services`. Copy one of them to `/etc/firewalld/services` and make your modifications.

```shell
# cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/ssh_customized.xml
#nano /etc/firewalld/services/ssh_customized.xml
```

An example of a service's xml is like this:
```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>DNS</short>
  <description>The Domain Name System (DNS) is used to provide and request host and domain names. Enable this option, if you plan to provide a domain name service (e.g. with bind).</description>
  <port protocol="tcp" port="53"/>
  <port protocol="udp" port="53"/>
</service>
```

### 2. Add your customized services to a zone
First check if your service is already allowed in the zone:
```shell
# firewall-offline-cmd --zone=home --list-services
dhcpv6-client mdns samba-client ssh
```
To add a service, run:
```shell
# firewall-offline-cmd --zone=home --add-service=ssh_customized
success
```
Then check the zone's allowed services:
```shell
# firewall-offline-cmd --zone=home --list-services
dhcpv6-client mdns samba-client ssh ssh_customized
```
It's important to add your customized services, especially ssh, to the default zone before starting the firewall.
### 3. Enable/start the `firewalld` service
```shell
# systemctl enable --now firewalld
```
### 4. Change existing interfaces to a different zone
*As of `firewalld 0.6.3`, `firewall-offline-cmd` can't change a network interface's zone even though it returns success. You have to start the firewall first before making zone changes.*

```shell
# firewall-cmd --permanent --zone=home --change-interface=eth0
```