---
title: "Data sources"
permalink: /docs/readme/datasources/
excerpt: "Data sources are how Kismet gets packets (and packet-like) data; many can be automatically configured but some need special options."
docgroup: "readme"
toc: true
---

## Kismet Data Sources

Kismet gets data (which can be packets, devices, or other information) from "data sources".

### Configuring Datasources

Datasources can be defined in multiple ways:

* The Kismet config file (preferably `kismet_site.conf`):

    ```
    source=foo:name=foo,opt2=bar,...
    ```

* The Kismet command line when Kismet is started:

    ```bash
    $ kismet -t somelogname -c foo:name=foo,opt2=bar,...
    ```

* The web interface

    Sources can be opened in the Kismet web UI in the `Data sources` window.

* Scriptable via the REST API

    Sources can be opened via the REST API, via shell scripts and `curl`, Python scripts, etc.

### Datasource options

Datasource configurations are all formatted the same, with the capture interface first, and any number of options for that capture following.

For example to capture from a Linux Wi-Fi device on `wlan1` with no special options:

   ```
   source=wlan1
   ```

To capture from a Linux Wi-Fi device on wlan1 while setting some special options, like telling it to not change channels and to capture from channel 6:

   ```
   source=wlan1:channel_hop=false,channel=6
   source=wlan1:channel_hop=false,channel=11HT-
   ```

All datasources support basic options for naming, basic channel control, and so on.  Each datasource type also supports custom options depending on what sort of controls that type of interface needs.

### Default options

When no options are provided for a data source, the defaults are controlled by settings in kismet.conf; these defaults are applied to all new datasources:

* `channel_hop=true | false`

    Universally enable or disable channel hopping.

    A radio can typically only tune to a single channel at a time.  To capture from multiple channels, Kismet need to rapidly change channel.

    Typically, channel hopping should be left on.  It can be disabled per-source as a source option to zero in on a specific channel.

* `channel_hop_speed=channels/sec | channels/min`

    Control how quickly Kismet hops channels.

    Finding the right balance of channel hopping speed can depend on your environment, hardware, and capture goals.

    The faster Kismet hops channels, the more likely it is to spot a device, but the less likely it is to capture useful data because it will leave the channel equally quickly.  Conversely, a slow hopping speed may show a more accurate representation of data use, but can miss devices which briefly transmit.

    The maximum channel hop rate is also impacted by both the protocol itself - the minimum amount of time for a complete packet to be transmitted - and the hardware and drivers ability to set channels.

    Adding additional datasources is often the best way to increase coverage, as it allows multiple channels to be observed at the same time.

    By default, Kismet hops 5 times a second, which is a reasonable balance for most datasource types.  Individual datasources can also have independent hop rates.

    Examples:

    ```
    channel_hop_speed=5/sec
    channel_hop_speed=10/min
    ```

* `split_source_hopping=true | false`

    Divide channels between multiple datasources with the same coverage.

    Kismet can capture from multiple datasources at once; for example two, three, or more Wi-Fi cards.  To increase coverage, Kismet will split the channel list between datasources which have the same channel support and hopping rate.

    Generally there is no reason to disable this option.

* `randomized_hopping=true | false`

    Offset the channel hop pattern to maximize coverage when channels overlap.

    On some common datasources, like Wi-Fi, channels can overlap (2.4GHz channels overlap by a significant amount).  By offsetting the channel hop sequence by the overlap, Kismet can use the overlap to increase coverage of adjacent channels.

    Generally, there is no reason to turn this off.

* `retry_on_source_error=true | false`

    Kismet will try to re-open a source which is in an error state after five seconds.  This helps Kismet re-open sources which are disconnected or have a driver error.

    There is generally no reason to turn this off.

* `timestamp=true | false`

    Typically, Kismet will override the timestamp of the packet with the local timestamp of the server; this is the default behavior for remote data sources, but it can be turned off either on a per-source basis or in `kismet.conf` globally.

    Generally the defaults have the proper behavior, especially for remote data sources which may not be NTP time synced with the Kismet server.

