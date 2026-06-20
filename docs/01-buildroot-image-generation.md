# Building a Complete Embedded Linux System with Buildroot

## Introduction

This document describes the process of generating a complete Linux image for the BeagleBone Black using Buildroot.

In this approach, Buildroot is responsible for generating the complete embedded Linux environment, including:

- Cross compilation toolchain
- U-Boot bootloader
- Linux Kernel
- Device Tree
- BusyBox
- Root filesystem
- SD card image


The main advantage is reproducibility. After configuring the project, the complete Linux image can be generated simply by running:

```bash
make
```

## 1. Download Buildroot

Download Buildroot:

```bash
wget https://buildroot.org/downloads/buildroot-2025.02.tar.xz
```

Extract:

```bash
tar xf buildroot-2025.02.tar.xz
```

Enter the source directory:

```bash
cd buildroot-2025.02
```

## 2. Clean Previous Configuration

If Buildroot was previously configured using an external toolchain, clean the project:

```bash
make distclean
```

This removes previous generated configurations.

## 3. Configure BeagleBone Black

Buildroot provides predefined configurations.

Search for BeagleBone support:

```bash
make list-defconfigs | grep beaglebone
```

Load the default configuration:

```bash
make beaglebone_defconfig
```

## 4. Enable Buildroot Internal Toolchain

Open the configuration menu:

```bash
make menuconfig
```

Go to:

```
Toolchain
```

Select:

```
Toolchain type -> Buildroot toolchain
```

With this option, Buildroot will automatically generate the ARM cross compiler.

No external toolchain is required.

## 5. Configure Target Architecture

Inside:

```
Target options
```

Configure:

```
Target Architecture:
ARM (little endian)

Target Architecture Variant:
cortex-A8
```

The BeagleBone Black uses the TI AM335x SoC with an ARM Cortex-A8 processor.

## 6. Build the Linux Image

Start the build:

```bash
make -j$(nproc)
```

Buildroot will automatically build:

- Cross compiler
- U-Boot
- Linux Kernel
- Device Tree
- BusyBox
- Root filesystem
- SD card image

The first build can take some time because the complete environment is generated.

## 7. Generated Files

After the build:

```bash
ls -l output/images/
```

Example:

```
MLO
u-boot.img
zImage
am335x-boneblack.dtb
rootfs.ext4
rootfs.tar
sdcard.img
uEnv.txt
```

Important files:

| File | Description |
|------|-------------|
| MLO | U-Boot SPL |
| u-boot.img | U-Boot bootloader |
| zImage | Linux Kernel |
| am335x-boneblack.dtb | Device Tree |
| rootfs.ext4 | Root filesystem |
| uEnv.txt | U-Boot environment |
| sdcard.img | Complete SD card image |

## 8. Flashing the SD Card using Balena Etcher

The generated image:

```
output/images/sdcard.img
```

contains the complete SD card layout.

Use Balena Etcher to write the image.

Download:

https://etcher.balena.io/

Steps:

1. Open Balena Etcher
2. Select `sdcard.img`
3. Select the SD card
4. Start flashing

After flashing, insert the SD card into the BeagleBone Black.

## 9. Necessary Customizing in U-Boot Environment

The generated U-Boot environment comes from:

```
board/beagleboard/beaglebone/uEnv.txt
```

It is necessary to include one command to make it work.
Modify this file, by adding the following command required by `uenvcmd` before `set_mmc1`:

```ini
loadimage=load mmc ${bootpart} ${loadaddr} ${bootdir}/${bootfile}
```

Without this variable, U-Boot will fail with:

```
## Error: can´t get kernel image!
```

After changing the file, rebuild:

```bash
make -j$(nproc)
```

The generated file will be updated:

```
output/images/uEnv.txt
```

Flash new image in the board, Linux should boot correctly now. User is root and there is not password in this image.