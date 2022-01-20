---
title: "Logging"
permalink: /docs/readme/logging_kismetdb/
excerpt: "Kismet unified logging format"
docgroup: "readme"
toc: true
---

## Kismetdb logging

The `kismet` log format is a unified common log of packets, non-packet data, devices, location, system messages, datasource records, historical trends, and almost everything else Kismet can track.  It replaces the legacy logs where multiple log files were needed to reconstruct history.

The `kismet` log file cam be converted to many other log types using the associated log tools, including pcap, pcapng, wiglecsv, json records, KML, and more.

### Log type

```
log_types=kismet
```

### Kismet log journal files

The `kismet` log format uses sqlite3 to create a dynamic random-access file.  If Kismet exits abnormally (such as running out of RAM or the power to the device failing), it may leave behind a `...-journal` file.  

This file is part of the sqlite3 integrity protection, and contains partial data which was not written to the database.

The journal file will be automatically merged when the log file is opened, or you can manually merge them with sqlite command line tools:

```bash
$ sqlite3 Kismet-foo-whatever.kismet 'VACUUM;'
```

or via the included Kismet log cleanup tool:

```bash
$ kismetdb_clean foo.kismet
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

### Duplicate packets

Kismet can record duplicate packets when multiple datasources capture the same data.  In some instances, keeping duplicate packets is desirable (such as remote captures used for device location), while in others, logging duplicates may be a waste of space.

```
kismetdb_log_duplicate_packets=false
```

### Data packets

Kismet typically logs all types of packets.  To discard data packets, retaining only management frames:

```
kismetdb_log_data_packets=false
```

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

