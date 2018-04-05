---
layout: post
title: OP-TEE on i.MX 6Quad SABRE-SD
description: How to flash OP-TEE on I.MX 6Quad SABRE-SD
category: note
---
### 1. Partition the SD card

1.1 Umount SD card:  (my SD card name is sdb)
```
sudo umount /dev/sdb
sudo umount /dev/sdb1
sudo umount /dev/sdb2
```

1.2 Partition:

```
sudo fdisk /dev/sdb
p [lists the current partitions]
d [to delete existing partitions. Repeat this until no unnecessary partitions are reported by the 'p' command to start fresh.]
n [create a new partition]
p [create a primary partition - use for both partitions]
1 [the first partition]
20480 [starting at offset sector]
1208320 [size for the first partition to be used for the boot images]
p [to check the partitions]
n
p
2
1228800 [starting at offset sector, which leaves enough space for the kernel, the bootloader and its configuration data]
<enter> [using the default value will create a partition that extends to the last sector of the media]
p [to check the partitions]
w [this writes the partition table to the media and fdisk exits]
```
```
sudo mkfs.vfat /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
```
I will put the rootfs in sdb2 later. I do not put anything in sdb1, because the zImage, imx6q-sabresd.dtb and uTee will be downloaded from a TFTP server into RAM directly under u-boot. 

### 2. Download OP-TEE OS

2.1 Download the repo from storage.googleapis.com:
```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

2.2 Download OP-TEE:
```
cd ~/op-tee
mkdir optee
cd optee
repo init -u https://github.com/OP-TEE/manifest.git -m juno.xml
repo sync
````

### 3. Download compiler for OP-TEE OS

```
cd ~/op-tee/optee
mkdir toolchains
cd toolchains
wget https://releases.linaro.org/components/toolchain/binaries/4.9-2016.02/arm-linux-gnueabihf/gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf.tar.xz
tar xvf gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf.tar.xz
export PATH=/home/xu/op-tee/optee/toolchains/gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf/bin:$PATH
```

### 4. Compile U-boot for OP-TEE

4.1 Download u-boot:
```
cd ~/op-tee
$ git clone https://github.com/MrVan/u-boot.git -b imx_v2016.03_4.1.15_2.0.0_ga
```

4.2 Compile u-boot:
```
cd ~/op-tee/u-boot
vim board/freescale/mx6sabresd/mx6sabresd.c   //gd->ram_size = imx_ddr_size() - SZ_32M;
make ARCH=arm mx6qsabresd_defconfig
CROSS_COMPILE=arm-linux-gnueabihf- make ARCH=arm
```

4.3 Flash u-boot.imx to offset 0x400 of SD card:
```
sudo dd if=u-boot.imx of=/dev/sdb bs=512 seek=2 conv=sync conv=notrunc
```

### 5. Compile Linux for OP-TEE

5.1 Download kernel:
```
cd ~/op-tee
git clone https://github.com/MrVan/linux.git -b imx_4.1.15_2.0.0_ga
```

5.2 Compile kernel:
```
cd ~/op-tee/linux
make ARCH=arm imx_v7_defconfig
CROSS_COMPILE=arm-linux-gnueabihf- make ARCH=arm
vim .config    //set CONFIG_FHANDLE=y 
CROSS_COMPILE=arm-linux-gnueabihf- make ARCH=arm
```
We do not have to flash the zImage and imx6q-sabresd.dtb to SD card now, we can use tftp to download them to RAM directly under u-boot

### 6. Compile OP-TEE OS

```
cd ~/op-tee/optee/optee_os
CROSS_COMPILE=arm-linux-gnueabihf- make PLATFORM=imx-mx6qsabresd ARCH=arm CFG_BUILT_IN_ARGS=y CFG_PAGEABLE_ADDR=0 CFG_NS_ENTRY_ADDR=0x12000000 CFG_DT_ADDR=0x18000000 CFG_DT=y CFG_PSCI_ARM32=y DEBUG=y CFG_TEE_CORE_LOG_LEVEL=1 CFG_BOOT_SYNC_CPU=n CFG_BOOT_SECONDARY_REQUEST=y
cp ~/op-tee/u-boot/tools/mkimage ./     //recommend the use of mkimage from u-boot/tools/mkimage to make uTee image
./mkimage -A arm -O linux -C none -a 0x4dffffe4 -e 0x4e000000 -d out/arm-plat-imx/core/tee.bin uTee
```

use tftp to download uTee to RAM directly under u-boot

### 7. Compile OP-TEE Client

```
cd ~/op-tee/optee/optee_client
make ARCH=arm
```

Copy all file in out/export to the rootfs of the target board

### 8. Compile OP-TEE XTEST

```
cd ~/op-tee/optee/optee_test/
export TA_DEV_KIT_DIR=/home/xu/op-tee/optee/optee_os/out/arm-plat-imx/export-ta_arm32
export OPTEE_CLIENT_EXPORT=/home/xu/op-tee/optee/optee_client/out/export
export CROSS_COMPILE_HOST=arm-linux-gnueabihf-
export CROSS_COMPILE_TA=arm-linux-gnueabihf-
CROSS_COMPILE=arm-linux-gnueabihf- make ARCH=arm
```

copy all "xx.ta" "xx.elf" in out/* to rootfs "/lib/optee_armtz/" \\
copy "xtest" to rootfs "/bin" \\
copy rootfs to SD card (sdb2)

### 9. Boot OP-TEE On SABRESDB

Under the u-boot:
```
setenv serverip 192.168.10.4    //set your tftp serverip
setenv ipaddr 192.168.10.7    //set your board ip
setenv bootargs console=ttymxc0,115200 root=/dev/mmcblk2p2 rootwait rw
saveenv
tftp 0x12000000 zImage    //zImage will be loaded to 0x12000000
tftp 0x18000000 imx6q-sabresd.dtb    //dtb will loaded to 0x18000000
tftp 0x20000000 uTee    //uTee will be loaded to 0x20000000
bootm 0x20000000 - 0x18000000
```

### 10. Test OP-TEE

```
tee-supplicant &
xtest
```

>Reference:\\
[http://mrvan.github.io/optee-imx6q-sabresd](http://mrvan.github.io/optee-imx6q-sabresd)\\
[http://blog.lineo.co.jp/archives/629](http://blog.lineo.co.jp/archives/629)
