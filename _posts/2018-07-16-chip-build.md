---
layout: default
title: "Build image for C.H.I.P"
date:  2018-07-16 22:20:00 -0600
---
# Build images for C.H.I.P

## Build and flash uboot:

Checkout u-boot source code from following repository:  
git https://github.com/NextThingCo/CHIP-u-boot.git  

And revision c2d284fbba74083eed8ae853a10f665f6febfdf1 is what I used to build.

Download Linaro tool chain:  
gcc version 4.9.2 20140904 (prerelease) (crosstool-NG linaro-1.13.1-4.9-2014.09 - Linaro GCC 4.9-2014.09)

Build:  
    make CHIP_defconfig
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- SUNXI_NAND_OOB_SIZE=1280 V=1  -j5

Place C.H.I.P in FEL mode by shorting FEL pin to GND, and then power it up. "sunxi-fel" can be built from repository https://github.com/linux-sunxi/sunxi-tools.git.

FEL boot target:  
    ./sunxi-fel uboot ./u-boot-sunxi-with-spl.bin write 0x43000000 sunxi-spl-with-ecc.bin


C.H.I.P should now boot to u-boot. Connect to C.H.I.P serial port use minicom (serial port setup: 1152008n1)

At u-boot command line, use following commands to flash SPL to NAND:

    nand erase.chip     <== Only erase from 0x000000 to 0x400000 if other parttions need to be reserved.
    nand write.raw.noverify 0x430000000 0x0 0xc4

Note:  
    0xc4, write size = filesize/(pagesize + oobsize)     0xc4 = 3462144/(16384+1280)

FLASH U-BOOT to NAND:

First FEL boot target:  
    ./sunxi-fel uboot ./u-boot-sunxi-with-spl.bin write 0x43000000 u-boot-dtb.img

Then at u-boot command line:  
    nand erase.spread 0x800000 0x400000
    nand write 0x43000000 0x800000 0x400000

## Build kernel/rootfs from yocoto

## Set usb net.

## Boot kernel by tftp
    setenv bootargs "root=/dev/sda1 rootwait rw"
    tftpboot 0x42000000 /zImage;tftpboot 0x43000000 /sun5i-r8-chip.dtb; tftpboot 0x44000000 /initrd
    bootz 0x42000000 - 0x43000000

