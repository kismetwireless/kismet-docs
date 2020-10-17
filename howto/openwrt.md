---
title: "Building Kismet for OpenWRT"
permalink: /docs/howto/openwrt/
toc: true
---

# Building Kismet for OpenWRT 

## Kismet on OpenWRT

Thanks to the hard work of [Foxtrot](https://twitter.com/justfoxtrot) at Hak5, there are now official Kismet Makefiles for OpenWRT, and the packages are already included in the repositories for the Wi-Fi Pineapples!

For building in generic OpenWRT instances, read on...

## Kismet needs a lot of resources

Kismet tracks a *lot* of information about different networks and devices, which takes a lot of RAM.  Most OpenWRT targets have very limited resources for flash and RAM and are not best suited for running Kismet.  Usually, these devices are best suited for running the Kismet remote capture code, feeding packets to a full Kismet server on 'real' hardware.

It is possible to run a full Kismet server, and the package scripts provided attempt to disable the most memory-consuming aspects of Kismet while retaining functionality, however most systems running OpenWRT will run into limitations.

## Get the OpenWRT code

Check out a recent version of the OpenWRT/LEDE codebase:

```bash
$ git clone https://github.com/openwrt/openwrt.git
```

## Get the Kismet packaging code

```bash
$ git clone https://github.com/kismetwireless/kismet-packages.git
```

## Install the OpenWRT feeds

We need to tell OpenWRT to pull the feeds into the build system.  The feeds system includes many packages Kismet needs to compile.  Change to the OpenWRT directory you checked out, and run:

```bash
$ cd openwrt
$ ./scripts/feeds update -a
$ ./scripts/feeds install -a
```

This will download all the third-party package definitions.  They'll be needed by the Kismet packages.

## Copy or link the Kismet packages

Copy, or symlink, the Kismet package definitions.  From the `openwrt` directory you should already be in:

```bash
$ cp -R ../kismet-packages/openwrt/kismet-openwrt packages/
```

This assumes you checked out the kismet-packages repository in the same directory that  you checked out OpenWRT; if you used a different directory, of course copy from there instead.

## Configure OpenWRT

You will need to configure your basic options for OpenWRT, such as the processor and board.  This needs to match the processor of the target system you are building packages for.

```bash
$ make menuconfig
```

## Enable Kismet

Now we need to enable the Kismet package.  In the OpenWRT config tool still:

1. Navigate to `Network` 
2. Scroll to `kismet`.
3. Inside the `kismet` option will be many possible sub-packages.  Enable the packages you need *as modules*.

## Kismet packages

Kismet is split into multiple packages to minimize the storage required; you can pick and choose what sub-packages you need:

* `kismet` 

    The base Kismet server and WebUI.  This is required for running a full Kismet instance, and is the most resource intensive.  Many OpenWRT systems are not suitable for running a Kismet server, and are better suited running the remote capture tools.

    To capture packets, you will need one or more of the `kismet-capture-xyz` packages.

* `kismet-manuf-database`

    The compressed Kismet manufacturer database allows Kismet to resolve MAC addresses to manufacturers.  This package is only of use when running a full Kismet server.

* `kismet-icao-database`

    The compressed Kismet ICAO database is a very large dataset of ICAO aircraft registration.  This package is only of use when running a full Kismet server, and using the ADSB SDR capture tool.

* `kismet-capture-linux-wifi`

    The Linux Wi-Fi capture driver for Kismet; this provides both *local* capture for a full Kismet server install, and *remote* capture for using an OpenWRT device as a capture node.

    This is required to capture Wi-Fi packets on OpenWRT.

* `kismet-capture-linux-bluetooth`

    The Linux HCI Bluetooth capture driver for Kismet; this provides both *local* capture for a full Kismet server install, and *remote* capture for using an OpenWRT device as a capture node.

    This is required to detect Bluetooth networks, and requires a HCI Bluetooth adapter.

* `kismet-capture-sdr-rtladsb`

    The ADSB capture driver for Kismet; this provides both *local* capture for a full Kismet server install, and *remote* capture for using an OpenWRT device as a capture node.

    This is required to detect aircraft via ADSB, and requires a RTLSDR radio.  This also requires python3 support, and can be quite resource intensive.

* `kismet-capture-sdr-rtlamr`

    The AMR capture driver for Kismet; this provides both *local* capture for a full Kismet server install, and *remote* capture for using an OpenWRT device as a capture node.

    This is required to detect utility meters over AMR, and requires a RTLSDR radio.  This also requires python3 support, and can be quite resource intensive.

* `kismet-capture-sdr-rtl433`

    The 433Mhz switch, sensor, thermometer, and related capture driver for Kismet; this provides both *local* capture for a full Kismet server install, and *remote* capture for using an OpenWRT device as a capture node.

    This is required to detect multiple types of devices which can be seen with the rtl433 tool, including weather stations, TPMS sensors, thermometers, switches, and similar, and requires a RTLSDR radio.  This also requires python3 support, and can be quite resource intensive.

* `kismet-tools`

    Assorted Kismet host tools for log manipulation and conversion; these are typically better run on a full system with more resources.

* Assorted Python3 libraries

    Some Kismet capture drivers use Python3 modules not currently present in normal OpenWRT feeds; package definitions for these modules are included in the `kismet-openwrt` packaging directory and are automatically enabled when one of the Kismet python3-based capture tools is selected.

## Compile OpenWRT

Now we need to start the build process:  It will take a while.

```
$ make
```

Depending on how many processors your system has, you can speed this up with

```
$ make -j$(nproc)
```

or similar.

## Copy the packages!

If everything went well, you now have a bunch of packages to copy (or a firmware image to flash).  They can be found in `build_dir/target_[your target processor]`.  Remember you may need to copy many sub-packages to your device as well!


