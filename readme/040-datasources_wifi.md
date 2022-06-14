---
title: "Wi-Fi sources"
permalink: /docs/readme/datasources_wifi/
excerpt: "Wi-Fi (802.11) data sources capture packets from an interface in monitor mode."
docgroup: "readme"
toc: true
---

## Wi-Fi

### Wi-Fi Channels

Wi-Fi channels in Kismet define both the basic channel number, and extra channel attributes such as 802.11N 40MHz channels, 802.11AC 80MHz and 160MHz channels, and non-standard half and quarter rate channels at 10MHz and 5MHz.

Kismet will auto-detect the supported channels on most Wi-Fi cards.  Monitoring on HT40, VHT80, and VHT160 requires support from your card.

Channels can be defined by number or by frequency.

| Definition | Interpretation                                               |
| ---------- | ------------------------------------------------------------ |
| xx         | Basic 20MHz channel, such as `6` or `153`                    |
| xxxx       | Basic 20MHz frequency, such as `2412`                        |
| XXHT20     | 20MHz HT20 channel, such as `6HT20`                          |
| XXXXHT20   | 20MHz frequency, such as `2412HT20`                          |
| xxHT40+    | 40MHz 802.11n with upper secondary channel, such as `6HT40+` |
| xxHT40-    | 40MHz 802.11n with lower secondary channel, such as `6HT40-` |
| xxVHT80    | 80MHz 802.11ac channel, such as `116VHT80`                   |
| xxVHT160   | 160MHz 802.11ac channel, such as `36VHT160`                  |
| xxW10      | 10MHz half-channel, a non-standard channel type supported on some Atheros devices.  This cannot be automatically detected, you must manually add it to the channel list for a source. |
| xxW5       | 5MHz quarter-channel, a non-standard channel type supported on some Atheros devices.  This cannot be automatically detected, you must manually add it to the channel list for a source. |

### Datasource: Linux Wi-Fi

Most likely this will be the main data source most people use when capturing with Kismet.

The Linux Wi-Fi data source handles capturing from Wi-Fi interfaces using the two most recent Linux standards:  The new netlink/mac80211 standard present since approximately 2007, and the legacy ioctl-based IW extensions system present since approximately 2002.

Packet capture on Wi-Fi is accomplished via "monitor mode", a special mode where the card is told to report all packets seen, and to report them at the 802.11 link layer instead of emulating an Ethernet device.

The Linux Wi-Fi source will auto-detect supported interfaces by querying the network interface list and checking for wireless configuration APIs.  It can be manually specified with `type=linuxwifi`:
```
source=wlan1:type=linuxwifi
```

The Linux Wi-Fi capture uses the 'kismet_cap_linux_wifi' tool, and should
typically be installed suid-root:  Linux requires root to manipulate the
network interfaces and create new ones.

Example source definitions:
```
source=wlan0
source=wlan1:name=some_meaningful_name
```

### Lockfiles

Linux doesn't gracefully handle probing and creating multiple monitor mode VIFs at once.  To prevent this from happening, Kismet uses a lockfile in `/tmp/.kismet_cap_linux_wifi_interface_lock`;  The Linux capture tool uses this to ensure that only one Kismet process is creating a monitor mode interface at once.

In some rare circumstances this file may be created with privileges that are not accessible from Kismet while running as suid-root; in these instances you will see an error opening an interface that it could not acquire the lock file.  Fixing these incidents should be as simple as a one-time removal of the file:

```bash
$ sudo rm /tmp/.kismet_cap_linux_wifi_interface_lock
```

### Supported Hardware

Not all hardware and drivers support monitor mode, but the majority do.  Typically any driver shipped with the Linux kernel supports monitor mode, and does so in a standard way Kismet understands.  If a specific piece of hardware does not have a Linux driver yet, or does not have a standard driver with monitor mode support, Kismet will not be able to use it.

