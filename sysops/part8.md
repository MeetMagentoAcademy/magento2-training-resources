# Magento Infrastructure Workshops #8

This part of workshops will be dedicated to installing two Redis instances for
session and cache.

## Two Redis instances

We assume we have unmodified Debian 8.8, so our init deamon is systemd. sysVinit
would also do the trick but with different commands. 

Before installing Redis we need to adjust some kernel parameters.

In `/etc/sysctl.conf` set `vm.overcommit_memory` to `1` and load the
configuration `sysctl -p`.

We’re ready to install Redis. We need to instances: cache i session running on
different ports.
```
root@mma1:/etc/php/7.0/fpm# apt-get install redis-server
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  redis-tools
Suggested packages:
  ruby-redis
The following NEW packages will be installed:
  redis-server redis-tools
0 upgraded, 2 newly installed, 0 to remove and 1 not upgraded.
Need to get 1,119 kB of archives.
After this operation, 3,736 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://packages.dotdeb.org/ jessie/all redis-tools amd64 2:3.2.8-1~dotdeb+8.1 [605 kB]
Get:2 http://packages.dotdeb.org/ jessie/all redis-server amd64 2:3.2.8-1~dotdeb+8.1 [514 kB]
Fetched 1,119 kB in 1s (726 kB/s)
Selecting previously unselected package redis-tools.
(Reading database ... 30649 files and directories currently installed.)
Preparing to unpack .../redis-tools_2%3a3.2.8-1~dotdeb+8.1_amd64.deb ...
Unpacking redis-tools (2:3.2.8-1~dotdeb+8.1) ...
Selecting previously unselected package redis-server.
Preparing to unpack .../redis-server_2%3a3.2.8-1~dotdeb+8.1_amd64.deb ...
Unpacking redis-server (2:3.2.8-1~dotdeb+8.1) ...
Processing triggers for man-db (2.7.0.2-5) ...
Processing triggers for systemd (215-17+deb8u7) ...
Setting up redis-tools (2:3.2.8-1~dotdeb+8.1) ...
Setting up redis-server (2:3.2.8-1~dotdeb+8.1) ...
Processing triggers for systemd (215-17+deb8u7) ...
```

```
root@mma1:/etc/php/7.0/fpm# systemctl disable redis-server 
Executing /usr/sbin/update-rc.d redis-server defaults
Executing /usr/sbin/update-rc.d redis-server disable
insserv: warning: current start runlevel(s) (empty) of script `redis-server' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `redis-server' overrides LSB defaults (0 1 6).
Removed symlink /etc/systemd/system/redis.service.
root@mma1:/etc/php/7.0/fpm$ systemctl stop redis-server
```

Next we copy the INI Redis file:
```
cp /lib/systemd/system/redis-server.service /etc/systemd/system/redis-cache.service
cp /lib/systemd/system/redis-server.service /etc/systemd/system/redis-session.service
```
In `/etc/systemd/system/redis-cache.service` file we modify starting settings:
```
ExecStart=/usr/bin/redis-server /etc/redis/redis-cache.conf
PIDFile=/var/run/redis/redis-cache.pid
ReadWriteDirectories=-/var/lib/redis-cache
```

In `/etc/systemd/system/redis-session.service` file we modify starting settings:
```
ExecStart=/usr/bin/redis-server /etc/redis/redis-session.conf
PIDFile=/var/run/redis/redis-session.pid
ReadWriteDirectories=-/var/lib/redis-session
```

From both redis-session.service and redis-cache.service files we remove lines
containing `Alias` and `RunTimeDirectory`. If we don’t we will meet some errors.

For each instance we create it’s folder:
```
mkdir /var/lib/redis-cache && chown redis:redis /var/lib/redis-cache
mkdir /var/lib/redis-session && chown redis:redis /var/lib/redis-session
```

We change current folder to `/etc/redis`, we duplicate the configuration so it listens on different port and use different PID and log files.
```
root@mma1: ~$ cd /etc/redis 
root@mma1: ~$ cp redis.conf redis-cache.conf && chown redis:redis redis-cache.conf
root@mma1: ~$ cp redis.conf redis-session.conf && chown redis:redis redis-session.conf
```

In `redis-cache.conf` we change:
```
dir /var/lib/redis-cache
logfile /var/log/redis/redis-cache.log
pidfile /var/run/redis/redis-cache.pid
port 6379
```
In `redis-session.conf` we change:
```
dir /var/lib/redis-session
logfile /var/log/redis/redis-session.log
pidfile /var/run/redis/redis-session.pid
port 6380
```

We make systemd, to prepare for two active Redis instances:
```
systemctl daemon-reload
systemctl enable redis-session.service redis-cache.service
systemctl start redis-session.service redis-cache.service
```

After starting two instances (knowing their ports) we can check if they work properly:
```
redis-cli -p 6379 ping
redis-cli -p 6380 ping
```
