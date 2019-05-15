# `nginx` Notes

*First edition 2019-05-06.*
*Modified 2019-05-15: added http basic auth*

## 1. HTTP Basic Authentication

To configure HTTP Basic Authentication, add the following lines to either a server block, or a location block.
```
auth_basic "don't know how to name it :)";
auth_basic_user_file /etc/nginx/.basic_auth_public;
```

A user file defines users and passwords. The password can be generated with salted SHA-512, as most modern Linux distros support it with `crypt()`.
```
# openssl passwd -6
```
Then append the string to a user file:
```
username:$6$wRiU.xWVgGWqqV6n$aV9Kx6HB2AO24AYHNzsgoXmsBZX6Iih5fZBftLqbfCO/yx5EFBtreOEBtFkZ2RihSV.lL0gaQFwyTIu7AzUMN0
```

Note that HTTP Basic Authentication is not considered safe.

### References
* [NGINX Docs | Restricting Access with HTTP Basic Authentication](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)
* [Module ngx_http_auth_basic_module](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)
* [tls - Is BASIC-Auth secure if done over HTTPS? - Information Security Stack Exchange](https://security.stackexchange.com/questions/988/is-basic-auth-secure-if-done-over-https)
* [Is every hash format that nginx accepts for HTTP Basic Auth weak against brute force? - Information Security Stack Exchange](https://security.stackexchange.com/questions/46883/is-every-hash-format-that-nginx-accepts-for-http-basic-auth-weak-against-brute-f)

## 2. WebDAV
The `nginx` package provided with your distro should be compiled with [`ngx_http_dav_module`](http://nginx.org/en/docs/http/ngx_http_dav_module.html), which provides basic WebDAV support. It's recommended to add an [extension module](https://github.com/arut/nginx-dav-ext-module). See references for specific steps on Arch Linux.

WebDAV users access files as `http` user. To make files writable, either change owner to `http` or give other user write permission. `dav_access` defines permissions for new files and folders created by WebDAV user. To limit access to a WebDAV server, use `auth_basic`.
```json
location /dav {
    root   /srv/http;

    auth_basic "don't know how to name it :)";
    auth_basic_user_file /etc/nginx/.basic_auth_public;

    dav_methods PUT DELETE MKCOL COPY MOVE;
    dav_ext_methods PROPFIND OPTIONS;

    # Adjust as desired:
    dav_access user:rw group:rw all:r;
    client_max_body_size 0;
    create_full_put_path on;
    client_body_temp_path /srv/client-temp;
    autoindex on;

    allow 192.168.178.0/24;
    deny all;
}
```

### References
* [WebDAV - ArchWiki](https://wiki.archlinux.org/index.php/WebDAV)
* [AUR (en) - nginx-mainline-mod-dav-ext](https://aur.archlinux.org/packages/nginx-mainline-mod-dav-ext/)
* [Module ngx_http_dav_module](http://nginx.org/en/docs/http/ngx_http_dav_module.html)
* [arut/nginx-dav-ext-module: nginx WebDAV PROPFIND,OPTIONS,LOCK,UNLOCK support](https://github.com/arut/nginx-dav-ext-module)

## 3. Configuration Examples
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