The Linux Wi-Fi source is known to support, among others:
* All Atheros-based cards (ath5k, ath9k, ath10k with some restrictions,  USB-based atheros cards like the AR9271) (* Some issues)
* Modern Intel-based cards (all supported by the iwlwifi driver including the 3945, 4965, 7265, 8265 and similar) (* Some issues)
* Realtek USB devices (rtl8180 and rtl8187, such as the Alfa AWUS036H)
* RALink rt2x00 based devices
* ZyDAS cards
* Broadcom cards such as those found in the Raspberry Pi 3 and Raspberry Pi 0W, *if you are using the nexmon drivers*.  It is not posisble to use Kismet with the *default drivers* from Raspbian or similar distributions.
   The Kali distribution for the Raspberry Pi *includes the nexmon patches already* and will work.
   To patch your own distribution with nexmon, consult the nexmon site at: https://github.com/seemoo-lab/nexmon
* Mediatek mt7612u 802.11AC devices; these are some of the most supported devices.
* Almost all drivers shipped with the Linux kernel

Devices known to have issues:
* ath9k Atheros 802.11abgn cards are typically the most reliable, however they appear to return false packets with valid checksums on very small packets such as phy/control and powersave control packets.  This may lead Kismet to detect spurious devices not actually present.
* ath10k Atheros 802.11AC cards have many problems, including floods of spurious packets in monitor mode.  These packets carry 'valid' checksum flags, making it impossible to programmatically filter them.  Expect large numbers of false devices.  It appears this will require a fix to the closed-source Atheros firmware to resolve.
* iwlwifi Intel cards, with older kernel drivers and firmware, have significant crashing issues when tuning to HT40 and HT80 channels.  Modern kernels appear to have resolved this issue; if you cannot upgrade your kernel, you can disable HT and VHT channels by passing the source options `ht_channels=false` and `vht_channels=false`; such as `source=wlp4s0:name=someintel,ht_channels=false,vht_channels=false`
* rtl8812 and 8814 USB 802.11AC cards are known to have many strange problems.  While extremely common hardware, these cards use out-of-kernel drivers which do not support standard monitor mode vif configuration.  There are many flavors of these drivers, many of which cannot enter monitor mode, or silently fail to enable monitor mode.  Despite being a common and cheap chipset, these cards are best avoided because they will take a lot of work to get running.
* rtl88x2bu based cards have an out-of-kernel driver which doesn't support mac80211 VIFs or modern channel control.  Kismet will fall back to the old WEXT ioctl control method, but these drivers will not support setting HT channels.

Kismet generally *will not work* with most other out-of-kernel (drivers not shipped with Linux itself), specifically drivers such as the SerialMonkey RTL drivers used for many of the cheap, tiny cards shipped with devices like the Raspberry Pi and included in distributions like Raspbian.  Some times it's possible to find other, supported drivers for the same hardware, however some cards have no working solution.

Many more devices should be supported - if yours isn't listed and works, let us know via Twitter (@kismetwireless).

#### Linux Wi-Fi Source Parameters
Linux Wi-Fi sources accept several options in the source definition, in addition to the common name, informational, and UUID elements:

* `add_channels="channel1,channel2,channel3"`

   A comma-separated list of channels *that will be appended* to the detected list of channels on a data source.  Kismet will autodetect supported channels, then include channels in this list.
   The list of channels *must be enclosed in quotes*, as in:
   ```
   source=wlan0:add_channels="1W5,6W5,36W5",name=foo
   ```
   If you are configuring the list of Kismet sources from the command line, you will need to escape the quotes or the shell will try to interpret them incorrectly:
   ```bash
   $ kismet -c wlan0:add_channels=\"1W5,6W5\",name=foo
   ```
   This option is most useful for including special channels which are not auto-detected, such as the 5MHz and 10MHz custom Atheros channels.

* `channel=channel definition`

   When channel hopping is disabled, set the channel the card monitors.
   ```
   source=wlan0:name=foo,channel=6
   ```

* `channels="channel,channel,channel"`

   Override the autodetected channel list and provide a fixed list.  Unlike `add_channels` this *replaces* the list of channels Kismet would normally use.
   This must be quoted, as in:
   ```
   source=wlan0:name=foo,channels="1,6,36,11HT40-"
   ```
   If you are defining the Kismet sources from the command line, you will need to escape the quotes or the shell will try to interpret them incorrectly:
   ```bash
   $ kismet -c wlan0:name="foo",channels=\"1,6,36,11HT40-\"
   ```

