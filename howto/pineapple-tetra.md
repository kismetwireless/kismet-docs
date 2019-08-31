---
title: "Building Kismet for OpenWRT"
permalink: /docs/howto/pineapple_tetra/
toc: true
---

# Building Kismet-Git for OpenWRT 

## Kismet needs a lot of resources

Kismet tracks a *lot* of information about different networks and devices, which takes a lot of RAM.  Most OpenWRT targets have very limited resources for flash and RAM and are not best suited for running Kismet.  Usually, these devices are best suited for running the Kismet remote capture code, feeding packets to a full Kismet server on 'real' hardware.

## Get the OpenWRT code

Check out whatever version of OpenWRT/LEDE you're planning to use.

## Get the Kismet code

You'll want the Kismet source code to get the openwrt package definition.

```
$ git clone https://www.kismetwireless.net/kismet.git
```

## Do the basic OpenWRT Config

You will need to select the basic options for OpenWRT and enable the external feed for additional libraries Kismet needs.  When running `make menuconfig` you may see warnings about needing additional packages - install any that OpenWRT says you are missing.

```bash
# Go into the directory you just cloned
$ cd openwrt

# Start the configuration tool
$ make menuconfig
```

Inside the OpenWRT configuration you will want to:

1. Confirm that the correct platform is selected for whatever your target is.
2. Navigate to `Image Configuration`
3. Navigate to `Separate Feed Repositories`
4. Select `Enable feed packages`
5. Exit the config tool.  When prompted to save, do so.

## Install the feeds

We need to tell OpenWRT to pull the feeds into the build system.  Still in the openwrt directory you checked out, run:

```bash
$ ./scripts/feeds update -a
$ ./scripts/feeds install -a
```

This will download all the third-party package definitions.  They'll be needed by the Kismet packages.

## Copy the Kismet package definition

We want to copy the Kismet package over, because we'll potentially be making some modifications.  We assume you checked out the Kismet code into `~/src/`, if not, just substitute whatever path you used:

```
$ cp -R ~/src/kismet/packaging/openwrt/kismet-2019-pineapple ~/src/openwrt/package/network
$ cp -R ~/src/kismet/packaging/openwrt/kismet-remote-2019 ~/src/openwrt/package/network
```

## Install libprotoc-c

In a perfect world the libprotoc-c package in OpenWRT would install the proper host binary for protoc-c, but it does not.  Fortunately, there is only one version of libproto-c (the C-only version), so the package for your host distribution should be sufficient.

```
$ sudo apt-install protobuf-c-compiler
```


will suffice on Ubuntu-style distributions; your distribution may vary.  Note:  This is for the *protobuf-c* version, *not* the normal protobuf (which is C++, and which has a working OpenWRT package with proper host tools).

Newer versions of the OpenWRT build environment may fix this, but performing this step won't hurt.

## Enable Kismet

Now we need to enable the Kismet package.  Still in your OpenWRT directory:

1. Enter OpenWRT configuration again:  `make menuconfig`
2. Navigate to 'Network'.
3. Scroll all the way down to 'kismet', it will be several screens down.
4. Enable kismet-hak5 as a *module*.  Hit 'm' to do so.  If you have multiple kismet packages, make sure to select the right one - viewing the help on the entry will show you the version.
5. Exit, saving when prompted to do so.

To compile just the remote capture drivers (for smaller systems like the Nano or other very limited devices), select `kismet-remote-capture` as a module.

## Compile OpenWRT

Now we need to start the build process:  It will take a while.

```
$ make
```

Depending on how many processors your system has, you can speed this up with

```
$ make -j10
```

or similar.

## Copy the packages!

If everything went well, you now have a bunch of packages to copy (or a firmware image to flash).


