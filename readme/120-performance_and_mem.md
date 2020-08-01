---
title: "Performance and Memory Tuning"
permalink: /docs/readme/performance_and_memory/
excerpt: "Tuning options for performance and memory can resolve issues when dealing with very large data sets or very small servers."
docgroup: "readme"
toc: true
---

## Kismet Memory and Processor Tuning
Kismet has several options which control how much memory and processing it uses.  These are found in `kismet_memory.conf`.  Generally it is not necessary to tune these values unless you are running on extremely limited hardware or have a very large number of devices (over 10,000) detected.

* `tracker_device_timeout=seconds`

    Kismet will forget devices which have been idle for more than the specified time, in seconds.

    Kismet will also forget links between devices (such as access points and clients) when the device has been idle for more than the specified time.

    This is primarily useful on long-running fixed Kismet installs.

* `tracker_max_devices=devices`

    Kismet will start forgetting the oldest devices when more than the specified number of devices are seen.

    There is no terribly efficient way to handle this, so typically, leaving this option unset is the right idea.  Memory use can be tuned over time using the `tracker_device_timeout` option.

* `tcp_buffer_kb=kb`

    Kismet allocates a 512Kb buffer for incoming remote datasources, for each datasource.  On very RAM-limited devices, this may be a significant percentage of the available resources; The size of the buffer can be tuned.  Tuning this value too small may cause "buffer full" errors if the buffer cannot be serviced quickly enough.

* `ipc_buffer_kb=kb`

    Kismet allocates a 512Kb buffer for datasource IPC, for each datasource.  On very RAM-limited devices, this may be a significant percentage of the available resources; The size of the buffer can be tuned.  Tuning this value too small may cause "buffer full" errors if the buffer cannot be serviced quickly enough.

* `keep_location_cloud_history=true|false`

    Kismet can track a 'cloud' style history of locations around a device; Similar to a RRD (round robin database), the precision of the records decreases over time.

    The location cloud can be useful for plotting devices on a map, but also takes more memory per device.

* `keep_datasource_signal_history=true|false`

    Kismet can keep a record of the signal levels of each device, as seen by each data source.  This is used for tracking signal levels across many sensors, but uses more memory.

* `track_device_seenby_view=true|false`

    Kismet normally provides a device view API organizing devices by the data source which captured them; this is used by scripts and may be used by the web UI.  On a stand-alone Kismet system which is primarily used for logging, not real-time display, turning this off can save some memory.

* `track_device_phy_view=true|false`

    Kismet normally provides a device view API organizing devices by the PHY type they belong to; this is used by scripts and may be used by the web UI.  On a stand-alone Kismet system which is primarily used for logging, not real-time display, turning this off can save some memory.

* `manuf_lookup=true|false`

    Kismet normally looks up the manufacturer information in the IEEE OUI database.  Indexing this databases uses RAM, as does each unique manufacturer record.  Disabling this option will result in all manufacturer names being `unknown` but will save some RAM.

* `alertbacklog=number`

    The number of alerts Kismet saves for displaying to new clients; setting this too low can prevent clients from seeing alerts but saves memory.

    Alerts will still be logged.

* `packet_dedup_size=packets`

    When using multiple datasources, Kismet keeps a list of the checksums of previous packets; this prevents multiple copies of the same packet triggering alerts.

* `packet_backlog_warning=packets`

    Kismet will start raising warnings when the number of packets waiting to be processed is over this number; no action will be taken, but an alert will be generated.

   This can be set to zero to disable these warnings; Kismet defaults to zero.  Disabling these warnings will NOT disable the backlog limit warnings.

* `packet_backlog_limit=packets`

    This is a *hard limit*.  If the packet processing thread is not able to process packets fast enough, new packets will be dropped over this limit.

    This can be set to 0; Kismet will never drop packets.  This may lead to a runaway memory situation, however.

