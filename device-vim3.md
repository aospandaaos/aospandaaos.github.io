### [Back to index](README.md)

# Project: Building AOSP for the VIM3 dev board

Objectives:

* get AOSP and build images for the VIM3 dev board
* set up VIM3 hardware
* test that it all works


## The VIM3

The VIM3 is a dev board from Khadas: <https://www.khadas.com/vim3>

It is one of the boards supported by AOSP, so you can build images for VIM3
without needing any external code 

The board is available in basic and pro versions. The basic version is priced
at $105 and the pro $139. The pro has more RAM and flash storage so it is the
better choice for running Android, but either will work fine

The main features of the board are:

* CPU: Amlogic A311D 4 x 2.2GHz Cortex-A73 cores, 2 x 1.8GHz Cortex-A53 cores
* GPU: ARM Mali G52
* RAM: 2GB or (pro)4GB
* eMMC flash storage: 16GB or (pro)32GB
* Dual display: one HDMI and one MIPI-DSI
* WiFi: 802.11a/b/g/n/ac
* Bluetooth: 5.0
* Ethernet: 10/100/1000M
* USB: 2 x USB A 2.0; 1 x USB C OTG
* M.2 Slot for SSD & LTE module


## Hardware

You will need

* VIM3 board: <https://www.khadas.com/vim3>
* HDMI touchscreen. These usually come with a USB touch interface that you can
  plug into a USB A port on the VIM3
* Power supply for the touchscreen (!)
* HDMI cable
* Optional, but very useful, a 3V3 serial to USB cable, for example
  <https://www.adafruit.com/product/954>

(!) Make sure the touchscreen is not drawing power from the VIM3 USB port, 
which would then not have enough power to run. Touchscreens that are powered
via USB often have a second USB connector that you can wire to a USB charger

## Getting AOSP for VIM3
You need to set up a build server as described [here](build-server.md)	

Since the VIM3 board has device support in AOSP itself you just need to get a
vanilla copy of AOSP. For the examples below I am using build tag
android-14.0.0_r28. This is the only version that I have tested

```bash
$ mkdir ~/aosp
$ cd ~/aosp
$ repo init -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r28
$ repo sync -c
```

## Building
The board support for the VIM3 is in `aosp/device/amlogic/yukawa`. In fact
the yukawa directory has support for several boards based on Amlogic chips:
sei610, VIM3 and VIM3L. There is information about building images for yukawa
boards in `device/amlogic/yukawa/board-info/README`, and at
<https://gitlab.baylibre.com/baylibre/amlogic/atv/aosp/device/amlogic/yukawa/-/wikis/home>.
This project is based on that information

There are several Make variables that influence how the images are built:

| Variable                   | Default | Description                                            |
| -------------------------- | ------- | ------------------------------------------------------ |
| `TARGET_VIM3`              | false   | true = build for VIM3 instead of sei610                |
| `TARGET_VIM3L`             | false   | true = build for VIM3L instead of sei610               |
| `TARGET_KERNEL_USE`        | 5.10    | Pre built kernel version to use                        |
| `TARGET_AVB_ENABLE`        | false   | true = use AVB                                         |
| `TARGET_USE_AB_SLOT`       | false   | true = enable AB partitions                            |
| `TARGET_BUILTIN_EDID`      | false   | true = preload 1920x1080 EDID structure                |
| `TARGET_USE_TABLET_LAUNCHER` | false   | true = use tablet launcher instead of Android TV launcher     |

There are prebuilt kernels and device tree binaries in `aosp/device/amlogic/yukawa-kernel`. 
You can select the kernel version to be one of 4.19, 5.4, 5.10, 5.15 or 6.1 by
setting Make variable `TARGET_KERNEL_USE=x.y` The default is 5.10

So, to build VIM3 tablet with all other options default:
```bash
$ cd aosp
$ source build/envsetup.sh
$ lunch yukawa-userdebug
$ m TARGET_USE_TABLET_LAUNCHER=true TARGET_VIM3=true
```

To build VIM3 Android TV:
```bash
$ cd aosp
$ source build/envsetup.sh
$ lunch yukawa-userdebug
$ m TARGET_VIM3=true
```

## Testing
Now it's time to try it all out


### Granting USB permissions
The VIM3 needs permissions for three USB vendor product IDs. Edit
`/etc/udev/rules.d/51-android.rules` and add these lines:

```
# khadas vim3 initial flashing
SUBSYSTEM=="usb", ATTR{idVendor}=="1b8e", ATTR{idProduct}=="c003", MODE="0660", TAG+="uaccess"

# khadas vim3 fastboot (initial bootloader)
SUBSYSTEM=="usb", ATTR{idVendor}=="1b8e", ATTR{idProduct}=="fada", MODE="0660", TAG+="uaccess"

# khadas vim3 adb
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e40", MODE="0660", TAG+="uaccess"
```

When you have saved those changes restart udev:
```bash
$ sudo udevadm control --reload
$ sudo udevadm trigger
```

