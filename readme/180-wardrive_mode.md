---
title: "Kismet Wardriving Mode"
permalink: /docs/readme/wardriving/
excerpt: "Simplified wardriving mode for smaller platforms"
docgroup: "readme"
toc: true
---

The wardriving mode configuration overlay (and code to support it) was added to Kismet in `2022-01-git` and subsequent releases.

## Wardriving mode

Kismet is already equipped for what most would consider to be wardriving out of the box:  With a GPS and one (or more) Wi-Fi cards, Kismet will generate logs suitable for uploading to Wigle or generating your own maps and logs.

Many people however want to run Kismet on what would typically be considered light-weight or even underpowered hardware, which presents challenges in high-density areas (or even lower density areas, depending how under-powered the hardware is).

Kismet provides a configuration overlay file which preconfigures Kismet to optimize it for basic wardriving - this turns off most logging and data retention, disables processing most packets as much as possible, and configures the radios for optimized AP detection.  All this comes at the cost of normal functionality - in wardrive mode, Kismet won't track non-access-point Wi-Fi devices, perform most IDS functionality, log data packets, or retain handshakes.  What it *will* do is function much better for the specific goal of mapping access points and Bluetooth devices.

## It's just a config overlay

Remember - wardriving mode is *optional*, and it's *just a configuration overlay*.  If the example overlay doesn't suit your needs, just copy it and change the config!

## Using wardriving mode

To use wardriving mode, just launch Kismet with the `--override` option (in addition to any other command line options you pass), for instance:

```bash
$ kismet -t some_wardrive --override wardrive
```

You'll receive an alert that wardriving mode is active and that logging is greatly reduced, and Kismet will automatically be optimized for pure AP collection.

## Digging into the wardriving mode config

What does wardriving mode change?  You can see all the options in the `kismet_wardrive.conf` config file:

* Raise an alert

    Kismet will fire an extra alert to make sure it's obvious wardriving mode is engaged and not all data will be logged.

    ```
    load_alert=WARDRIVING:Kismet is in survey/wardriving mode.  This turns off tracking non-AP devices and most packet logging.
    ```

* Enable wiglecsv logging

    Kismet can log directly to wiglecsv format; this adds it to whatever other log types are already configured.

    ```
    # Turn on wiglecsv format
    log_types+=wiglecsv
    ```

* Turn off HT Wi-Fi channels

    Access point advertisements only happen on the primary Wi-Fi channels; there's no need to tune to HT20, HT40, VHT80, or VHT160 channels.  By eliminating them, we increase the effective channel coverage by hopping through the list faster, meaning we're less likely to miss APs while in motion.

    The options are appended to any 802.11 datasource, local or remote, which doesn't have an explicit option already set.  Specific sources could be left on VHT channels by adding `ht_channels=true,vht_channels=true,default_ht20=true,expand_ht20=true` to those datasource `source=` defintiions.

    ```
    # Turn off HT20, HT40, and VHT options on wifi datasources (unless they explicitly set them)
    dot11_datasource_opt=ht_channels,false
    dot11_datasource_opt=vht_channels,false
    dot11_datasource_opt=default_ht20,false
    dot11_datasource_opt=expand_ht20,false
    ```

* Tune 802.11 tracking

    Kismet normally tracks all device it sees, keeps fingerprints, optionally keeps IE tags for display, and keeps handshakes.  All of this takes memory, so we turn it off.

    ```
    dot11_ap_only_survey=true
    dot11_fingerprint_devices=false
    dot11_keep_ietags=false
    dot11_keep_eapol=false
    ```

* Turn off some other logging

    The kismetdb log can contain channel usage over time, datasource rates over time, and other info - we don't care about that for wardriving, turn it off.  This saves both disk space and IO time to slower disks, like microsd cards.

    ```
    kis_log_channel_history=false
    kis_log_datasources=false
    ```