* `channel_hop=true | false`

   Enable or disable channel hopping on this source only.  If this option is omitted, Kismet will use the default global channel hopping configuration.

* `channel_hoprate=channels/sec | channels/min`

   Control the per-source channel hop rate.  If this option is omitted, Kismet will use the default global channel hop rate.

* `default_ht20=true | false`

   Added `2019-04-git`

   If the interface is HT capable, automatically use HT20 channels for all 20mhz wide channels.  This explicitly tells the interface to set the HT20 attributes instead of a basic channel.  If `default_ht20=true`, then `expand_ht20` is ignored.  This will likely become the default value in the future after testing.

* `expand_ht20=true | false`

   Added `2019-04-git`

   If the interface is HT capable, automatically expand 20MHz channels to define the basic *and* the HT20 channel; for example instead of channel `1`, you would now have both `1` and `1HT20`.
   This has the possibility to drastically increase the number of channels in the hop list, which increases the hop time.
   This option is most useful on interfaces which may not report non-HT data packets when tuned to HT20.

* `fcsfail=true | false`

   Wi-Fi packets contain a `frame checksum` or `FCS`.  Some drivers report this as the FCS bytes, while others report it as a flag in the capture headers which indicates if the packet was received correctly.
   Generally packets which fail the FCS checksum are garbage - they are packets which are corrupted, usually due to in-air collisions with other packets.  These can be extremely common in busy wireless environments.
   Usually there is no reason to set this option unless you are doing specific research on non-standard packets and hope to glean some information from corrupted packets.

* `filter_locals=true | false`

    Automatically detect all local interfaces and build a BPF filter to exclude them from the capture.  This is most useful for remote capture instances which are connected over wireless.  The filter can only exclude the first 8 devices found, because of limits in the kernel memory buffer for BPF filtering.

* `filter_mgmt=true | false`

    Use a kernel-level BPF filter to filter out all *except* 802.11 management and EAPOL (WPA handshake) packets.

    Enabling this filter will *drastically* reduce the amount of processing power required for Kismet, however it will exclude *all other data packets*.  Wireless APs and clients will be visible, however wired/bridged clients will not, and data-based statistics like bandwidth will not be available, however beacon-based statistics like QBSS reports will be retained.

    This feature is used by the wardrive-mode overlay.

* `ht_channels=true | false`

   Kismet will detect and tune to HT40 channels when available; to disable this, set `ht_channels=false` on your source definition.
   Kismet will automatically disable HT channels on some devices which are known to have problems tuning to HT channels; if your device has trouble tuning to HT channels, or you simply don't want to tune to HT channels when the capability is seen, specify `ht_channels=false`.
   See the `vht_channels` option for similar control over 80MHz and 160MHz VHT channels.

* `ignoreprimary=true | false`

   Linux mac80211 drivers use `virtual interfaces` or `VIFs` to set different interface modes and behaviors:  A single Wi-Fi card might have `wlan0` as the "normal" (or "managed") Wi-Fi interface; Kismet would then create `wlan0mon` as the monitor-mode capture interface.
   Typically, all non-monitor interfaces must be disabled (set to `down` state) for capture to work reliably and for channel setting (and channel hopping) to function.
   In the rare case where you are attempting to run Kismet on the same interface as an access point or client, you will want to leave the base interface configured and running (while losing the ability to channel hop); by setting `ignoreprimary=true` on your Kismet source line, Kismet will no longer bring down any related interface on the same Wi-Fi card.
   This **almost always** must be combined with also setting `channel_hop=false` because channel control is not possible in this configuration, and depending on the Wi-Fi card type, may prevent proper data capture.

