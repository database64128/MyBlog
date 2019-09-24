# Wordpress: Setting up my blog with `nginx` + `Wordpress` + `php-fpm`

> Setting up a WordPress site takes five minutes.

And it took me almost a day to set up my own Wordpress site. Yes, almost a day.

Quite a few friends have recently set up their website with `Wordpress`, or its derivatives. Decorated with gorgeous themes and neat little gadgets, their carefully-crafted web pages have once again, woke the idea of renovating my own half-broken poorly-maintained websites.

> No one:
> Definitely no one:
> Me: Modified a Adobe Dreamweaver web page template to be my web page.

So, with little to no experience with front-end development, I decided to start with the 5-min-setup Wordpress.

## `nginx` as reverse proxy + `apache` in a subdirectory: A failed experiment

So basically, why I failed with this setup? Actually, it turns out there is nothing wrong with this setup. Theoretically it should work. But when I was working with this setup, I didn't realize this:

> Wordpress has a built-in feature to correct the URL so that, even if you typed the wrong address, it can automatically fix the address and redirect you to the home page. This feature works by comparing the current URL with the one set in its database. If the URL doesn't match, Wordpress sends a 301 to redirect you to the URL set in its database.

So natually, I started the `apache` web server and finished the setup from the internal port. Then I attempted to make a reverse proxy to the internal port from nginx. For the whole time I just can't figure out why it keeps redirecting. In the end, I found a wrong StackOverflow answer which is for a totally different problem and decided to give up on this setup.

## Steps: `nginx` + `php-fpm`

### Packages

Install all necessary packages with `pacman`:

```bash
# pacman -S --needed nginx mariadb php php-gd php-fpm phpmyadmin
```

We are not installing `wordpress` with `pacman` but instead manually download and unzip:

```bash
# wget https://wordpress.org/latest.tar.gz
# tar xvzf latest.tar.gz
```

### Preparing `mariadb` and `php`

If you plan to install the database on a btrfs file system, the CoW feature should be disabled for the database directory before installation:

```bash
# btrfs subvolume create /var/lib/mysql
# chattr +C /var/lib/mysql
```

Finish MariaDB installation and start the service:

```bash
# mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
# systemctl enable --now mariadb.service
# mysql_secure_installation
```

Login MariaDB as root and create a database and a user for Wordpress.

```bash
# mysql -u root -p
MariaDB> CREATE DATABASE wordpress;
MariaDB> GRANT ALL PRIVILEGES ON wordpress.* TO "wp-user"@"localhost" IDENTIFIED BY "choose_db_password";
MariaDB> FLUSH PRIVILEGES;
MariaDB> EXIT
```

Optionally, create a user for phpMyAdmin. You can't login phpMyAdmin using root as it's not allowed by MariaDB.

```bash
MariaDB> CREATE USER 'pmauser'@'localhost' IDENTIFIED BY 'password_here';
MariaDB> GRANT ALL PRIVILEGES ON *.* TO 'pmauser'@'localhost' WITH GRANT OPTION;
MariaDB> FLUSH PRIVILEGES;
```

Modify `/etc/php/php.ini` to enable extensions, increase the file size limit of `php` so that you can upload larger pictures.

```
date.timezone = America/Los_Angeles

extension=gd
extension=pdo_mysql
extension=mysqli
extension=zip

memory_limit = 256M
post_max_size = 128M
upload_max_filesize = 128M
```

Enable and start the `php-fpm` service:

```bash
# systemctl enable --now php-fpm.service
```

By default it creates a socket at `/run/php-fpm/php-fpm.sock`. We will be using it in `nginx`'s config.

### Configuring `nginx` and `phpmyadmin`

An example of using `nginx` as the front end for `phpmyadmin`:

```
upstream php {
    server unix:/run/php-fpm/php-fpm.sock;
}

server {
    listen [::]:8080 ipv6only=off;
    server_name "";
    root /usr/share/webapps/phpMyAdmin;
    index index.php;

    allow 192.168.50.0/24;
    allow 10.8.0.0/24;
    deny all;

    location / {
        try_files $uri $uri/ /index.php;
    }

    location ~ \.php$ {
        include fastcgi.conf;
        fastcgi_intercept_errors on;
        fastcgi_pass php;
        fastcgi_index index.php;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
    }
}
```

An example of using `nginx` as the front end for `nginx` and `php-fpm`:

```
location /wp {
    try_files $uri $uri/ /wp/index.php?$args;
}

location ~ \.php$ {
    include fastcgi.conf;
    fastcgi_intercept_errors on;
    fastcgi_pass php;
    fastcgi_index index.php;
}
```

Remember to set the root directive, the index directive and `client_max_body_size 0;` in the server block. Also you might want to limit the allowed IPs before finishiing setting up Wordpress.

### Setting Up `wordpress` in 5 minutes

Now open your web browser, enter the address and follow the setup wizard.

### Wordpress migration: URL changes

To be finished.

## References

* [MariaDB - ArchWiki](https://wiki.archlinux.org/index.php/MariaDB)
* [PHP - ArchWiki](https://wiki.archlinux.org/index.php/PHP)
* [PHP: List of Supported Timezones - Manual](https://secure.php.net/manual/en/timezones.php)
* [phpMyAdmin - ArchWiki](https://wiki.archlinux.org/index.php/PhpMyAdmin)
* [nginx - ArchWiki](https://wiki.archlinux.org/index.php/Nginx)
* [Apache HTTP Server - ArchWiki](https://wiki.archlinux.org/index.php/Apache_HTTP_Server)
* [Wordpress - ArchWiki](https://wiki.archlinux.org/index.php/Wordpress)

* [Can't log into phpMyAdmin: mysqli_real_connect(): (HY000/1698): Access denied for user 'root'@'localhost' | DevAnswers.co](https://devanswers.co/phpmyadmin-access-denied-for-user-root-localhost/)
* [PHP: Description of core php.ini directives - Manual](https://www.php.net/manual/en/ini.core.php#ini.upload-max-filesize)

* [How to install WordPress | WordPress.org](https://wordpress.org/support/article/how-to-install-wordpress/)
* [WordPress | NGINX](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/)
* [Nginx | WordPress.org](https://wordpress.org/support/article/nginx/)