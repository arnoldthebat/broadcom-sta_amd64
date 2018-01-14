
DISCLAIMER
----------
This is an unofficial Release of Broadcom's 64 bit hybrid Linux driver for use 
with Broadcom based hardware. It has been compiled against 4.14.x kernel only at the moment.

SUPPORTED DEVICES
-----------------
The cards with the following PCI Device IDs are supported with this driver.
Both Broadcom and and Dell product names are described.   Cards not listed
here may also work.

	   BRCM		    PCI		  PCI		  Dell
	  Product Name	  Vendor ID	Device ID	Product ID
          -------------	 ----------	---------   	-----------
          4311 2.4 Ghz	    0x14e4	0x4311  	Dell 1390
          4311 Dualband	    0x14e4	0x4312  	Dell 1490
          4311 5 Ghz	    0x14e4    	0x4313  	
          4312 2.4 Ghz	    0x14e4	0x4315  	Dell 1395
          4313 2.4 Ghz	    0x14e4	0x4727 		Dell 1501/1504
          4321 Dualband	    0x14e4	0x4328  	Dell 1505
          4321 Dualband	    0x14e4	0x4328  	Dell 1500
          4321 2.4 Ghz	    0x14e4	0x4329  	
          4321 5 Ghz        0x14e4	0x432a  	
          4322 	Dualband    0x14e4	0x432b  	Dell 1510
          4322 2.4 Ghz      0x14e4 	0x432c  	
          4322 5 Ghz        0x14e4 	0x432d  	
          43142 2.4 Ghz     0x14e4	0x4365
          43224 Dualband    0x14e4	0x4353  	Dell 1520
          43225 2.4 Ghz     0x14e4	0x4357  	
          43227 2.4 Ghz     0x14e4	0x4358
          43228 Dualband    0x14e4	0x4359  	Dell 1530/1540
          4331  Dualband    0x14e4	0x4331
          4360  Dualband    0x14e4	0x43a0
          4352  Dualband    0x14e4	0x43a0

To find the Device ID's of Broadcom cards on your machines do:
# lspci -n | grep 14e4

NOTABLE CHANGES
---------------
	1st release - compliant with kernel version 4.14.3

REQUIREMENTS
------------
Building this driver requires that your machine have the proper tools,
packages, header files and libraries to build a standard kernel module.
This usually is done by installing the kernel developer or kernel source
package and varies from distro to distro. Consult the documentation for
your specific OS.

If you cannot successfully build a module that comes with your distro's
kernel developer or kernel source package, you will not be able to build
this module either.

If you try to build this module but get an error message that looks like
this:

make: *** /lib/modules/"release"/build: No such file or directory. Stop.

Then you do not have the proper packages installed, since installing the
proper packages will create /lib/modules/"release"/build on your system.

On Fedora install 'kernel-devel' from 
Package Manager (System-> Administration-> Add/Remove Software)
or 
yum install kernel-devel 
or
yum install kernel-PAE-devel

On Ubuntu, you will need headers and tools.  Try these commands:
# apt-get install build-essential linux-headers-generic
# apt-get build-dep linux

To check to see if you have this directory do this:

# ls /lib/modules/`uname -r`/build

BUILD INSTRUCTIONS
------------------
1. Clone the GIT repo

2. Build the driver as a Linux loadable kernel module (LKM):

# make clean   (optional)
# make

When the build completes, it will produce a wl.ko file in the top level
directory.

If your driver does not build, check to make sure you have installed the
kernel package described in the requirements above.

This driver uses cfg80211 API. Code for Wext API is present and can be built
but we have dropped support for it.
As before, the Makefile will still build the matching version for your system.

# make API=CFG80211
 or
# make API=WEXT (deprecated)

INSTALL INSTRUCTIONS
--------------------

Upgrading from a previous version:
---------------------------------

If you were already running a previous version of wl, you'll want to provide
a clean transition from the older driver. (The path to previous driver is
usually /lib/modules/<kernel-version>/kernel/net/wireless)

# rmmod wl 
# mv <path-to-prev-driver>/wl.ko <path-to-prev-driver>/wl.ko.orig
# cp wl.ko <path-to-prev-driver>/wl.ko
# depmod
# modprobe wl

The new wl driver should now be operational and your all done.

Fresh installation:
------------------
1: Remove any other drivers for the Broadcom wireless device.

There are several other drivers (besides this one) that can drive 
Broadcom 802.11 chips. These include b43, brcmsmac, bcma and ssb. They will
conflict with this driver and need to be uninstalled before this driver
can be installed.  Any previous revisions of the wl driver also need to
be removed.

Note: On some systems such as Ubuntu 9.10, the ssb module may load during
boot even though it is blacklisted (see note under Common Issues on how to
resolve this. Nevertheless, ssb still must be removed
(by hand or script) before wl is loaded. The wl driver will not function 
properly if ssb the module is loaded.

# lsmod  | grep "brcmsmac\|b43\|ssb\|bcma\|wl"

If any of these are installed, remove them:
# rmmod b43
# rmmod brcmsmac
# rmmod ssb
# rmmod bcma
# rmmod wl

To blacklist these drivers and prevent them from loading in the future:
# echo "blacklist ssb" >> /etc/modprobe.d/blacklist.conf
# echo "blacklist bcma" >> /etc/modprobe.d/blacklist.conf
# echo "blacklist b43" >> /etc/modprobe.d/blacklist.conf
# echo "blacklist brcmsmac" >> /etc/modprobe.d/blacklist.conf

2: Insmod the driver.

Otherwise, if you have not previously installed a wl driver, you'll need
to add a security module before using the wl module.  Most newer systems 
use lib80211 while others use ieee80211_crypt_tkip. See which one works for 
your system.

# modprobe lib80211 
  or 
# modprobe ieee80211_crypt_tkip

If your using the cfg80211 version of the driver, then cfg80211 needs to be
loaded:

# modprobe cfg80211

Then:
# insmod wl.ko

wl.ko is now operational.  It may take several seconds for the Network 
Manager to notice a new network driver has been installed and show the
surrounding wireless networks.