* `plcpfail=true | false`

   Some drivers have the ability to report data that *looked* like a packet, but which have invalid radio-level packet headers (the Wi-Fi `PLCP` which is not typically exposed to the capture layer).  Generally these events have no meaning, and few drivers are able to report them.
   Usually there is no good reason to turn this on, unless you are doing research attempting to capture Wi-Fi-like data.

* `vif=foo`

   Many drivers use `virtual interfaces` or `VIFs` to control behavior.  Kismet will make a monitor mode virtual interface (vif) automatically, named after some simple rules:
   * If the interface given to Kismet on the source definition is already in monitor mode, Kismet will use that interface and not create a VIF
   * If the interface name is too long, such as when some distributions use the entire MAC address as the interface name, Kismet will make a new interface named `kismonX`
   * Otherwise, Kismet will add `mon` to the interface; ie given an interface `wlan0`, Kismet will create `wlan0mon`
   
   The `vif=` option allows setting a custom name which will be used instead of generating the monitor interface name.
   
* `vht_channels=true | false`

   Kismet will tune to VHT80 and VHT160 channels when available; `vht_channels=false` will exclude them from this list.
   Kismet will automatically exclude VHT channels from devices known to have problems tuning to them; if your device has trouble tuning to VHT channels, or you simply don't want to tune to VHT channels when the capability is seen, specify `vht_channels=false`.
   See the ht_channels option for similar control over HT40 channels.
   
* `retry=true | false`

   Automatically try to re-open this interface if an error occurs.  If the capture source encounters a fatal error, Kismet will try to re-open it in five seconds.  If this is omitted, the source will use the global retry option.

* `verbose=true | false`

   Added `2019-07-git`

   Turn on verbose error reporting and warnings; this will raise alerts when channel operations take an extended period of time or if a channel fails to set correctly.

#### Custom / non-standard channels

Some sources - the Atheros 9k and possibly 10k series - support half-width (10MHz) and quarter-width (5MHz) channels.  These channels must be specified manually with `channels=...` or `add_channels=...`.

Often these devices seem to have difficulty switching between normal and custom channel modes; you may need to set a card to use *only* 5MHz or 10MHz channels instead of mixing with normal mode.

### Data source: OSX Wifi
Kismet can use the built-in Wi-Fi on a Mac, but ONLY the built-in Wi-Fi; Unfortunately currently there appear to be no drivers for OSX for USB devices which support monitor mode.

Kismet uses the `kismet_cap_osx_corewlan_wifi` tool for capturing on OSX.

#### OSX Wi-fi Parameters

OSX Wi-Fi sources support the standard options supported by all sources (such as name, uuid, and informational elements) as well as:
* `channels="channel,channel,channel"`

   Override the autodetected channel list and provide a fixed list.  Unlike `add_channels` this *replaces* the list of channels Kismet would normally use.
   This must be quoted, as in:
   ```
   source=wlan0:name=foo,channels="1,6,36,11HT40-"
   ```
   If you are defining the Kismet sources from the command line, you will need to escape the quotes or the shell will try to interpret them incorrectly:
   ```bash
   $ kismet -c wlan0:name="foo",channels=\"1,6,36,11HT40-\"
   ```

* `channel_hop=true | false`

   Enable or disable channel hopping on this source only.  If this option is omitted, Kismet will use the default global channel hopping configuration.

* `channel_hoprate=channels/sec | channels/min`
   Control the per-source channel hop rate.  If this option is omitted, Kismet will use the default global channel hop rate.

### Tuning Wi-Fi Packet Capture
Kismet has a number of tuning options to handle quirks in different types of packet captures.  These options can be set in the kismet.conf config file to control how Kismet behaves in some situations:

* `dot11_process_phy=[true|false]`

   802.11 Wi-Fi networks have three basic packet classes - Management, Phy, and Data.  The Phy packet type is the shortest, and contains the least amount of information - it is used to acknowledge packet reception and controls the packet collision detection CTS/RTS system.  These packets can be useful, however they are also the most likely to become corrupted and still pass checksum.
   Kismet turns off processing of Phy packets by default because they can lead to spurious device detection, especially in high-data captures.  For complete tracking and possible detection of hidden-node devices, it can be set to 'true'.

