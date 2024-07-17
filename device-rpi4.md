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
$ git clone https://github.com/aospandaaos/a3m-rpi-manifest.git .repo/local_manifests
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


## Hardware
* Raspberry Pi 4B with 4GB or 8GB(recommended) RAM
* Decent quality micro SD card, ideally class A1 or A2
* Touch screen, such as the Elecrow 7" or 10" HDMI panel with USB touch interface

Connect the HDMI interface on the screen to the Raspberry Pi micro HDMI port

Connect the USB touch interface to one of the USB A ports on the Raspberry Pi

Connect a USB cable from your computer to the USB C port on the Raspberry Pi

### Power considerations
In order that ADB and fastboot work over USB, we need to use the USB C port on
the Raspberry Pi to connect to your computer. In this configuration your computer is powering the
Raspberry Pi, which will quite likely exceed the current available vi USB. The
best solution is to power the Pi via GPIO, using a 5V 3A source. A workaround
that seems to work OK for me is to power the touch screen via it's own USB power
port. Also, remember that the current limit on PC USB connectors is different
for different USB connector types (I am simplifying things a lot here: read the
USB specs for full details):

USB A 2.0: 500mA

USB A 3.x: 900 mA

USB C: 1.5 A

YMMV


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

## ADB
Once the Pi has booted Android, you can connect using ADB over the USB cable:

```bash
$ adb devices
List of devices attached
10000000650b1601	device

$ adb shell
rpi4:/ $ 
```
