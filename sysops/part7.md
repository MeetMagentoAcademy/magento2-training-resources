# Magento Infrastructure Workshops part #7

This part of workshops will be dedicated to installing web server.

## Web server

First we need to install Nginx with all addons:
```
root@mma1:/etc/php/7.0/fpm# apt-get install nginx-full
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libgd3 libnginx-mod-http-auth-pam libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libossp-uuid16 libvpx1 nginx-common
Suggested packages:
  libgd-tools uuid fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  libgd3 libnginx-mod-http-auth-pam libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libossp-uuid16 libvpx1 nginx-common nginx-full
0 upgraded, 11 newly installed, 0 to remove and 1 not upgraded.
Need to get 1,909 kB of archives.
After this operation, 5,037 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://ftp.pl.debian.org/debian/ jessie/main libvpx1 amd64 1.3.0-3 [599 kB]
Get:2 http://packages.dotdeb.org/ jessie/all nginx-common all 1.10.3-1~dotdeb+8.1 [104 kB]
Get:3 http://packages.dotdeb.org/ jessie/all libnginx-mod-http-auth-pam amd64 1.10.3-1~dotdeb+8.1 [82.0 kB]
(....)
Setting up libnginx-mod-stream (1.10.3-1~dotdeb+8.1) ...
Setting up nginx-full (1.10.3-1~dotdeb+8.1) ...
Processing triggers for libc-bin (2.19-18+deb8u9) ...
Processing triggers for systemd (215-17+deb8u7) ...
```

We’ll use the [sample Magento
configuration](https://github.com/magento/magento2/blob/develop/nginx.conf.sample)
and put it in `/etc/nginx/snippets/magento2.conf`(it can be used for many
virtual hosts if we have more than one):
```
root@mma1:/etc/php/7.0/fpm# curl --silent --show-error --location https://raw.githubusercontent.com/magento/magento2/develop/nginx.conf.sample > /etc/nginx/snippets/magento2.conf
```

Next we create virtualhost `/etc/nginx/sites-available/supershop.conf`:
```
upstream fastcgi_backend {
    server  127.0.0.1:9000;
}
server {
    listen 80;
    server_name mage.dev;
    set $MAGE_ROOT /home/supershop/public_html;
    include /etc/nginx/snippets/magento2.conf;
}
```

an we activate it (also removing default virtualhost):
```
root@mma1:~# cd /etc/nginx/sites-enabled
root@mma1:/etc/nginx/sites-enabled# ln -s ../sites-available/supershop.conf
root@mma1:/etc/nginx/sites-enabled# rm default
```

Hint
It's a good practice to validate the configuration of Nginx with `nginx -t`
command.

We change the owner of the Nginx processes in `/etc/nginx/nginx.conf` to our `supershop` user:
```
user supershop;
```

Restart Nginx:
```
root@mma1:/etc/php/7.0/fpm# systemctl restart nginx
```

We’re done with Nginx.
