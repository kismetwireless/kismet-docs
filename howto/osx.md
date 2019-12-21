---
title: "Kismet on OSX"
permalink: /docs/howto/osx/
toc: true
---

# Compiling Kismet on OSX

Kismet is capable of running on OSX and capturing data on OSX natively.

## Building Kismet

Kismet should build on OSX directly, but requires some libraries be installed.

1. Install XCode from the Apple App Store.   

    XCode is needed for the base compilers and build tools.
    
    Once installed, launch XCode at least once and accept the licenses.  You will need to do this to use the command line tools.

2. Install the 'brew' tool.

    Follow the directions at [brew.sh](https://brew.sh) to install the brew tool.

    There are other package managers for OSX; you're welcome to use any of them which have the required packages, but Brew is known to work.

3. Install the needed components

    ```bash
    $ brew install pkg-tool libmicrohttpd python3 libpcap protobuf protobuf-c pcre librtlsdr libbtbb ubertooth
    ```

4. Make a source directory for Kismet (optional, but recommended)

    ```bash
    $ mkdir src
    $ cd src
    ```

5. Get the Kismet code

    To get the latest version from git:

    ```bash
    $ git clone https://www.kismetwireless.net/git/kismet.git
    ```

6. Configure Kismet

    If you're using brew, the libraries and headers should be automatically detectable.  If you are using a different package manager, you may need to provide CFLAG, CXXFLAG, and LDFLAG environment variables.
   
    ```bash
    $ ./configure
    ```

7. Compile Kismet.

    ```bash
    $ make
    ```

    You can typically increase the speed of compiling by using multiple processors, for instance:

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

Since a Wi-Fi card cannot be in monitor mode and be connected to a network at the same time, you will notice that your mac disconnects from Wi-Fi while Kismet runs.  When you are done running Kismet, you can re-select your wireless network to reconnect.

## Connect to Kismet

The Kismet web UI should be accessible on the OSX system by going to `http://localhost:2501` with your browser.

The first time you visit the Kismet UI you'll be prompted to create a password and log in.  

## Errors during `./configure`

Errors during running `./configure` typically mean you have not installed a dependency Kismet needs, or you have installed it into a location that your system doesn't search by default.

The `brew` tool installs packages to `/usr/local/` by default, which should be in the default search path.  If you use other tools or other paths, you may need to specify the location manually to `configure` with the `CFLAGS`, `CXXFLAGS`, and `LDFLAGS` variables, for example:

```bash
$ CFLAGS=-I/opt/ports/include LDFLAGS=-L/opt/ports/lib -CXXFLASGS=-I/opt/ports/include ./configure ...
```

## Errors during `make`

* Error during linking about target x64

    This happens when MacOS or XCode has been updated between builds.  This can usually be fixed doing a clean build:

    ```bash
    $ make distclean
    $ ./configure ...
    $ make
    ```

