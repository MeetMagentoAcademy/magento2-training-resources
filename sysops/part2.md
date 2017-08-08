# Magento Infrastructure Workshops #2

This part of workshops will be dedicated to configuring network for our newly
created virtual machine and logging in to it.

## How to log in

The most convenient way to manage our Linux is to login in via SSH. On
VirtualBox we need to do some preparations.  With VirtualBox do as follows:

1. Power off the virtual machine
2. Go to Network Settings
3. Turn on Card 2
4. Attach it as *host-only* interface
5. Pick the default name *VirtualBox Host-Only Ethernet Adapter*
6. The press OK
7. In the /etc/network/interfaces add the following lines at the end:
	```
    allow-hotplug eth1 
    iface eth1 inet dhcp
	```
8. Run ifup command: `ifup eth1`

 
Then on VirtualBox we need to check what’s the IP and log in via putty/SSH as
follows:

1. Before you’ll use ssh you need to check what’s the ip address of the machine
2. Check the ip address of your machine with ip command (in the example we have
eth1 for VirtualBox).
	```
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	(.....)
	3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
		link/ether 08:00:27:53:6e:70 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.101/24 brd 192.168.56.255 scope global eth1
		   valid_lft forever preferred_lft forever
		inet6 fe80::a00:27ff:fe53:6e70/64 scope link
		   valid_lft forever preferred_lft forever
	root@mma1:~#
	```
3. Take eth1’s address.
4. Download putty and use the IP in *Host Name (or ip address)*.
5. Log in as a user you’ve just created and then switch to super user with `su`
command.
