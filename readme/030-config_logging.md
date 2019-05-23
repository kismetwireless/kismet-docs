---
title: "Logging"
permalink: /docs/readme/logging/
excerpt: "Kismet has many logging options; here's how to pick which options you need."
docgroup: "readme"
toc: true
---

## Logging

Kismet supports logging to multiple file types:

* `kismet` is the primary log format now used by Kismet.  This log combines all the data Kismet is able to gather - packets, device records, alerts, system messages, GPS location, non-packet received data, and more.  This file can be manipulated with the tools in the `log_tools/` directory.  Under the covers, a `kismet` log is a sqlite3 database.
* `pcapppi` is the legacy pcap format using the PPI headers.  This format saves Wi-Fi packets, GPS information, and *some* (but not all) of the signal information per packet.  Information about which datasource captured a packet is not preserved.
* `pcapng` is the modern pcap format.  While not all tools support it, Wireshark and TShark have excellent support.  Most tools written using libpcap can read pcap-ng files with a *single* data source.  When using pcap-ng, Kismet can log packets from multiple sources, preserving the datasource information and the original, complete, per-packet signal headers. 

### Picking a log format

Kismet can log to multiple logs simultaneously, configured in the `kismet_logging.conf` config file (or in the `kismet_site.conf` override configuration).  Logs are configured by the `log_types=` config option, and multiple types can be specified:

```
log_types=kismet,pcapng
```

### Log names and locations

Log naming and location is configured in `kismet_logging.conf` (or `kismet_site.conf` for overrides).  Logging can be disabled entirely with:

```
logging_enabled=false
```

or it can be disabled at launch time by launching Kismet with `-n`:

```bash
$ kismet -n ...
```



The default log title is 'Kismet'.  This can be changed using the `log_title=` option:

```
log_title=SomeCustomName
```

or it can be changed at launch time by running Kismet with `-t ...`:

```bash
$ kismet -t SomeCustomeName ...
```



Kismet stores logs in the directory it is launched from.  This can be changed using the `log_prefix=` option; this is most useful when launching Kismet as a service from systemd or similar when the directory it is being launched from may not be where you want to store logs:

```
log_prefix=/tmp/kismet
```

### Kismet log journal files

The `kismet` log format uses sqlite3 to create a dynamic random-access file.  If Kismet exits abnormally (such as running out of RAM or the power to the device failing), it may leave behind a `...-journal` file.  

This file is part of the sqlite3 integrity protection, and contains partial data which was not written to the database.

The journal file will be automatically merged when the log file is opened, or you can manually merge them with sqlite command line tools:

```bash
$ sqlite3 Kismet-foo-whatever.kismet 'VACUUM;'
```

### Kismetdb ephemeral and timed logs

The `kismet` log format can be used as an ephemeral (non-permanent) log file, and can automatically time-limit the data.  This is extremely useful if you are using Kismet as a stand-alone sensor, where the data is being collected over the REST interface.

Kismetdb logs can be automatically purged of old data.  The timeout values are in seconds - `60 * 60 * 24`, which is `86400`, sets a 24 hour timeout on data.

```
kis_log_alert_timeout=86400
kis_log_device_timeout=86400
kis_log_message_timeout=86400
kis_log_packet_timeout=86400
kis_log_snapshot_timeout=86400
```

Data in each of the categories is *removed from the kismetdb log file* after the timeout.  Devices which have been idle for more than the `kis_log_device_timeout` are removed.

A kismetdb log may be marked as *ephemeral* by setting:

```
kis_log_ephemeral_dangerous=true
```

An ephemeral log is removed from the filesystem upon creation; the log is not accessible on disk, and will be removed immediately upon Kismet exiting.  Ephemeral logs are most useful when deploying Kismet as a permanent fixed sensor with a time-limited history; for instance retaining the past day of devices, packets, etc, but where there is no need (or desire) to keep log files.

## Filtering

Kismet can filter packets and devices logged in the `kismetdb` log file.  This can be used to exclude known devices, or include ONLY certain types of devices and packets.

Filtering is most useful when coupled with other tools, and most users won't need to worry about it, but can be a powerful tool in specific situations.

### Pass and block modes
Kismet filters operate as `pass` or `block`.  Events which are `passed` are allowed to be logged, while events which are `blocked` are excluded.

### Default and match modes
Kismet filters have a default mode:  if a record does not match any other filter, the default mode is applied.

Records can match a pass or block mode before reaching the default mode.  A filter with a default of `block` can still allow specific records through with a matching `pass` element.

### Group matches
MAC addresses can be specified as a single address `AA:BB:CC:DD:EE:FF`, or as a masked group `11:22:33:00:00:00/FF:FF:FF:00:00:00`.  

A masked group works like an IP netmask specification:  the value after the `/` indicates the bytes to match on; in the case above, the match will grab any MAC address beginning with `11:22:33:...`. 

The most common use for masked matching is to match OUIs which are the first 3 bytes of the MAC address.

### Device filters
Filtering device records from the `kismetdb` log will not prevent Kismet from showing them, but prevents Kismet from logging them.

Devices are filtered by phyname and mac address.  Mac addresses can be specified as a single MAC or as a masked group.

The default filter controls are in `kismet_filter.conf`.

* `kis_log_device_filter_default=pass | block`
    By default, kismetdb logs are not filtered, and the default for devices is `pass`.

* `kis_log_device_filter=phyname,macaddress,value`
    Adds a device filter for the given phyname and MAC address.  For example:
    `kis_log_device_filter=IEEE802.11,aa:bb:cc:dd:ee:ff,pass`
    `kis_log_device_filter=IEEE802.11,00:11:22:00:00:00/ff:ff:ff:ff:ff:ff,block`

### Packet filters
Filtering packets from the `kismetdb` log will not prevent Kismet from processing them and creating related devices, but will prevent the actual packet data from being logged.

Packets can be filtered based on the MAC address of the source, destination, network, or 'other' address.  For Wi-Fi, the 'other' address is used only in WDS environments where the quad-MAC headers are used.  Packet filters may also filter on 'any' address, which matches any of the four.

The default filter controls are in `kismet_filter.conf`.

* `kis_log_packet_filter_default=pass | block`
   By default, kismetdb logs are not filtered, and the default for packets is `pass`.

* `kis_log_packet_filter=phyname,addresstype,macaddress,value`
   Adds a packet filter for the given phyname, address type (source, destination, network, other, any), and MAC address.  For example:
   `kis_log_packet_filter=IEEE802.11,source,aa:bb:cc:dd:ee,pass`
   `kis_log_packet_filter=IEEE802.11,any,00:11:22:00:00:00/ff:ff:ff:ff:ff:ff,block`

### Live filter control
Packet filters can be enabled and manipulated live via the [filter REST API](/docs/devel/webui_rest/filters/).

