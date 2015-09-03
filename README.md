ts7800 Buildroot configuration
==============================

This repository provides configuration files and instructions to build an
Linux kernel, busybox-based initramfs, and a full Arch Linux
installation.

Start by cloning this repository.

Partition an SD card
--------------------
First, use `lsblk` to find the device name for your SD card.  On my
machine, it was `/dev/sdc`, but I'll use `/dev/sdx` in all the examples
below to avoid any unfortunate copy-and-paste errors.

```
# fdisk /dev/sdx

Welcome to fdisk (util-linux 2.26.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): o
Created a new DOS disklabel with disk identifier 0xeb7534de.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-7729151, default 2048): <press enter>
Last sector, +sectors or +size{K,M,G,T,P} (2048-7729151, default 7729151): +6M

Created a new partition 1 of type 'Linux' and of size 6 MiB.

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): da
Changed type of partition 'Linux' to 'Non-FS data'.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (14336-7729151, default 14336): <press enter> 
Last sector, +sectors or +size{K,M,G,T,P} (14336-7729151, default 7729151): <press enter>

Created a new partition 2 of type 'Linux' and of size 3.7 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

The first partition can be any size, but 6M should be plenty for the
kernel, initramfs, and device tree blob. It needs to be type `da`,
though for the TS-BOOTROM to work.

Write the bootloader code
-------------------------
Now write the bootloader code to the SD card.

```
# dd if=mbr-ts-7800.dd of=/dev/sdx bs=446 count=1
```

Create an image with Buildroot
------------------------------
[Download Buildroot][br] and extract it to a directory adjacent to this
repository.  `cd` into the Buildroot project.

```
$ make BR2_EXTERNAL=../ts7800-buildroot ts7800_defconfig
```

From now on, you can omit the `BR2_EXTERNAL` variable; Buildroot wrote
it into a secret file for you.

If you would like to customize the build, `make menuconfig`, `make
linux-menuconfig`, and `make busybox-menuconfig` will launch menus to
configure the buildroot image, the linux kernel, and busybox
respectively. Once everything is to your liking, build the project.

```
$ make
```

Write the image
---------------
Still from within the Buildroot directory:

```
# dd if=output/images/zImage of=/dev/sdx1 bs=4M conv=fsync
```

[br]: http://buildroot.uclibc.org/download.html
