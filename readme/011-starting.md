---
title: "Starting Kismet"
permalink: /docs/readme/starting_kismet/
excerpt: "When starting Kismet you can define multiple options on the command line, config files, or perform many operations via the web interface."
docgroup: "readme"
---

## Starting Kismet

Kismet can be started normally from the command line, and will run in a small ncurses-based wrapper which will show the most recent server output, and a redirect to the web-based interface.

Kismet can also be started as a service; typically in this usage you should also pass `--no-ncurses` to prevent the ncurses wrapper from loading.

An example systemd script is in the `packaging/systemd/` directory of the Kismet source; if you are installing from source this can be copied to `/etc/systemd/system/kismet.service`, and packages should automatically include this file.

When starting Kismet via systemd, you should install kismet as suidroot, and use `systemctl edit kismet.service` to set the following:

```
[Service]
User=your-unprivileged-user
Group=kismet
```

Also, when using systemd (or any other startup script system), you will need to be sure to configure Kismet to log to a valid location.  By default, Kismet logs to the directory it is launched from, which is unlikely to be valid when starting from a boot script.

Be sure to put a `log_prefix=...` in your [`kismet_site.conf`](https://www.kismetwireless.net/docs/readme/config_files/#configuration-override-files---kismet_siteconf); for example

```
log_prefix=/home/kismet/logs
```

