---
title: "Starting Kismet"
permalink: /docs/readme/starting_kismet/
excerpt: "When starting Kismet you can define multiple options on the command line, config files, or perform many operations via the web interface."
docgroup: "readme"
---

## Starting Kismet

Kismet can be started normally from the command line, and will run in a small ncurses-based wrapper which will show the most recent server output, and a redirect to the web-based interface.

Alternately, for appliance-style installs, Kismet can also be started as a service; typically in this usage you should also pass `--no-ncurses` to prevent the ncurses wrapper from loading.

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

## Navigate to the Web UI

Point your browser at http://localhost:2501 (or the address of the server Kismet is running on)

If you are running Kismet on your laptop (or other system with a browser), you can see the Kismet UI at `http://localhost:2501`.

If you are running Kismet on a Raspberry Pi, Wi-Fi Pineapple, or other device, you will need to point your computer at the address *of the device running Kismet*.  You will need to have the system running Kismet plugged into wired Ethernet, or it will need a second Wi-Fi card configured to connect to your network:  You cannot run Kismet and connect to a network on the same Wi-Fi card at the same time.

You will be prompted to do basic configuration - Kismet has many options in the web UI which can be tweaked.  Explore and have fun!

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

