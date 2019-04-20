# Setting Up `nginx` with a free TLS certificate

This tutorial assumes you use Fedora Server/Workstation 29.

## Installation
Install `nginx`, `certbot` and `certbot-nginx`:
```shell
# dnf update
# dnf install nginx certbot certbot-nginx
```

## Obtain a certificate
certbot's nginx plugin can automatically obtain a certificate and configure `nginx` to use it.
```shell
# certbot --nginx
```
If port 80 is not available, you have to manually use DNS challenge. In this case, no automatic renewal or configuration is possible.
```shell
# certbot certonly --manual --preferred-challenges dns
```

## Turn on renewal timer
On most Linux distributions, `certbot` comes with a `systemd` timer. Enable and start the timer:
```shell
# systemctl enable --now certbot-renew.timer
```
Note that automatic renewal only works with certificates obtained with a plugin.

## Configure `nginx`
`certbot-nginx` will automatically start `nginx` as a standalone process after obtaining a certificate. Kill `nginx` with:
```shell
# ps -ax | grep nginx
 4521 ?        Ss     0:00 nginx: master process /usr/sbin/nginx
 4522 ?        S      0:07 nginx: worker process
20042 pts/0    S+     0:00 grep --color=auto nginx
# kill -s QUIT 4521
```

Then modify `/etc/nginx/nginx.conf` to remove duplicate entries.

On Fedora 29, there is a bug with `nginx`, where it might report "nginx: [warn] could not build optimal types_hash, you should increase either types_hash_max_size: 2048 or types_hash_bucket_size: 64; ignoring types_hash_bucket_size"

To workaround the bug, modify `types_hash_max_size` from `2048` to `4096` in `/etc/nginx/nginx.conf`. Then use `nginx -t` to test your configuration again.

## References
* [Nginx on Fedora 26: could not build optimal types_hash error message - Stack Overflow](https://stackoverflow.com/questions/46031491/nginx-on-fedora-26-could-not-build-optimal-types-hash-error-message)
* [Certbot - ArchWiki](https://wiki.archlinux.org/index.php/Certbot#Manual)
* [Beginner’s Guide - nginx.org](http://nginx.org/en/docs/beginners_guide.html#control)