# Magento Infrastructure Workshops #5

This part of workshops will be dedicated to installing MySQL database.

## MySQL installation

We’ll use Percona instead of MySQL. It’s a compatible fork of MySQL, but it’s
faster and better from DBA perspective.

During the installation we’ll be asked for MySQL root account’s password. Pick
*qwerty*.
```
root@mma1:~# apt-get install percona-server-server-5.7
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libaio1 libmecab2 libnuma1 percona-server-client-5.7 percona-server-common-5.7
The following NEW packages will be installed:
  libaio1 libmecab2 libnuma1 percona-server-client-5.7 percona-server-common-5.7 percona-server-server-5.7
0 upgraded, 6 newly installed, 0 to remove and 1 not upgraded.
Need to get 26.1 MB of archives.
After this operation, 208 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://ftp.pl.debian.org/debian/ jessie/main libaio1 amd64 0.3.110-1 [9,312 B]
(.....)
 * See http://www.percona.com/doc/percona-server/5.7/management/udf_percona_toolkit.html for more details


Processing triggers for libc-bin (2.19-18+deb8u9) ...
Processing triggers for systemd (215-17+deb8u7) ...
```

After mysql is installed we log in and create the database and a user for our Magento store.
```
root@mma1:~# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.17-13 Percona Server (GPL), Release '13', Revision 'fd33d43'

Copyright (c) 2009-2016 Percona LLC and/or its affiliates
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE supershop;
Query OK, 1 row affected (0.00 sec)

mysql> CREATE USER 'supershop'@'localhost' IDENTIFIED BY 'pasło';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON supershop.* TO 'supershop'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql>
```

We’re done with the database.

Notice that we’ve restricted the connections to MySQL *supershop* database to
user *supershop* and *localhost*. If we were on the production, we would need to
allow other hosts to connect.
