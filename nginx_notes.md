# `nginx` Notes

*First edition 2019-05-06.*

## 1. WebDAV
The `nginx` package provided with your distro should be compiled with [`ngx_http_dav_module`](http://nginx.org/en/docs/http/ngx_http_dav_module.html), which provides basic WebDAV support. It's recommended to add an [extension module](https://github.com/arut/nginx-dav-ext-module). See references for specific steps on Arch Linux.

### References
* [WebDAV - ArchWiki](https://wiki.archlinux.org/index.php/WebDAV)
* AUR (en) - nginx-mainline-mod-dav-ext
https://aur.archlinux.org/packages/nginx-mainline-mod-dav-ext/
* [arut/nginx-dav-ext-module: nginx WebDAV PROPFIND,OPTIONS,LOCK,UNLOCK support](https://github.com/arut/nginx-dav-ext-module)

## 2. Configuration Examples
* Rewrite http request to https
```json
server {
    listen [::]:443 ssl http2 ipv6only=off;
}

server {
    if ($host = la.cube64128.cn) {
        return 301 https://$host$request_uri;
    }

    listen       [::]:80 ipv6only=off;
    server_name la.cube64128.cn;
    return 404; 
}
```

* location alias
```json
server {
    location / {
        root /srv/http/;
    }
    location /pics {
        alias /root/Pictures/shared/;
    }
}
```

* listen on http and https in one server block
server {
    listen [::]:80 ipv6only=off;
    listen [::]:443 ssl http2 ipv6only=off;
}


### References
* [nginx HTTPS serving with same config as HTTP - Server Fault](https://serverfault.com/questions/10854/nginx-https-serving-with-same-config-as-http)
* [How to alias directories in nginx? - Server Fault](https://serverfault.com/questions/748634/how-to-alias-directories-in-nginx)
* [nginx - ArchWiki](https://wiki.archlinux.org/index.php/Nginx)
* [Do you need separate IPv4 and IPv6 listen directives in nginx? - Server Fault](https://serverfault.com/questions/638367/do-you-need-separate-ipv4-and-ipv6-listen-directives-in-nginx)