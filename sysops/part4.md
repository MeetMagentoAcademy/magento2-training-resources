# Magento Infrastructure Workshops #4

This part of workshops will be dedicated to installing Debian repositories
containing software required to run Magento.

## Debian repositories


Debian offers it’s own repositories, but we’ll add *Dotdeb*, which offers newest
versions of Redis and PHP 7.

We can follow the instructions at [Dotdeb](https://www.dotdeb.org/instructions/).

In `/etc/apt/sources.list.d/dotdeb.list` we add lines (the folder should exist,
but the file will be new):
```
deb http://packages.dotdeb.org jessie all
deb-src http://packages.dotdeb.org jessie all
```

Install the curl command:
```
root@mma1:~# apt-get install curl
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libcurl3
The following NEW packages will be installed:
  curl libcurl3
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
(....)
Processing triggers for man-db (2.7.0.2-5) ...
Setting up libcurl3:amd64 (7.38.0-4+deb8u5) ...
Setting up curl (7.38.0-4+deb8u5) ...
Processing triggers for libc-bin (2.19-18+deb8u9) ...
```

The we need to authorize the repository with adding it’s public key to trusted keys:
```
root@mma1:~# curl -sSL https://www.dotdeb.org/dotdeb.gpg | apt-key add -
OK
```

Update the repositories:
```
root@mma1:~# apt-get update
Ign http://ftp.pl.debian.org jessie InRelease
Get:1 http://security.debian.org jessie/updates InRelease [63.1 kB]
(....)
Ign http://packages.dotdeb.org jessie/all Translation-en_US
Ign http://packages.dotdeb.org jessie/all Translation-en
Fetched 1,251 kB in 2s (485 kB/s)
Reading package lists... Done
```

MySQL repositories:
```
root@mma1:~# wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
--2017-05-09 06:46:58--  https://repo.percona.com/apt/percona-release_0.1-4.jessie_all.deb
Resolving repo.percona.com (repo.percona.com)... 74.121.199.234
Connecting to repo.percona.com (repo.percona.com)|74.121.199.234|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6580 (6.4K) [application/octet-stream]
Saving to: ‘percona-release_0.1-4.jessie_all.deb’

percona-release_0.1-4.jessie_all.deb                        100%[==========================================================================================================================================>]   6.43K  --.-KB/s   in 0s

2017-05-09 06:46:58 (34.4 MB/s) - ‘percona-release_0.1-4.jessie_all.deb’ saved [6580/6580]

root@mma1:~# dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
Selecting previously unselected package percona-release.
(Reading database ... 30065 files and directories currently installed.)
Preparing to unpack percona-release_0.1-4.jessie_all.deb ...
Unpacking percona-release (0.1-4.jessie) ...
Setting up percona-release (0.1-4.jessie) ...
```

Update the repositories again to see everything is all right:
```
root@mma1:~# apt-get update
Ign http://ftp.pl.debian.org jessie InRelease
Hit http://security.debian.org jessie/updates InRelease
(.....)
Ign http://packages.dotdeb.org jessie/all Translation-en
Get:5 http://repo.percona.com jessie/main amd64 Packages [21.3 kB]
Ign http://repo.percona.com jessie/main Translation-en_US
Ign http://repo.percona.com jessie/main Translation-en
Fetched 54.7 kB in 7s (7,445 B/s)
Reading package lists... Done
root@mma1:~#
```

We’re done with repositories.

