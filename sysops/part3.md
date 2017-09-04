# Magento Infrastructure Workshops #3

This part of workshops will be dedicated to creating dedicated user for Magento
processes.

## Dedicated user for Magento processes

Magento application server uses PHP running with PHP-FPM. For security reasons
we’ll create a dedicated user for a web server and PHP application processes.

We run the command:
```
root@mma1:~# useradd --shell /bin/bash --create-home --base-dir /home supershop
```

We can check if everything went OK:
```
root@mma1:~# grep supershop /etc/passwd
supershop:x:1001:1001::/home/supershop:/bin/bash
```

We can see the user got 1001 ID in our system. Notice, it’s not important to
remember the ID of the user.

Tuning – if we want to avoid problems with limit of open file descriptor it’s
good to set up in `/etc/security/limits.conf` at the end of file the following
settings:
```
root    soft    nofile  51200
root    hard    nofile  65535
*       soft    nofile  51200
*       hard    nofile  65535
```