### (Optional) Serial console
A serial console is useful so that you can see the console output from the
bootloader and kernel. The pins for the serial port are on the 40 pin header,
which is documented here:
<https://docs.khadas.com/products/sbc/vim3/hardware/start#tab__gpio-pinout>

If you are using the <https://www.adafruit.com/product/954>, the connection
looks like this:
```
                  1 x x 21
                  2 x x 22
                  3 x x 23
                  4 x x 24
                  5 x x 25
                  6 x x 26
                  7 x x 27
                  8 x x 28
                  9 x x 29
                 10 x x 30
                 11 x x 31
                 12 x x 32
                 13 x x 33
                 14 x x 34
                 15 x x 35
                 16 x x 36
   black     GND 17 x x 37
   green      RX 18 x x 38
   white      TX 19 x x 39
                 20 x x 40
```

The serial port will be /dev/ttyUSB0, baudrate 15200. For example, if you are
using minicom, you would connect using:

```bash
$ minicom -D /dev/ttyUSB0 -b 115200
```

### Flashing images - first time
You need to install the correct version of the U-Boot bootloader. Looking in
directory `aosp/device/amlogic/yukawa/bootloader` you will see binary versions
of u-boot for all the yukawa dev boards: we need `u-boot_kvim3_ab.bin`

First, put VIM3 in USB Upgrade MODE by:

* Powering-on the VIM3
* Quickly pressing the F button 3 times in 2 seconds

You will see the Power-LED (Blue) blink for about 3 seconds. After the Power-LED
(Blue) turns OFF, this indicates that the board is in upgrade mode. This is
quite tricky and will probably take several attempts

If you have a serial console you should see something like this:
```
bl2_stage_init 0x81
hw id: 0x0000 - pwm id 0x01
bl2_stage_init 0xc1
bl2_stage_init 0x02

[...]

emmc switch 1 ok
Load FIP HDR from eMMC, src: 0x00010200, des: 0x01700000, size: 0x00004000, part: 1
Load BL3X from eMMC, src: 0x00078200, des: 0x01768000, size: 0x000b1400, part: 1
G12B:BL:6e7c85:2a3b91;FEAT:E0F83180:402000;POC:D;RCY:0;USB:0
```

Next you will use the update tool to copy the u-boot binary. Update requires
libusb-0, so you may need to install it now. For Ubuntu:

```bash
$ sudo apt install libusb-0.1
```

Then

```bash
$ cd aosp/device/amlogic/yukawa/bootloader
$ ./tools/update write u-boot_kvim3_noab.bin 0xfffa0000 0x10000
..
Transfer Complete! total size is 65536 Bytes

$ ./tools/update run 0xfffa0000
[update]Run at Addr fffa0000
AmlUsbRunBinCode:ram_addr=fffa0000

$ ./tools/update bl2_boot u-boot_kvim3_noab.bin
[LUSB][AMLC]dataSize=16384, offset=65536, seq 0
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=49152, offset=393216, seq 1
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=16384, offset=229376, seq 2
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=49152, offset=245760, seq 3
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=49152, offset=294912, seq 4
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=16384, offset=65536, seq 5
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB][AMLC]dataSize=1110384, offset=81920, seq 6
[LUSB]requestType=0
[LUSB]before wait sum
[LUSB]check sum OKAY
[LUSB]BL2 END, waiting TPL plug-in...
```

When you get to this stage, power off the board and power on again. If you have
a serial console, you can see that u-boot has loaded and is running fastboot:

```bash
U-Boot 2021.07-00051-g79f19c6307 (Jul 12 2021 - 12:25:19 +0200) khadas-vim3

Model: Khadas VIM3
SoC:   Amlogic Meson G12B (A311D) Revision 29:b (10:2)
DRAM:  3.8 GiB
MMC:   sd@ffe03000: 0, sd@ffe05000: 1, mmc@ffe07000: 2

[...]

BCB: Bootloader boot...
Running Fastboot...
crq->brequest:0x0

```
Check that the VIM3 is listed by the `fastboot` command:

```
$ fastboot devices
C86314724D12	 Android Fastboot
```

Now you can format the flash memory and install the images:

```
$ croot
$ fastboot oem format
$ fastboot flash bootloader device/amlogic/yukawa/bootloader/u-boot_kvim3_noab.bin
$ fastboot erase bootenv
$ fastboot reboot bootloader
$ fastboot flash boot
$ fastboot flash super        # this takes a few minutes, be patient
$ fastboot flash cache
$ fastboot flash userdata
$ fastboot flash recovery
$ fastboot flash dtbo $OUT/dtbo-unsigned.img
$ fastboot reboot
```

It should boot into Android. The fist boot takes a couple of minutes, subsequent
boots are more like 40 seconds

### Flashing images the second and subsequent times

If you have already installed Android and want to load new images, you can
skip updating the firmware. Boot from Android into U-Boot/fastboot:

```
adb reboot bootloader
```

Then you can format the flash memory and install the images as above

