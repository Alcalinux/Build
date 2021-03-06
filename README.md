# Build

Example usage:
```sh
#!/bin/sh
set -e -v

# Configuration variables
export ARCH=arm ARM_ARCH=armv7-a
export HOST="$(printf '%s\n' "$MACHTYPE" | sed "s/-[^-]*/-cross/")"
export TARGET=arm-linux-musleabihf FLOAT=hard FPU=vfpv4

export ROOT="$(pwd)"
export DOWNLOADS="$(pwd)/src" TOOLS="$(pwd)/tools" BUILD="$(pwd)/build"
mkdir -p "$DOWNLOADS" "$TOOLS" "$BUILD"

export DEVICE=sun8i-h3-orangepi-one DEFCONFIG=sunxi
export BOOTARGS="loglevel=1 cma=64M root=/dev/mmcblk0p1 rootwait"
#export OVERLAYS="$(pwd)/overlays"

export UBOOT_DEFCONFIG=orangepi_one UBOOT_IMAGE=u-boot-sunxi-with-spl.bin

export PATH="$PATH:$(pwd)/scripts"

# If you need crosstools
scripts/meta/tools
export PATH="$PATH:$TOOLS/bin"

# Base packages
scripts/base/musl-*
scripts/base/busybox-*
scripts/base/linux-*
scripts/base/uboot-*

# Formating the SD card
printf 'o\n n\n p\n 1\n 2048\n \n a\n w\n' | fdisk /dev/mmcblk0
mkfs.ext4 "/dev/mmcblk0p1"

# Installing the packages
mkdir -p .mnt .info
mount "/dev/mmcblk0p1" .mnt

chateau -r .mnt -i .info add "$BUILD/filesystem.tar.xz"
chateau -r .mnt -i .info add "$BUILD/network.tar.xz"
chateau -r .mnt -i .info add "$BUILD/linux-"*.tar.xz
chateau -r .mnt -i .info add "$BUILD/musl-"*.tar.xz
chateau -r .mnt -i .info add "$BUILD/busybox-"*.tar.xz

cp scripts/chateau .mnt/bin/
mv .info .mnt/etc/packages

umount .mnt && rm -rf .mnt

# Flashing u-boot
dd if="$BUILD/u-boot.bin" of=/dev/mmcblk0 bs=1024 seek=8
```
