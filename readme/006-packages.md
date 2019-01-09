---
title: "Official Kismet packages"
permalink: /docs/readme/packages/
toc: true
---
There are automatically-built repositories for Kismet on several Linux distributions.  More are being added over time, and your distribution may already have modern packages (Pentoo, for instance).

These repositories contain the latest Kismet versions, which may not be available in the standard repositories for your distribution; distributions typically pick up new releases at relatively long intervals, and will not include git or beta versions in the official packages.

## Remove any installed Kismet
You will need to remove any installed Kismet code.

If you installed *from a Kismet package in your distribution*, remove it:

```bash
$ sudo apt remove kismet kismet-plugins
```

If you installed *Kismet from source* yourself, either the old Kismet or the new Kismet git or beta code, you will need to remove it.  Asssuming you installed it in the default location:

```bash
$ sudo rm -rfv /usr/local/bin/kismet* /usr/local/share/kismet* /usr/local/etc/kismet*
```

## Release or git
If you'd like to be on the cutting edge of testing, you can pull Kismet from nightly git builds.  These builds take the latest git version and compile it - this version has all the absolutely latest features, but also is the most likely to have new, exciting bugs.  The git version is *generally* fine to use, but is not recommended for installations that need consistency or long-term support.

The release version is build from the latest release tag, *or* the latest beta tag.  These versions are generally tagged to allow a consistent installation version from code that *should* be known-good.

## Configuration and locations
The Kismet packages install Kismet and the capture tools into `/usr/bin/`, and the configuration files into `/etc/kismet/`.

If you're used to compiling from source, these are new directories, which match the standard locations for system packages.

## Kali Linux (Intel, Raspberry Pi)
Kali Linux (on i386, amd64, armhf - Raspberry Pi 3, arm64 - Raspberry Pi 64bit, and armel - Raspberry Pi 0w)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/kali rolling main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/kali rolling main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

## Debian Stretch (Intel, Raspberry Pi)
Debian Stretch (i386, amd64, armhf - Raspberry Pi 3, armel - Raspberry Pi 0w)

*WARNING* - You will *not* be able to capture from the built-in Wi-Fi on the Raspberry Pi 3 or Pi 0w unless you also install the [nexmon driver patch](https://github.com/seemoo-lab/nexmon/).  This patch adds reverse-engineered monitor mode to the Broadcom driver.  You can still use USB devices, though!

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/stretch stretch main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/stretch stretch main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

## Ubuntu 16.04 Xenial (Intel)
Ubuntu 16.04 Xenial (i386, amd64)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/xenial xenial main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/xenial xenial main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

## Ubuntu 18.04 Bionic (Intel)
Ubuntu 18.04 Bionic (i386, amd64):

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/bionic bionic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/bionic bionic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

## Ubuntu 18.10 Cosmic (Intel)
Ubuntu 18.10 Cosmic  (i386, amd64)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/cosmic cosmic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/cosmic cosmic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
```

## Installing Kismet
Once you've enabled the repo and updated your apt-cache, it's time to install Kismet itself.

There are 2 primary builds of Kismet:  The debug build, and the normal build.

The debug build contains all the debugging symbols.  If you are helping test Kismet or debugging a problem, you want the debug symbols, however the debug version will take *significantly* more space.

To install the debug version:
```bash
$ sudo apt install kismet-core-debug kismet-capture-linux-bluetooth kismet-capture-linux-wifi kismet-capture-nrf-mousejack python-kismetcapturertl433 python-kismetexternal python-kismetlog python-kismetrest kismet-logtools 
```

To install the stripped-down version and all related tools:
```bash
$ sudo apt install kismet2018
```

## Installing piecemeal
If you only need the capture tools, you can install individual capture components.  

Follow the same instructions for adding the repository, and then install only the capture drivers you need:

```bash
$ sudo apt install kismet-capture-linux-wifi
```

```bash
$ sudo apt install kismet-capture-linux-bluetooth
```

