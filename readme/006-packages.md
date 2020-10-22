---
title: "Official Kismet packages"
permalink: /docs/readme/packages/
excerpt: "Most distributions will not have the latest Kismet versions, but you can install the official Kismet packages for many common distros and platforms."
docgroup: "readme"
toc: true
redirect_from:
  - /docs/howto/repos_deb/
---

These repositories contain the latest Kismet versions, which may not be available in the standard repositories for your distribution; distributions typically pick up new releases at relatively long intervals, and will not include git or beta versions in the official packages.

There are automatically-built repositories for Kismet on several Linux distributions.  More are being added over time, and your distribution may already have modern packages (Pentoo, for instance).

## Remove any Kismet installed from source
Before you switch to using packages, you will need to remove any Kismet versions installed from source.

```bash
$ sudo rm -rfv /usr/local/bin/kismet* /usr/local/share/kismet* /usr/local/etc/kismet*
```

Once you have switched to using the Kismet packages here, you should be able to upgrade with the standard distribution tools.

## Release or git
If you'd like to be on the cutting edge of testing, you can pull Kismet from nightly git builds.  These builds take the latest git version and compile it - this version has all the absolutely latest features, but also is the most likely to have new, exciting bugs.  The git version is *generally* fine to use, but is not recommended for installations that need consistency or long-term support.

The release version is build from the latest release tag, *or* the latest beta tag.  These versions are generally tagged to allow a consistent installation version from code that *should* be known-good.

## Configuration and locations
The Kismet packages install Kismet and the capture tools into `/usr/bin/`, and the configuration files into `/etc/kismet/`.

If you're used to compiling from source, these are new directories, which match the standard locations for system packages.

## Kali Linux (Intel, Raspberry Pi)
Kali Linux (on i386, amd64, armhf - Raspberry Pi 3, Raspberry Pi 4, arm64 - Raspberry Pi 3 64bit, and armel - Raspberry Pi 0w)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/kali kali main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/kali kali main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Debian / Raspbian Stretch (Intel, Raspberry Pi)
Debian Stretch (i386, amd64, armhf - Raspberry Pi 3, Raspberry Pi 0w)

*WARNING* - You will *not* be able to capture from the built-in Wi-Fi on the Raspberry Pi 3 or Pi 0w unless you also install the [nexmon driver patch](https://github.com/seemoo-lab/nexmon/).  This patch adds reverse-engineered monitor mode to the Broadcom driver.  You can still use USB devices, though!

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/stretch stretch main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/stretch stretch main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Debian / Raspbian Buster (Intel, Raspberry Pi)
Debian Buster (amd64, armhf - Raspberry Pi 3, Raspberry Pi 4)

*WARNING* - You will *not* be able to capture from the built-in Wi-Fi on the Raspberry Pi 3 or Pi 4 unless you also install the [nexmon driver patch](https://github.com/seemoo-lab/nexmon/).  This patch adds reverse-engineered monitor mode to the Broadcom driver.  You can still use USB devices, though!

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/buster buster main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/buster buster main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Ubuntu 16.04 Xenial (Intel)
Ubuntu 16.04 Xenial (i386, amd64)

***Note***: The Xenial packages do not include support for all features, due to limitations in the packages available for Xenial.  You may be able to manually install newer versions of the required libraries and compile from source.

The following are not available in the Xenial packages:
* Python based datasources (rtl433, ADSB, AMR, Freaklabs)
* Ubertooth datasource
* Remote capture via websockets (server support is present, but datasources will not be able to export over remote capture)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/xenial xenial main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/xenial xenial main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Ubuntu 18.04 Bionic (Intel)
Ubuntu 18.04 Bionic (i386, amd64):

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/bionic bionic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/bionic bionic main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Ubuntu 20.04 Focal (Intel)
Ubuntu 20.04 Focal  (amd64)

### Release (beta and release versions)
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/release/focal focal main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

### Nightly git
```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/focal focal main' | sudo tee /etc/apt/sources.list.d/kismet.list
$ sudo apt update
$ sudo apt install kismet
```

## Installing Kismet
There are 2 primary builds of Kismet:  The debug build, and the normal build.

The debug build contains all the debugging symbols.  If you are helping test Kismet or debugging a problem, you want the debug symbols, however the debug version will take *significantly* more space.  By default, the `kismet` metapackage installs the stripped version.

To install the standard version and all related tools, the simplest method is by using the metapackage:
```bash
$ sudo apt install kismet
```

Individual tools can still be installed:
```bash
$ sudo apt install kismet-core kismet-capture-linux-bluetooth kismet-capture-linux-wifi kismet-capture-nrf-mousejack python-kismetcapturertl433 python-kismetcapturertladsb python-kismetcapturertlamr python-kismetcapturefreaklabszigbee kismet-logtools 
```

To install the debug version:
```bash
$ sudo apt install kismet-core-debug kismet-capture-linux-bluetooth kismet-capture-linux-wifi kismet-capture-nrf-mousejack python-kismetcapturertl433 python-kismetcapturertladsb python-kismetcaptureamr python-kismetcapturefreaklabszigbee kismet-logtools 
```

## Installing piecemeal
Most of the Kismet components will work independently - with the caveat of course that you will not be able to capture from a device if you don't have the required capture tool.

To install only the capture tools, for instance to build a remote-capture node, you can install just the individual components:

Follow the same instructions for adding the repository, and then install only the capture drivers you need:

```bash
$ sudo apt install kismet-capture-linux-wifi
```

or,
```bash
$ sudo apt install kismet-capture-linux-bluetooth
```

