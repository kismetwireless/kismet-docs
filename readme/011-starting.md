---
title: "Starting Kismet"
permalink: /docs/readme/starting_kismet/
excerpt: "Configuring and starting Kismet for the first time"
docgroup: "readme"
---

## The Basics

Kismet can be started normally from the command line, and shows a small text-based display of the recent output from the server.  For the modern UI experience, you'll need to connect to the Kismet webserver.

If you're on the same hardware as Kismet (such as a laptop), you will connect to `http://localhost:2501` to reach the Kismet server.

If you're running Kismet on dedicated hardware (such as a Raspberry Pi, Hak5 Wi-Fi Pineapple, or other dedicated device), you'll need to connect to *the address of the device*.  For a Raspberry Pi this is often `http://raspberyrpi.local:2501` by default, but you will need to check the configuration of your device.  It will be the same address you connect to over `ssh` to launch Kismet.

## Launching Kismet

Kismet can be run with no options and configured completely from the web interface:

```bash
$ kismet
```

Or you can point it directly at a network interface and it will configure it during startup.  Different operating systems, kernel versions, and distributions all name network interfaces differently; your Wi-Fi interface may be something like `wlan0`, `wlan1`, or `wlp0s1`.  It may be named based on the MAC address, such as `wlx00aabbccddee`.  On OSX, it is typically named something like `en1`.

If you do not already know the name of your network interface, you can either use the Datasources panel in the WebUI to automatically discover it, or you can use `ifconfig`, `ip`, `iw dev`, and similar commands to find the available network interfaces and identify which one is your wireless interface you plan to use with Kismet.

Once you have identified an interface, you can start Kismet with that source already defined, for example:

```bash
$ kismet -c wlan0
```

*Remember, until you add a data source, Kismet will not be capturing any packets!*

## More than just Wi-Fi

Kismet handles much more than just Wi-Fi, assuming you have the proper capture hardware.  Be sure to check out the data sources category of the Kismet documentation!

## First-time setup

*THE FIRST TIME YOU RUN KISMET*, you *MUST* go to the Kismet WebUI and set your login and password.

This login will be saved in the config file: `~/.kismet/kismet_httpd.conf` which is in the *home directory of the user* starting Kismet when installed in suidroot mode.  This is the preferred way to run Kismet.

If you start Kismet as or via sudo (or via a system startup script where it runs as root), this will be in *roots* home directory: `/root/.kismet/kismet_httpd.conf`

## Automatically launching Kismet

An example systemd script is in the `packaging/systemd/` directory of the Kismet source; if you are installing from source this can be copied to `/etc/systemd/system/kismet.service`, and packages should automatically include this file.

When starting Kismet via systemd, you should install kismet as suidroot, and use `systemctl edit kismet.service` to set the following:

```
[Service]
User=your-unprivileged-user
Group=kismet
```

Also, when using systemd (or any other startup script system), you will need to be sure to configure Kismet to log to a valid location.  By default, Kismet logs to the directory it is launched from, which is unlikely to be valid when starting from a boot script.

Be sure to put a `log_prefix=...` in your [kismet_site.conf](https://www.kismetwireless.net/docs/readme/config_files/#configuration-override-files---kismet_siteconf); for example

```
log_prefix=/home/kismet/logs
```

If you encounter errors launching Kismet from a startup script, be sure to check either `journalctl -xe` or your syslogs for more information.

