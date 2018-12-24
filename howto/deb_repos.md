---
title: "Official Kismet Deb repos"
permalink: /docs/howto/repos_deb/
toc: true
---
There are automatically built repositories for Kismet on common Deb-based distributions (Ubuntu and Kali, currently).  Here's the quickest way to get started using them.

## Remove any installed Kismet
You will need to remove any installed Kismet code.

If you installed *from a Kismet package in your distribution*, remove it:

```bash
$ sudo apt-get remove kismet kismet-plugins
```

If you installed *Kismet from source* yourself, either the old Kismet or the new Kismet git or beta code, you will need to remove it.  Asssuming you installed it in the default location:

```bash
$ sudo rm -rfv /usr/local/bin/kismet* /usr/local/share/kismet* /usr/local/etc/kismet*
```

## Kali Linux (Intel, Raspberry Pi)
To enable the Kismet nightly build repository on Kali Linux (on i386, amd64, Raspberry Pi 3, Raspberry pi 64bit, and Raspberry Pi 0w):

```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/kali rolling main' | sudo tee /etc/apt/sources.list.d/kismet.list

$ sudo apt-get update
```

## Ubuntu 16.04 Xenial (Intel)
To enable to Kismet nightly build repository on Ubuntu 16.04 Xenial (i386, amd64):

```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/ubuntu xenial main' | sudo tee /etc/apt/sources.list.d/kismet.list

$ sudo apt-get update
```

## Ubuntu 18.04 Bionic (Intel)
To enable to Kismet nightly build repository on Ubuntu 18.04 Bionic (i386, amd64):

```bash
$ wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key | sudo apt-key add -
$ echo 'deb https://www.kismetwireless.net/repos/apt/git/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/kismet.list

$ sudo apt-get update
```

## Installing Kismet
Once you've enabled the repo and upated your apt-cache, it's time to install Kismet itself.

There are 2 primary builds of Kismet:  The debug build, an the normal build.

The debug build contains all the debugging symbols.  If you are helping test Kismet or debugging a problem, you want the debug symbols, however the debug version will take *significantly* more space.

To install the debug version:
```bash
$ sudo apt install kismet-core-debug kismet-capture-linux-bluetooth kismet-capture-linux-wifi kismet-capture-nrf-mousejack python-kismetcapturertl433 python-kismetexternal python-kismetlog python-kismetrest kismet-logtools 
```

To install the stripped down version and all related tools:
```bash
$ sudo apt install kismet2018
```

## Installing piecemeal
For small platforms you only need the remote capture tools, you can install individual capture components.  Follow the same instructions for adding the repository, and then install only the capture drivers you need:

```bash
$ sudo apt install kismet-capture-linux-wifi
```

