---
title: "Kismet on OSX"
permalink: /docs/howto/osx/
toc: true
---

# Compiling Kismet on OSX

Kismet is capable of running on OSX and capturing data on OSX natively.

## Building Kismet

Kismet should build on OSX directly, but requires some libraries be installed.

1. Install XCode from the Apple App Store.  You may be prompted, during the course of installing MacPorts, to install other components of XCode and the XCode command-line utilities. 

2. Install a ports library; for example:

   * MacPorts from https://www.macports.org
   * HomeBrew from https://brew.sh/

3. Install the needed external libraries; if prompted to install other necessary libraries or tools, of course say `yes`:

   * For `macports`:

     `$ sudo port install libmicrohttpd pcre protobuf-c protobuf-cpp libusb`

   * For `brew`:

     `$ brew install libmicrohttpd pcre protobuf protobuf-c libusb`

4. Install any desired external tools; to capture sensors using a rtl-sdr USB device and the `rtl_433` tool, you will need a USB device, the `librtlsdr` library, and the `rtl_433` tool set up.  You should be able to follow the guides on https://www.rtl-sdr.com for more information.

5. Install python2 and `pip`.  `pip` is the Python package manager; depending on your configuration it may be called `py-pip`; for example:

   * For `macports`:
     ```bash
     $ sudo port install py-pip
     ```

   Kismet uses Python to capture from `rtl_433` and for other plugin functions; installing `pip` will allow Python to fetch additional libraries automatically.

6. Make a source directory for Kismet (optional, but recommended)
   ```bash
   $ mkdir src`
   ```

5. Get the Kismet code by running, from within your `src` directory, the following:

   ```bash
   $ git clone https://www.kismetwireless.net/git/kismet.git
   ```

6. Configure Kismet.  You'll likely need to pass some options to tell the OSX compilers where to find the libraries and headers. From within your `kismet` directory, run:
   
   ```bash
   $ export CFLAGS="-I/opt/local/include" 
   $ export LDFLAGS="-L/opt/local/lib" 
   $ export CPPFLAGS="-I/opt/local/include" 
   $ ./configure
   ```

7. Compile Kismet.
   ```bash
   $ make
   ```

   There will be some warnings - generally they can be ignored.  As the OSX port evolves, the warnings will be cleaned up. 
   
   As with installation on Linux, you can accelerate the process by adding -j#, depending on how many CPUs you have.
   ```bash
   $ make -j4
   ```
   
8. Install Kismet
   ```bash
   $ sudo make suidinstall
   ```

   `make suidinstall` will install the Kismet helpers as suid-root, executeable by users in the `staff` group in OSX.  There is more information on the suidinstall method in the Kismet README; in general it increases the overall Kismet security by allowing you to launch Kismet as a normal user; only the packet capture tools will run as root.

## Configuring and Running Kismet

Kismet supports both local capture (from CoreWLAN / Apple wireless devices) and remote capture (from embedded Linux devices, etc, over the network).

Kismet will (currently) work *only* with Wi-Fi devices supported by the built-in Apple drivers; it will *not* work with USB devices; They use vendor drivers which do not support monitor mode or provide control APIs.

```bash
$ kismet
```

Kismet will list the available Wi-Fi sources in the `Data Sources` panel of the UI, or sources can be configured from the command line or the Kismet config files.

```bash
$ kismet -c en1
```

For more information on configuring Kismet in general, as well as logging formats and other Kismet features, be sure to check out the normal Kismet README file.

## Connect to Kismet

The Kismet web UI should be accessible on the OSX system by going to `http://localhost:2501` with your browser.

The first time you visit the Kismet UI you'll be prompted to create a password and log in.  

