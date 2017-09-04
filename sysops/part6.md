# Magento Infrastructure Workshops #6

This part of workshops will be dedicated to installing PHP and it's modules
required by Magento.

## PHP

We’ll install PHP 7 with all required
[modules](http://devdocs.magento.com/guides/v2.1/install-gde/system-requirements-tech.html).

* php7.0-fpm
* php7.0-bcmath
* php7.0-curl
* php7.0-gd
* php7.0-intl
* php7.0-mbstring
* php7.0-mcrypt
* php7.0-mysql
* php7.0-xml
* php7.0-xsl
* php7.0-zip
* php7.0-json
* php7.0-opcache
* php7.0-soap

```
root@mma1:~# apt-get install php7.0-fpm php7.0-bcmath php7.0-curl php7.0-gd php7.0-intl php7.0-mbstring php7.0-mcrypt php7.0-mysql php7.0-xml php7.0-xsl php7.0-zip php7.0-json php7.0-opcache php7.0-soap
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libapparmor1 libltdl7 libmcrypt4 libxpm4 libxslt1.1 libzip2 php-common php7.0-cli php7.0-common php7.0-readline
Suggested packages:
  libmcrypt-dev mcrypt php-pear
The following NEW packages will be installed:
  libapparmor1 libltdl7 libmcrypt4 libxpm4 libxslt1.1 libzip2 php-common php7.0-bcmath php7.0-cli php7.0-common php7.0-curl php7.0-fpm php7.0-gd php7.0-intl php7.0-json php7.0-mbstring php7.0-mcrypt php7.0-mysql php7.0-opcache
  php7.0-readline php7.0-soap php7.0-xml php7.0-xsl php7.0-zip
0 upgraded, 24 newly installed, 0 to remove and 1 not upgraded.
Need to get 5,370 kB of archives.
After this operation, 21.8 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://ftp.pl.debian.org/debian/ jessie/main libltdl7 amd64 2.4.2-1.11+b1 [45.4 kB]
Get:2 http://packages.dotdeb.org/ jessie/all php-common all 1:42-1~dotdeb+8.1 [10.8 kB]
Get:3 http://packages.dotdeb.org/ jessie/all php7.0-common amd64 7.0.18-1~dotdeb+8.1 [869 kB]
(....)

Creating config file /etc/php/7.0/mods-available/xmlwriter.ini with new version

Creating config file /etc/php/7.0/mods-available/xsl.ini with new version
Setting up php7.0-xsl (7.0.18-1~dotdeb+8.1) ...
Setting up php7.0-zip (7.0.18-1~dotdeb+8.1) ...

Creating config file /etc/php/7.0/mods-available/zip.ini with new version
Processing triggers for libc-bin (2.19-18+deb8u9) ...
Processing triggers for systemd (215-17+deb8u7) ...
Processing triggers for php7.0-fpm (7.0.18-1~dotdeb+8.1) ...
```

Next we need to adjust the configuration of  `/etc/php/7.0/fpm/php.ini`
changing the parameters:
```
memory_limit = 64M
max_execution_time = 18000
session.auto_start = off
zlib.output_compression = on
suhosin.session.cryptua = off
display_errors = Off
```

In the `/etc/php/7.0/fpm/pool.d/www.conf` we change the user and socket for php
processes:
```
user = supershop
group = supershop
listen = 127.0.0.1:9000
```

Next we restart PHP FPM:
```
root@mma1:/etc/php/7.0/fpm# systemctl restart php7.0-fpm.service
```

We’re done with PHP.

--------------------------------------------------------------------------------

#### Notice:

In order to preserve the same environment for cron jobs we need to apply the
same changes to `/etc/php/7.0/cli/php.ini`.

Tuning

PHP FPM umożliwia nam określenie polityki zarządzania procesami PHP. Służą do
tego m.in. zmienne:
PHP-FPM allows us to define policy of worker processes management. We can
configure it with variables like:

* pm.max\_children
* pm.start\_servers
* pm.min\_spare\_servers
* pm.max\_spare\_servers
* pm.max\_requests

If huge traffic is expected, it's good to increase `pm.max_children`. This will
allow PHP to handle more amount of queries at once(default values for this
variable are usually low). We can also set `catch_workers_output` to `yes`. It
won't speed up our application, but will direct PHP related logs to dedicated
log, so it will be easier to debug than browsing web server log.