### Naming and describing data sources

Datasources allow for annotations; these have no role in how Kismet operates, but the information is stored alongside the source definition and is available in the Kismet logs and in the web interface.

The following information can be set in a source, but is not required:

* `name=arbitrary name`

    Give the source a human-readable name.  This name shows up in the web UI and the Kismet log files.  This can be extremely useful when running remote capture where multiple sensors might all have `wlan0`, or simply to give interfaces a more descriptive name.

    ```
    source=wlan0:name=foobar_some_sensor
    ```

* `info_antenna_type=arbitrary antenna type`

    Give the source a human-readable antenna type.  This type shows up in the logs.

    ```
    source=wlan0:name=foobar,info_antenna_type=omni
    ```

* `info_antenna_gain=value in dB`

    Antenna gain in dB.  This gain is saved in the Kismet logs that describe the datasources.

    ```
    source=wlan0:name=foobar,info_antenna_type=omni,info_antenna_gain=5.5
    ```

* `info_antenna_orientation=degrees`

    Antenna orientation, in degrees.  This is useful for a fixed antenna deployment where different sources have physical coverage areas.

    ```
    source=wlan0:name=foobar,info_antenna_orientation=180
    ```

* `info_antenna_beamwidth=width in degrees`

    Antenna beamwidth in degrees.  This is useful for annotating sources with fixed antennas with specific beamwidths, like sector antennas.

    ```
    source=wlan0:info_antenna_type=sector,info_antenna_beamwidth=30
    ```

* `info_amp_type=amplifier type`

    Arbitrary human-readable type of amplifier, if one is present:
    ```
    source=wlan0:info_amp_type=custom_duplex
    ```

* `info_amp_gain=gain in dB`

    Amplifier gain, if any:
    ```
    source=wlan0:info_amp_type=custom_duplex,info_amp_gain=20
    ```

#### Setting source IDs

Typically Kismet generates a UUID based on attributes of the source - the interface MAC address if the datasource is linked to a physical interface, the device's position in the USB bus, or some other consistent identifier.

To override the UUID generation, the `uuid=...` parameter can be set:

```
source=wlan0:name=foo,uuid=AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE
```

If you are assigning custom UUIDs, you **must ensure** that every UUID is **unique**.  Each data source **must have** its own unique identifier.

Most datasource types do not need a custom UUID set, the most notable exception being some SDR-based **remote** sources, where multiple SDR devices can have the same serial number and same position on the USB bus on different capture devices.

#### Attaching a "meta" GPS

*Added in Kismet-2022-01-R3*

Sometimes, you might want to connect a specific source to a different GPS, for instance when using [remote capture](/docs/readme/datasources_remote_capture/).  A "meta-gps" device reports location to Kismet and is attached to a specific datasource, but obtains location data from the REST API.

To set a meta GPS, use the `metagps` parameter:

```
source=wlan0:name=foo,metagps=foobar
```

More likely, you will use this as part of [remote capture](/docs/readme/datasources_remote_capture/):

```bash
$ kismet_cap_linux_wifi --connect *host* --source wlan0:name=remote1,metagps=remote1
```

You will then need to use an additional tool to populate the GPS, using the [metagps api](/docs/devel/webui_rest/gps/#meta-gps).

Multiple datasources can use the same meta GPS, or have independent meta GPS devices (or use the system-wide GPS if no metagps is specified).

### Multiple Kismet Datasources

Kismet will attempt to open all the sources defined on the command line (with the `-c` option), or if no sources are defined on the command line, all the sources defined in the Kismet config files.

If a source has no functional type and encounters an error on startup, it will be ignored - for instance if a source is defined as:

```
source=wlx4494fcf30eb3
```

and that device is not connected when Kismet is started, it will raise an error but will be ignored.

To force Kismet to try to open a device which could not be found at startup, you will need to provide the source type; for instance, the same source defined with the type field:

```
source=wlx4494fcf30eb3:type=linuxwifi
```

will continually try to re-open the device.

