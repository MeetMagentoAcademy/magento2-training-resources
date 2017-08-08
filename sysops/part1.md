# Magento Infrastructure Workshops #1

This part of workshops will be dedicated to creating VirtualBox machine for
installing Magento.

## VirtualBox

If we have no access to the Internet (or we prefer to develop locally for
whatever reason, we’ll use VirtualBox an open virtualization platform licensed
with GNU GPL v2.  It’s stable and multiplatform  host system supporting all
mainstream guest operating systems.

1. Download [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
2. Download [Debian Linux Netinst image](https://www.debian.org/CD/netinst/).
   (pick amd64 architecture).
3. Install VirtualBox.
4. Start VirtualBox Click *New* button (or press Ctrl-N).
5. Type in:
    * name – Meet Magento Academy
    * type – Linux
    * version – Debian (64-bit)
6. Press Next and choose the amount of RAM you can assign (wise choice is 25% of
   what you have on your machine).
7. Press Next and choose to Create Virtual Drive with VDI and Dynamically
   Assigned Press.
8. Next and choose the 8GB size of a drive.
9. Press Create Press Start. VirtualBox will ask you to pick a image file (use
   the Debian Netinst you’ve just downloaded a few steps before).
10. The installer will boot. Choose Install option.
11. We’ll be fine with pressing Next in the installer until we’ll get to the
    point of Software Selection.
    * You can choose your own hostname (we suggest to pick mma1, and mma1.local
      as domain)
    * **On local machines (only!!)** you can type *qwerty* as the root password
      and create your own username (also with *qwerty* password).
    * For development purposes it’s OK to pick *use entire disk* for
      partitioning.
    * For Debian Packges Archive Country pick the country you’re in.
12. Uncheck everything except *SSH server* and *standard system utilities* on
    the Software Selection.
13. Agree to install grub boot loader on /dev/sda.
14. Finish and reboot the machine.

We have GNU/Linux Debian installation ready.