* `ulimit_mbytes=ram_in_megabytes`

    Kismet can hard-limit the amount of memory it is allowed to use via the 'ulimit' system; this could be set via a launch/setup script using the at startup. 

    If Kismet runs out of RAM , it *will exit immediately* as if the system had encountered an out-of-memory error.

    This setting should ONLY be combined with a restart script that relaunches Kismet, and typically should only be used on long-running WIDS-style installs of Kismet.

    If this value is set too low, Kismet may fail to start the webserver correctly or perform other startup tasks.  This value should typically only be used to control unbounded growth on long-running installs.

    The memory value is specified in *megabytes of RAM*.

    Some older kernels (such as those found on some Debian and Ubuntu versions still in LTS, such as Ubuntu 14.04) do not properly calculate memory used by modern allocation systems and will not count the memory consumed.  On these systems, it may be necessary to use externally-defined `cgroup` controls.

* `dot11_view_accesspoints=true|false`

    Normally, Kismet makes a view dedicated to access points; this can be turned off to save RAM.  Turning this off may break some scripts and will definitely disable the 'Access points only' view in the UI.  This will also disable related features that rely on aggregate Wi-Fi access points, such as timestamp grouping.

* `dot11_view_ssids=true|false`

    Normally, Kismet monitors and sorts the SSIDs seen, and shows them under the SSID tab in the UI.  Turning this off may break some scripts and will definitely disable the 'SSIDs' tab of the UI, but will save memory in very low memory or very high access point environments.

    Turning this off will not affect associating SSIDs with access points or displaying them inside the device details.

* `dot11_fingerprint_devices=true|false`

    Kismet does advanced fingerprinting of 802.11 devices.  This can be turned off to save RAM.  Turning this off will disable some alerts and fingerprint based checks.

* `dot11_keep_ietags=true|false`

    Kismet can keep a copy of the last-advertised IE tags for a SSID and display them; this takes more CPU and RAM and may not be useful to most users, so it is off by default.  This only affects the processing and display of the raw IE tag data.

* `dot11_keep_eapol=true|false`

    By default, Kismet will cache the last 16 EAPOL (WPA handshake) packets for each BSSID.  This enables WIDS detection of EAPOL replay attacks and easy downloading of WPA handshakes as a pcap file per BSSID, but it consumes more RAM and processing power.

    Turning this off will disable WPA handshake downloading and EAPOL replay detection, but EAPOL packets will still be logged and tagged in the kismetdb log.


## Runtime type checking

Kismet uses a dynamic type system to store data (that is then turned into JSON, etc).  Normally this system uses type enforcement; this turns programming errors in Kismet into controlled aborts instead of crashes.

This system can be disabled for extra performance - either on low-power systems such as rpi or embedded hardware, or on extremely high-load systems with many remote datasources.  

Turning off runtime type checking is *generally* safe, because if there was a type mismatch, Kismet would already terminate with a thrown exception / abort, however if there is an as-yet-undiscovered mismatch, Kismet will now segfault with a null pointer access.  This *should not* be a security risk, but a controlled abort is generally preferable.

Runtime type checking must be disabled at compile time:

```bash
$ ./configure --disable-element-typesafety
```

## Extremely large numbers of data sources

Using extremely large numbers of local data sources (in excess of 16 devices) can introduce a new set of instabilities and concerns; depending on the devices used, the kernel version, and if using an out-of-kernel driver such as the RTL8812AU driver set, the driver version.

While *reading* packets from a capture interface is generally very cheap (a bulk transfer operation), configuring an interface or changing the channel may be quite expensive, in terms of work done by the kernel and driver.

Some drivers and kernels seem especially impacted when first setting a very large number of interfaces to monitor mode; this can lead to timeouts or even kernel crashes on some drivers.  Kismet provides a set of tuning knobs in `kismet.conf`:

* `source_stagger_threshold=[number]`

    This determines when Kismet will start staggering local source bring-up - if you have more than this number of sources defined, Kismet will slow down the startup process.

* `source_launch_group=[number]`

    This determines how many sources will be bought up at a time.

* `source_launch_delay=[seconds]`

    The number of seconds between launching each group of sources.

While the default values may be sane for your application, adding this many local sources to Kismet implies an advanced configuration - you may find benefit to tuning these options for your specific configuration.

You may also find it necessary to decrease the channel hopping speed to alleviate contention in the kernel.  This can be especially true with some drivers, such as the rtl8812au/rtl88xx drivers, which have severe contention when setting channels.

When running an extremely large number of sources, remember also that Kismet will likely require a significant amount of CPU and RAM for the additional data being gathered.

