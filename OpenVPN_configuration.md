# Configuring `OpenVPN` for a Linux server and a Windows client

*First edition 2019-05-08.*

As of writing, `OpenVPN` is one of the most advanced and secure VPN solution. Its open-source nature and the use of certificates ensures a safe VPN connection. However, `OpenVPN` can be quite hard to configure for the first time. In this tutorial, I will walk you through the steps to configure a safe `OpenVPN` server with IPv4 and IPv6 dual-stack.

## Installation
On most Linux distros, install `openvpn` package. Or build from source. On Windows, download the community build from <https://openvpn.net/community-downloads/>.

Install `easy-rsa` package to manage certificates. Install `bridge-utils` for ethernet bridging.

## Generating Certificates (PKI)

### CA
For OpenVPN 2.4 or higher, use Easy-RSA to generate certificates and keys using elliptic curves.

Append the following lines to `/etc/easy-rsa/vars` to use elliptic curves.
```
set_var EASYRSA_ALGO     ec
set_var EASYRSA_CURVE    secp521r1
set_var EASYRSA_DIGEST   "sha512"
```

Now set up PKI and generate a CA certificate.
```shell
# cd /etc/easy-rsa
# export EASYRSA=$(pwd)
# export EASYRSA_VARS_FILE=/etc/easy-rsa/vars
# easyrsa init-pki
# easyrsa build-ca
```
The generated CA certificate `/etc/easy-rsa/pki/ca.crt` should be copied to both server and client.

### Server and client certificates
Generate and sign server certificate and client certificate.
```shell
# easyrsa gen-req servername nopass
# easyrsa sign-req server servername
# easyrsa gen-req clientname nopass
# easyrsa sign-req client clientname
```
Copy the server certificate and key to `/etc/openvpn/server/`.
### HMAC key
```shell
# openvpn --genkey --secret /etc/openvpn/server/ta.key
```

## Make a configuration file
Get configuration examples in `/usr/share/openvpn/examples/` on Arch Linux, or `/usr/share/doc/openvpn/sample` on Fedora. Copy `server.conf` to `/etc/openvpn/server/server.conf` and make modifications.

Here's an example:
```
port 1194
proto udp
dev tun
ca ca.crt
cert servername.crt
key servername.key
dh none
topology subnet
server 10.32.64.0 255.255.255.0
server-ipv6 2001:1234:5678:9abc:abcd::/112
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp ipv6"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 2001:4860:4860::8888"
client-to-client
keepalive 10 120
tls-crypt ta.key
cipher AES-256-GCM
auth SHA512
tls-version-min 1.2
persist-key
persist-tun
status openvpn-status.log
verb 3
explicit-exit-notify 1
```

## Other changes
### Enable packet forwarding for both IPv4 & IPv6
```shell
sysctl net.ipv4.ip_forward=1
sysctl net.ipv6.conf.all.forwarding=1
```
To make these changes permanent, modify `/etc/sysctl.d/30-ipforward.conf`:
```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

### Add `firewalld` rules
Suppose you have your interface in `FedoraServer` zone.

If you have `openvpn` running on a different port, copy the `openvpn` service definition and modify it:
```shell
cp /usr/lib/firewalld/services/openvpn.xml /etc/firewalld/services/openvpn_xxx.xml
nano /etc/firewalld/services/openvpn_xxx.xml
firewall-cmd --zone=FedoraServer --add-service openvpn_xxx.xml
```

Now add IPv4 NAT routing rules:
```shell
firewall-cmd --zone=FedoraServer --add-masquerade
firewall-cmd --zone=FedoraServer --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
firewall-cmd --runtime-to-permanent
```
## Finishing
Use `ovpngen` to generate a `.ovpn` file for clients to use. Get the script from its [GitHub repo](https://github.com/graysky2/ovpngen), or [AUR](https://aur.archlinux.org/packages/ovpngen/).
```shell
# ./ovpngen example.org /etc/openvpn/server/ca.crt /etc/easy-rsa/pki/signed/client1.crt /etc/easy-rsa/pki/private/client1.key /etc/openvpn/server/ta.key 1194 udp > foo.ovpn
```
The output `.ovpn` file should be modified to match server configuration.

Run `openvpn /etc/openvpn/server/server.conf` to test configuration. When everything is ready, start the server service: `systemctl enable --now openvpn-server@server.service`.

## Troubleshooting
### ERROR: Cannot open TUN/TAP dev /dev/net/tun: No such device (errno=19)
After a kernel update, a reboot is needed to be able to load new modules.

## References
* [OpenVPN - ArchWiki](https://wiki.archlinux.org/index.php/OpenVPN)
* [Easy-RSA - ArchWiki](https://wiki.archlinux.org/index.php/Easy-RSA)
* [OpenVPN with Modern Cryptography](https://www.maths.tcd.ie/~fionn/misc/ec_vpn.php)
* [Openvpn - Fedora Project Wiki](https://fedoraproject.org/wiki/Openvpn)
* [tls - Is Common Name encoded in the certificate? - Cryptography Stack Exchange](https://crypto.stackexchange.com/questions/1836/is-common-name-encoded-in-the-certificate)
* [Internet sharing - ArchWiki](https://wiki.archlinux.org/index.php/Internet_sharing#Enable_packet_forwarding)
* [GettingStartedwithOVPN – OpenVPN Community](https://community.openvpn.net/openvpn/wiki/GettingStartedwithOVPN)
* [2x HOW TO | OpenVPN](https://openvpn.net/community-resources/how-to/)
* [[Solved] OpenVPN 2.3.1 - ERROR: Cannot open TUN/TAP dev /dev/net/tun / Networking, Server, and Protection / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=163377)
* [OpenVPN 2.4 and pure elliptic curve crypto setup - Page 2 - OpenVPN Support Forum](https://forums.openvpn.net/viewtopic.php?t=23227&start=30)