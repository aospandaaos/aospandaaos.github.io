[Back to projects overview](projects.md)


# Project: Setting up a build server

## Hardware
Building AOSP is a non-trivial task. You need powerful hardware, running a recent Linux
distribution, with all the right packages installed. This link tells you what the
AOSP developers have to say on the topic:

[This link](https://source.android.com/source/initializing.html) tells you what the
AOSP developers have to say on the topic

In reality, you can build AOSP 14 with a minimum setup like this:

* 32 GB RAM
* 4 physical cores, 8 logical cores
* 500 GB disk

A machine like this will take 6 to 10 hours to build the images

If you are building images many times per day then you will need something more like this:

* 64 GB RAM
* 32 physical cores
* 500 GB fast SSD

Such a machine can build AOSP in an hour or two

You can can run the build locally on your own computer, or do it in the cloud.
Buying a decent specification machine is going to cost several thousand
Euros/Dollars.  Or you can rent a dedicated cloud instance for about one
hundred Euros/Dollars per month, or if you use on-demand services like Google
Cloud or AWS it will cost few Euros/Dollars per hour

Running Linux on bare metal will always be the fastest. Building in a VM,
either locally or in the cloud will slow things down by maybe a factor of two


## The operating system

The operating system must be Linux. In practice it is always Ubuntu, because
that is what the AOSP developers use. Generally, it's best to use a version of
Ubuntu that was current when AOSP was released. For AOSP 13 and 14, I use
Ubuntu 20.04

You need to install some packages:
```
$ sudo apt-get install git-core gnupg flex bison build-essential zip curl \
zlib1g-dev libc6-dev-i386 libncurses5 x11proto-core-dev libx11-dev lib32z1-dev \
libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```

The build requires Python 3. On Ubuntu 20.04, you can set python3 as default
with this command:
```
$ sudo apt install python-is-python3
```


## Setting up git

Make sure that you have configured git with your email and username:
```
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"
```


## Installing repo

AOSP uses the repo tool to manage the 1000 plus git repositories that make up AOSP.
Get a copy of repo:
```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > $HOME/bin/repo
$ chmod a+x $HOME/bin/repo
```


## General notes about downloading AOSP

First, choose a version of AOSP to download. You can see a complete list of
the versions released by the AOSP team here:

[https://source.android.com/docs/setup/reference/build-numbers](https://source.android.com/docs/setup/reference/build-numbers)

Generally I choose the latest stable release

Next, choose a directory for the AOSP source, e.g. ~/aosp:
```
$ mkdir ~/aosp
$ cd ~/aosp
```

Now you are ready to use repo to get the code for you

Repo works in two stages: first you download the manifest, then you synchronize
each of the git repositories listed in the manifest. The total download is
going to be about 160 GiB. This is how to get the manifest, using branch android-14.0.0_r28
as an example:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r28
```
Note: at this stage you can instruct repo to do a "shallow clone", which leaves
out the git history and cuts the total download down to about 100 GiB.
Here is the repo command to do that:
```
$ repo init --depth=1 -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r28
```
Whichever way you do it, repo init sets up the repo environment in (hidden)
directory ~/aosp/.repo.  The manifest is stored in ~/aosp/.repo/manifests/default.xml.
It contains a list of all of the git repositories that need to be cloned to get all the
source code

To get the source, you need to run the command repo sync
```
$ cd ~/aosp
$ repo sync -c
```

