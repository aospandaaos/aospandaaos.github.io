[Back to projects overview](projects.md)

# Project: Building AOSP for the Raspberry Pi 4B

Objectives:

* get AOSP and build images for the Raspberry Pi 4B
* set up the hardware
* test that it all works

This project is based on a fork of Glodroid, <https://glodroid.github.io>

General procedure:
* get vanilla AOSP
* get board support package for Raspberry Pi 4
* build images
* create a recovery micro SD card
* boot the Raspberry Pi using the recovery card
* use fastboot to flash the images


## Update and/or install packages for Ubuntu

The build uses mcopy from the mtools package to create images for U-Boot.
Use this command to install it:
```bash
$ sudo apt install mtools
```

The build requires meson version >= 1.0; Ubuntu 20.04 only has 0.53:
```bash
$ meson --version
0.53.2
```
So, in this case it needs to be updated:

```bash
$ sudo apt remove meson
$ sudo pip install meson
$ meson --version
1.0.0
```

## Get AOSP
You need to set up a build server as described [here](build-server.md)	

For the examples below I am using build tag android-13.0.0_r11
*I highly recommend you use this particular version*

```bash
$ mkdir ~/aosp
$ cd ~/aosp
$ repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r11
$ mkdir .repo/local_manifests
$ git clone https://github.com/aospandaaos/a3m-rpi-device.git .repo/local_manifests
$ repo sync -c
```

## Build AOSP for RPi 4

There are two lunch targets:

* rpi4-userdebug: Android tablet
* rpi4car-userdebug: Android Automotive OS

```bash
$ cd ~/aosp
$ source build/envsetup.sh
$ lunch rpi4-userdebug    # or 'lunch rpi4car-userdebug' for automotive
$ m images
```

The 'images' make target is implemented in device/glodroid/platform/tools/tools.mk.
The output is out/target/product/rpi4/images.tar.gz, which contains:

```bash
adb
fastboot
mke2fs
bootloader-sd.img
env.img
flash-sd.sh
deploy-sd.img
boot.img
boot_dtbo.img
vendor_boot.img
super.img
vbmeta.img
vbmeta_system.img
deploy-gpt.img
```


## Use fastboot to copy images to Pi

### First time: create a recovery SD Card

```bash
$ croot
$ mkdir images
$ cd images
$ tar xf ../out/target/product/rpi4/images.tar.gz
```

Use the Raspberry Pi Imager (or a similar program) to copy deploy-sd.img to a
micro SD card

Plug the micro SD card into the Raspberry Pi and power on. It should power up in
fastboot mode, which you can show with
```bash
$ fastboot devices
10000000650b1601	 Android Fastboot
```

Then use the script to flash the images:
```bash
$ ./flash-sd.sh
```

The Rasbperry Pi should boot up into Android tablet or automotive depending on
your lunch choice


### Subsequent times

There is no need to write deploy-sd.img to the micro SD card each time.
From a running Android, run

```bash
$ adb reboot bootloader
```

Check that fastboot is installed and working (you need to have run lunch previously)
```bash
$ fastboot devices
10000000650b1601	 Android Fastboot
```

Then use the script to flash the images:
```bash
$ ./flash-sd.sh
```

