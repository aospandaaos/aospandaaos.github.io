---
layout: default
---

[Back to projects overview](projects.md)

# Project: Building Android Automotive OS for the VIM3 board

Objectives:

* build AAOS for VIM3 Pro
* test


This project builds on the information in the [VIM3 device project](device-vim3.md)

## Patching the amlogic "yukawa" device

You can find device support for the VIM3 in upstream ASOP, in directory
device/amlogic/yukawa. Out of the box it has configurations for tablet and TV.
This project patches that code to add a third option for Android Automotive

Begin by getting a copy of AOSP. In the example, I am using build tag
android-14.0.0_r28. It should work with other releases of android_14.0.0, but
this is the only one I have tested:
```bash
$ mkdir ~/aosp
$ cd ~/aosp
$ repo init --partial-clone -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r28
```
Note that --partial-clone reduces the size of the download. It is optional

Get the patches that add the Automotive configuration. In this example, they
will be stored in $HOME/vim3-aaos:
```bash
$ cd
$ git clone https://github.com/aospandaaos/vim3-aaos.git
```

Patch the yukawa device:
```bash
$ cd ~/aosp
$ cd device/amlogic/yukawa
$ git am ~/vim3-aaos/*.patch
```

Build the Automotive configuration:
```bash
$ cd ~/aosp
$ source build/envsetup.sh
$ lunch yukawa-userdebug
$ m TARGET_AUTOMOTIVE=true TARGET_VIM3=true
```

When the build has completed you can flash the images to your VIM3 board by
following the instructions for the [VIM3 device project](device-vim3.md),
starting from heading "Testing"

