# Configuring `OpenVPN` for a Linux server and a Windows client

*First edition 2019-05-03.*

As of writing, `OpenVPN` is one of the most advanced and secure VPN solution. Its open-source nature and the use of certificates ensures a safe VPN connection. However, `OpenVPN` can be quite hard to configure for the first time. In this tutorial, I will walk you through the steps to configure a safe `OpenVPN` server.

## Installation
On most Linux distros, install `openvpn` package. Or build from source. On Windows, download the community build from <https://openvpn.net/community-downloads/>.

## Make a configuration file
