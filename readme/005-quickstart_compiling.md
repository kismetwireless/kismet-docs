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
       $ sudo apt install build-essential git libwebsockets-dev pkg-config zlib1g-dev libnl-3-dev libnl-genl-3-dev libcap-dev libpcap-dev libnm-dev libdw-dev libsqlite3-dev libprotobuf-dev libprotobuf-c-dev protobuf-compiler protobuf-c-compiler libsensors4-dev libusb-1.0-0-dev python3 python3-setuptools python3-protobuf python3-requests python3-numpy python3-serial python3-usb python3-dev python3-websockets librtlsdr0 libubertooth-dev libbtbb-dev
       ```

       On some older distributions, `libprotobuf-c-dev` may be called `libprotobuf-c0-dev`.

       For RTLSDR rtl_433 support, you will also need the [rtl_433 tool](https://github.com/merbanan/rtl_433) if it is not already a package in your distribution.

       On some older distributions, `libwebsockets` may not be available as a modern version.  Kismet uses the libwebsockets async API which was introduced a year ago, but some distributions still may not provide it.  You can try to compile libwebsockets yourself, or you can disable libwebsockets in the Kismet build with `--disable-libwebsockets` in the configure stage below. 

       Libwebsockets is used by the remote capture code; compiling without it will not remove websockets from the Kismet server, or prevent using websockets, but any remote capture code compiled without libwebsockets will only be able to use the legacy TCP connection mode.  If you're not planning to use remote capture nodes, none of this matters to you, and you can [get more info about remote capture here](/docs/readme/datasources_remote_capture).

   * *Linux Fedora (and related)*

       ```bash
       $ sudo dnf install make automake gcc gcc-c++ kernel-devel git libwebsockets-devel pkg-config zlib-devel libnl3-devel libcap-devel libpcap-devel NetworkManager-libnm-devel libdwarf libdwarf-devel elfutils-devel libsqlite3x-devel protobuf-devel protobuf-c-devel protobuf-compiler protobuf-c-compiler lm_sensors-devel libusb-devel fftw-devel
       ```

       You will also need the related python3, rtlsdr, and ubertooth packages.

   * *Other Linux distributions*

       Most distributions will have equivalent packages.  If your distribution splits binary and development packages, make sure to install both if you're compiling.

   * *MacOS*

       MacOS requires the XCode toolchain from the Apple store.  Once installed, you will need to launch the XCode IDE at least once to accept the license; do so before using the command line tools.

       You will need to install the `brew` tool from [brew.sh](https://brew.sh).  There are of course other package managers for MacOS; feel free to use any of them which have the required packages, but Brew is known to work.

       Install the required packages via Brew:

       ```bash
       % brew install pkg-config python3 libpcap protobuf protobuf-c pcre librtlsdr libbtbb ubertooth libusb openssl
       ```

       MacOS requires some additional flags during the `./configure` stage as well - be sure to read on in the configure section!

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

   On some platforms - such as MacOS - you may need to pass additional flags to `./configure`.

   * MacOS and Brew

       Brew does not place things where the compiler can find them by default, and does not install all development headers, to prevent conflict with MacOS system headers.  To work around this, extra options must be passed to the `./configure` stage:

       ```bash
       $ LDFLAGS="-L/opt/homebrew/Cellar/openssl\@1.1/1.1.1m/lib -L/opt/homebrew/Cellar/libusb/1.0.24/lib" CPPFLAGS="-I/opt/homebrew/Cellar/openssl\@1.1/1.1.1m/include -I/opt/homebrew/Cellar/libusb/1.0.24/include" ./configure --disable-libwebsockets
        ```

       This may not be necessary with other package managers under MacOS.

       If the libraries still can not be found with the above `./configure` command, you will need to examine your brew configuration to see if it is installing packages in a different location, or if a different version of the library has been installed.  In these cases, you will need to adjust the `LDFLAGS` and `CPPFLAGS` in the `./configure` command so that they point to the installed location and version of the libraries.

       The brew-supplied libwebsockets does not include the client-mode code, which is the only part Kismet actually needs.  It is necessary to disable client-mode websockets, this will only impact using remote capture over websockets *from* a MacOS system - websockets server support and receiving remote packets *as* a websockets server will work fine.

5. Compile Kismet.

    ```bash
    $ make
    ```

    You can accelerate the process by adding `-j #`, depending on how many CPUs you have.  To automatically compile on all the available cores:

    ```bash
    $ make -j$(nproc)
    ```

    C++ uses quite a bit of RAM to compile, so depending on the RAM available on your system you may need to limit the number of processes you run simultaneously.

    Sometimes, when updating the git repository, files have changed significantly enough that the Makefile system does not automatically recover fully.  If you encounter errors about missing header files (`foo.h not found` for example), try removing all `.d` files and running `make` again:

    ```bash
    $ rm *.d
    ```

    These files are used to identify which parts of the code need to be recompiled; rarely, when code is moved around, they get confused.

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

    On MacOS, Kismet is installed under the `staff` group, which the default user is part of.

8.  Reload your groups

    Groups are not updated automatically; you will need to reload the groups for your user.

    Either log back out and log in (or in extreme cases, reboot), or use the `newgrp` command if available:

    ```bash
    $ newgrp -
    ```

    Note that `newgrp` is *not* run via `sudo`.

9.  Check that you are in the Kismet group with:

    ```bash
    $ groups
    ```

    If you are not in the `kismet` group, you should log out and log back in, or reboot - some session and desktop managers don't reload the groups on logout, either.

10.  You're now ready to run Kismet!  
    
   Check out the [quick-start guide for running Kismet](/docs/readme/starting_kismet/) for more information!


