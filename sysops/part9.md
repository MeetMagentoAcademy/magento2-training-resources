# Magento Infrastructure Workshops #9

This part of workshops will be dedicated to deploying Magento code in our
virtual machine.

## Code deployment

We need to download the Magento code from
[magento.com](https://magento.com/tech-resources/download).  Pick .tar.gz
format in the Download on the bottom.

We can do so on our desktop and then upload it logging to our server using
winscp (instead of putty).

While I installed the Linux I created the *kalkos* account, so I’ll upload
Magento code to *kalkos*’s home folder and then from *supershop* user I’ll unpack it
to the *public\_html* folder.

We change the context to the *supershop* user, who’s supposed to run Magento:
```
root@mma1:/etc/nginx/sites-enabled# su - supershop
```

```
supershop@mma1:~$ mkdir ~/public_html && cd ~/public_html
```

and we unpack the code:
```
supershop@mma1:~/public_html$ tar zxvf /home/kalkos/Magento-CE-2.1.6-2017-03-29-01-08-05.tar.gz
(......)
```

We can now check *http://localhost/setup*.
