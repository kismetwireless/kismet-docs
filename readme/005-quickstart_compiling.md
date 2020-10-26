---
title: "Compiling quickstart"
permalink: /docs/readme/quickstart/
excerpt: "Kismet has many many configuration knobs and options, but check here for the quickest way to get Kismet working with the latest release (or git version) and what you need to compile and do the initial configuration."
docgroup: "readme"
redirect_from:
  - /docs/readme/
  - /README-latest.html
---

## Compiling or packages?

Often, distributions lag behind software releases and may be offering older - sometimes significantly older - packages.  To get the latest, you can either install Kismet from [a package in the Kismet official repositories](/docs/readme/packages) or you can compile it from source.

If you're looking to make changes to the Kismet code, or if you're installing to a distribution not currently supported by the official Kismet packages, you'll definitely need to compile from source.

If you're installing onto a very resource constrained system, like a Raspberry Pi, you may want to consider either the packages, or making a cross compiling environment - modern C++ can be very resource intensive to compile, and a Raspberry Pi 3 or Raspberry Pi 0 is unlikely to be able to successfully compile natively.  The official packages for these environments are built on an Intel server with an emulated Pi docker environment to overcome these hurdles.

## Compiling: Quick Setup

Kismet has many configuration knobs and options; but for the quickest way to get the basics working:

1. Uninstall any existing Kismet installs.  

    If you installed Kismet using a package from your distribution, uninstall it the same way; if you compiled it yourself, be sure to remove it

2. Install dependencies.
    
    Kismet needs a number of libraries and  development headers to compile; these should be available in nearly all distributions.

   * *Linux Ubuntu/Debian/Kali/Mint*

       ```bash
       $ sudo apt install build-essential git libwebsockets-dev pkg-config zlib1g-dev libnl-3-dev libnl-genl-3-dev libcap-dev libpcap-dev libnm-dev libdw-dev libsqlite3-dev libprotobuf-dev libprotobuf-c-dev protobuf-compiler protobuf-c-compiler libsensors4-dev libusb-1.0-0-dev python3 python3-setuptools python3-protobuf python3-requests python3-numpy python3-serial python3-usb python3-dev librtlsdr0 libubertooth-dev libbtbb-dev
       ```

       On some older distributions, `libprotobuf-c-dev` may be called `libprotobuf-c0-dev`.

       For RTLSDR rtl_433 support, you will also need the (rtl_433 tool)[https://github.com/merbanan/rtl_433] if it is not already a package in your distribution.

   * *Linux Fedora (and related)*

       ```bash
       $ sudo dnf install make automake gcc gcc-c++ kernel-devel git libwebsockets-devel pkg-config zlib-devel libnl3-devel libcap-devel libpcap-devel NetworkManager-libnm-devel libdwarf libdwarf-devel elfutils-devel libsqlite3x-devel protobuf-devel protobuf-c-devel protobuf-compiler protobuf-c-compiler lm_sensors-devel libusb-devel fftw-devel
       ```

       You will also need the related python3, rtlsdr, and ubertooth packages.

   * *Other Linux distributions*

       Most distributions will have equivalent packages.  If your distribution splits binary and development packages, make sure to install both if you're compiling.

   * *OSX*

       OSX requires the XCode toolchain from the Apple store.  Once installed, you will need to launch the XCode IDE at least once to accept the license; do so before using the command line tools.

       You will need to install the `brew` tool from [brew.sh](https://brew.sh).  There are other package managers for OSX; feel free to use any of them which have the required packages, but Brew is known to work.

       Install the required packages via Brew:

       ```bash
       % brew install pkg-config python3 libpcap protobuf protobuf-c pcre librtlsdr libbtbb ubertooth libusb libwebsockets
       ```

       *2020-10* If you are using a Kismet *git version* from October 2020 or newer, you *do not need* libmicrohttpd.  However if you are attempting to compile *any Kismet release older than October 2020* you will need the following workaround for libmicrohttpd!

       *OSX libmicrohttpd warning* Currently, the latest version of libmicrohttpd (0.9.71) which is included in Brew appears to have a significant stalling issue, which will prevent Kismet from working properly.  This can be worked around, temporarily, by switching to the 0.9.63 version.  Unfortunately to do this requires several steps due to limitations of the `brew` system which have been imposed in recent versions.  This will also need to install several additional libraries and tools needed to compile libmicrohttpd, as it will not be able to install from a precompiled bottle.

       ```bash
       % brew unlink libmicrohttpd
       % git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core" fetch --unshallow
       % brew tap-new kismet/libmicrohttpd
       % brew extract --version 0.9.63 libmicrohttpd kismet/libmicrohttpd
       % brew install kismet/libmicrohttpd/libmicrohttpd@0.9.63
       ```

       Hopefully this workaround will not be needed in the future.

       If you have *already built* Kismet with the 0.9.71 version of libmicrohttpd on OSX, you will have to recompile Kismet.

3. Clone Kismet from git.  If you haven't cloned Kismet before:
    
    ```bash
    $ git clone https://www.kismetwireless.net/git/kismet.git
    ```

    If you have a Kismet repo already:

    ```bash
    $ cd kismet
    $ git pull
    ```

4. Run configure.  
    
    This will find all the specifics about your system and prepare Kismet for compiling.  If you have any missing dependencies or incompatible library versions, they will show up here.

    ```bash
    $ cd kismet
    $ ./configure
    ```

    Pay attention to the summary at the end and look out for any warnings! The summary will show key features and raise warnings for missing dependencies which will drastically affect the compiled Kismet.

   If you're compiling for a remote capture platform *only*, check the [remote capture docs](/docs/readme/datasources_remote_capture/) for more information.

5. Compile Kismet.
    
    ```bash
    $ make
    ```

    You can accelerate the process by adding `-j #`, depending on how many CPUs you have.  To automatically compile on all the available cores:

    ```bash
    $ make -j$(nproc)
    ```

    C++ uses quite a bit of RAM to compile, so depending on the RAM available on your system you may need to limit the number of processes you run simultaneously.

6.  Install Kismet.  

    Generally, you should install Kismet as suid-root; Kismet will automatically add a group and install the capture binaries accordingly.

    When installed suid-root, Kismet will launch the binaries which control the channels and interfaces with the needed privileges, but will keep the packet decoding and web interface running without root privileges.

    ```bash
    $ sudo make suidinstall
    ```

7.  Add your user to the `kismet` group (Linux)

    ```bash
    $ sudo usermod -aG kismet $USER
    ```

    This will add your current logged in user to the `kismet` group.

    On OSX, Kismet is installed under the `staff` group, which the default user is part of.

8.  Log out and back in.  

    Linux does not update groups until you log in; if you have just added yourself to the Kismet group you will have to re-log in.

9.  Check that you are in the Kismet group with:

    ```bash
    $ groups
    ```

    If you are not in the `kismet` group, you should log out entirely, or reboot.

10.  You're now ready to run Kismet!  
    
    Check out the [quick-start guide for running Kismet](/docs/readme/starting_kismet/) for more information!


