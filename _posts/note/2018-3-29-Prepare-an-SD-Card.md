---
layout: post
title: Prepare an SD Card
description: How to flash the linux image to an SD card
category: note
---

往SD card里刷Linux系统里刷linux image有两种方法：\\
1 直接刷预编译好的full SD card image (.sdcard)\\
2 Individually load the bootloader, kernel, device tree, and rootfs

### 1. Linux Image Contain Four Separate Pieces

• Linux OS kernel image (zImage, eg: zImage)\\
• Device tree file (\*.dtb, eg: zImage-imx6q-sabresd.dtb)\\
• U-Boot bootloader image  (eg: u-boot-imx6qsabresd.imx-sd)\\
• Root (\*.ext3 or \*.ext4, eg: fsl-image-validation-imx-imx6qsabresd.ext4)

### 2. Identify the SD Card

```
cat /proc/partitions
```

我的SD card分区后名字是sdb, sdb1, sdb2. 可以插拔一下SD card来确认SD card的名字。

### 3.1 Copying the Full SD Card Image

```
sudo dd if=fsl-image-validation-imx-imx6qsabresd.sdcard of=/dev/sdb bs=1M && sync
```

The full SD card image (with the extension .sdcard) already contains partitions, and contains U-Boot, Linux image, device trees, and the rootfs. \\
"of"的参数根据自己SD card的名字而定。


至此Linux image已经刷到SD card里面。

### 3.2 Copy Individual Parts

(1) Partition the SD/MMC card

卸载SD card:
```
sudo umount /dev/sdb
sudo umount /dev/sdb1
sudo umount /dev/sdb2
```

分区：
```
 sudo fdisk /dev/sdb
p [lists the current partitions]
d [to delete existing partitions. Repeat this until no unnecessary partitions are reported by the 'p' command to start fresh.]
n [create a new partition]
p [create a primary partition - use for both partitions]
1 [the first partition]
20480 [starting at offset sector]
1024000 [size for the first partition to be used for the boot images]
p [to check the partitions]
n
p
2
1228800 [starting at offset sector, which leaves enough space for the kernel, the bootloader and its configuration data]
<enter> [using the default value will create a partition that extends to the last sector of the media]
p [to check the partitions]
w [this writes the partition table to the media and fdisk exits]
```

(2) Copy a bootloader image

```
sudo dd if=u-boot-imx6qsabresd.imx-sd of=/dev/sdb bs=512 seek=2 conv=fsync
```

(3) Copy the kernel image and DTB file

• Format partition 1 on the card as VFAT:

```
sudo mkfs.vfat /dev/sdb1
```

The pre-built SD card image uses the VFAT partition for storing kernel image and DTB, which requires a VFAT partition that is mounted as a Linux drive and the files are simply copied into it. 

• Mount the formatted partition 1:

```
cd ~
mkdir mountpoint
sudo mount /dev/sdb1 mountpoint
```

 • Copy the zImage and *.dtb files to the mountpoint:

```
mv zImage-imx6q-sabresd.dtb imx6q-sabresd.dtb
sudo cp zImage mountpoint/
sudo cp imx6q-sabresd.dtb mountpoint/
```

这里需要把zImage-imx6q-sabresd.dtb 重命名为 imx6q-sabresd.dtb. 因为U-Boot配置文件中写的device tree file name 是 imx6q-sabresd.dtb.

• Unmount the partition:

```
sudo umount mountpoint
```


(4) Copy the root file system (rootfs)

• Format the partition 2:

```
sudo mkfs.ext4 /dev/sdb2
```

• Copy the target file system to the partition 2:

```
sudo mount /dev/sdb2 mountpoint
```

• Extract a rootfs package to a directory:

```
mkdir rootfs
sudo mount -o loop -t ext4 fsl-image-validation-imx-imx6qsabresd.ext4 rootfs
```

• Copy the rootfs files to the mountpoint:

```
cd rootfs/
sudo cp -a * /home/xu/mountpoint/
umount /home/xu/mountpoint
sudo umount /home/xu/rootfs
sync
```

如果umount出现device is busy，可以强制卸载：sudo umount -l /home/xu/rootfs

至此Linux image已经刷到SD card里面。

### 4. Boot up system by HDMI

在系统启动过程中：

```
Hit any key to stop autoboot:  0
u-boot> setenv mmcargs 'setenv bootargs console=ttymxc0,115200 root=/dev/mmcblk2p2 rootwait rw video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24'
u-boot> saveenv
u-boot> boot
```

console，root参数可以在系统的启动日志里可以看到：\\
Kernel command line: console=ttymxc0,115200 root=/dev/mmcblk2p2 rootwait rw \\
所以我们只需要在这个后面添加：
video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24

>参考资料：
>i.MX Linux® User's Guide

