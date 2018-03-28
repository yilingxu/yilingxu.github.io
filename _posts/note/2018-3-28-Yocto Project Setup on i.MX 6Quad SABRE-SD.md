---
layout: post
title: Yocto Project Setup on i.MX 6Quad SABRE-SD
description: How to setup Yocto Project
category: note
---

### 1. Setting Up Host Machine Packages

My host machine is Red Hat 4.8.2-16, Linux version 3.10.0-123.20.1.el7.x86_64  (官方推荐存储不少于120G)

Go to Yocto Project [Quick Start](https://www.yoctoproject.org/docs/current/ref-manual/ref-manual.html) and check for the packages that must be installed on the host machine


### 2. Set Up The Repo Utility

#### Create a bin folder in the home directory

```
 mkdir ~/bin
 curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
 chmod a+x ~/bin/repo
```

#### Add the following line to the .bashrc file

```
export PATH=~/bin:$PATH
```

### 3. Download Yocto Project

```
git config --global user.name zhangsan
git config --global user.email zhangsan@163.com
git config --list 
mkdir fsl-release-bsp
cd fsl-release-bsp
repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-morty -m imx-4.9.11-1.0.0_ga.xml
repo sync
```

等待同步完成，同步过程出错，可以用ctrl+c中断，然后执行repo sync继续同步。

### 4. Build Image:

```
DISTRO=fsl-imx-x11 MACHINE=imx6qsabresd source fsl-setup-release.sh -b build-x11
bitbake fsl-image-validation-imx
```

其中DISTRO有如下四种设置：\\
• fsl-imx-x11 : Only X11 graphics\\
• fsl-imx-wayland : Wayland weston graphics\\
• fsl-imx-xwayland : Wayland graphics and X11. X11 applications using EGL are not supported\\
• fsl-imx-fb : Frame Buffer graphics - no X11 or Wayland


MACHINE指的是板子的型号

-b 跟的是安装目录

bitbake可以编译以下多种镜像：\\
• core-image-minimal : A small image that only allows a device to boot\\
• core-image-base : A console-only image that fully supports the target device hardware\\
• core-image-sato : An image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme and uses Pimlico applications. It contains a terminal, an editor and a file manager\\
• fsl-image-machine-test : An FSL Community i.MX core image with console environment - no GUI interface\\
• fsl-image-validation-imx : Builds an i.MX image with a GUI without any Qt content\\
• fsl-image-validation-qt5-imx : Builds an opensource Qt 5 image. These images are only supported for i.MX SoC with hardware graphics. They are not supported on the i.MX 6UltraLite, i.MX 6UltraLiteLite, and i.MX 7Dual


bitbake的时间非常长，如果编译中途出错，可以用ctrl+c中断，然后重新执行 

```
bitbake fsl-image-validation-imx
```

如果编译过程中关掉终端，可以重新进入fsl-release-bsp文件夹，先执行 

```
source setup-environment build-x11
```

然后执行 

```
bitbake fsl-image-validation-imx
````

### 4. Image
编译成功后，在 ~/fsl-release-bsp/build-x11/tmp/deploy/images/imx6qsabresd 文件夹下存在如下文件：\\
sl-image-validation-imx-imx6qsabresd-20180323140502.rootfs.ext4\\
fsl-image-validation-imx-imx6qsabresd-20180323140502.rootfs.manifest\\
fsl-image-validation-imx-imx6qsabresd-20180323140502.rootfs.sdcard  
fsl-image-validation-imx-imx6qsabresd-20180323140502.rootfs.tar.bz2  
fsl-image-validation-imx-imx6qsabresd.ext4                            
fsl-image-validation-imx-imx6qsabresd.manifest                        
fsl-image-validation-imx-imx6qsabresd.sdcard                         
fsl-image-validation-imx-imx6qsabresd.tar.bz2                         
modules--4.9.11-r0-imx6qsabresd-20180323140502.tgz                   
modules-imx6qsabresd.tgz                                            
README_-_DO_NOT_DELETE_FILES_IN_THIS_DIRECTORY.txt                
u-boot.imx                                                       
u-boot-imx6qsabresd.imx                                           
u-boot-imx6qsabresd.imx-sd                                           
u-boot.imx-sd\\
u-boot-sd-2017.03-r0.imx\\
zImage\\
zImage--4.9.11-r0-imx6qsabresd-20180323140502.bin\\
zImage--4.9.11-r0-imx6q-sabresd-20180323140502.dtb\\
zImage--4.9.11-r0-imx6q-sabresd-btwifi-20180323140502.dtb\\
zImage--4.9.11-r0-imx6q-sabresd-enetirq-20180323140502.dtb\\
zImage--4.9.11-r0-imx6q-sabresd-hdcp-20180323140502.dtb\\
zImage--4.9.11-r0-imx6q-sabresd-ldo-20180323140502.dtb\\
zImage-imx6qsabresd.bin\\
zImage-imx6q-sabresd-btwifi.dtb\\
zImage-imx6q-sabresd.dtb\\
zImage-imx6q-sabresd-enetirq.dtb\\
zImage-imx6q-sabresd-hdcp.dtb\\
zImage-imx6q-sabresd-ldo.dtb

