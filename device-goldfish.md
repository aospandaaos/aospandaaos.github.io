[Back to projects overview](projects.md)

# Project: Building AOSP for the Goldfish Emulator

Objectives:

* build a Phone image from AOSP
* test using the Android Goldfish emulator

The Goldfish emulator is the same emulator that you get with Android Studio to
test apps. You launch it with the 'emulator' executable, but the real work is
done by QEMU, almost always running in full system emulation mode with the help
of a hypervisor such as KVM on Linux or equivalent virtualisation drivers for
Windows or macOS

 
## Get AOSP
You need to set up a build server as described [here](build-server.md)	

For the examples below I am using build tag android-14.0.0_r28. Feel free to
experiment with other tags

```bash
$ mkdir ~/aosp
$ cd ~/aosp
$ repo init -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r28
$ repo sync -c
```

## Build AOSP for Goldfish

Set up the shell environment for the Android build system
```bash
$ cd ~/aosp
$ source build/envsetup.sh
```
Then use the 'lunch' command to select the target you want to build. For this
example, we want to build a phone emulator target that will run on an x86 PC,
so select sdk_phone64_x86_64-userdebug:

```bash
$ lunch sdk_phone64_x86_64-userdebug
```

You set the build running by typing 'm' (or 'make'). By default, 'm' will run
(nproc + 2) parallel jobs where nproc is the number of CPU cores, so an 8 core
machine, you would get 10 jobs. You may find that this causes it to run out of
memory, or overheat. You can override the default job calculation by adding
-j[number of jobs], for example 'm -j6' will run 6 parallel jobs

In this case, we will let the AOSP build system decide, so just type:
```bash
$ m
```
The build is likely to take several hours unless you have a really powerful machine

At the end you should see something like:
```bash
#### build completed successfully (02:20:44 (hh:mm:ss)) ####
```


### Testing using the Goldfish emulator

If the build server is local and has a display you can run the emulator right away:
```bash
$ cd ~/aosp
$ source build/envsetup.sh
$ lunch sdk_phone64_x86_64-userdebug
$ emulator
```

If, like me, you see nothing but a black window after several minutes you might
have a problem with the GPU drivers - I have an ancient Nvidia card which seems
not to be well supported - try this:
```bash
$ emulator -gpu guest
```

We will cover how to install an emulator build on other machines in a separate
project (which is still a work on progress, watch this space)

### ADB

You can use abd to talk to the emulator. Note that since this is a userdebug
build you can get to root via 'su'

```bash
$ adb shell
emu64x:/ $ su
emu64x:/ # id
uid=0(root) gid=0(root) groups=0(root),1004(input),1007(log),1011(adb), ...
```

### Kernel messages

You can see messages from Linux as it boots up by adding -show-kernel:
```bash
$ emulator -show-kernel
```


