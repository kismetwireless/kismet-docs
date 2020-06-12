---
title: "Phy802.11 Scanning-mode Sources"
permalink: /docs/devel/webui_rest/phy80211_scansource/
toc: true
docgroup: "devel-rest"
excerpt: "A simple API for non-packet-capture 802.11 devices to report scanning results to Kismet"
---

## Scanning mode

Properly capturing packets in Wi-Fi requires monitor mode and either a local Wi-Fi device or a high bandwidth connection.  Scanning mode allows devices without special drivers to report networks to Kismet, but with some severe limitations:

* Clients will not be visible.  
* The scanning mode device may transmit probe requests while scanning.
* Enhanced information from the beacon such as max speed, etc, will often not be available.
* Actual packet data will not be available.

Scanning mode is really only appropriate for specific configurations, such as:

* Mobile devices like Android or IOS reporting scan results to a central Kismet server
* Embedded devices such as the ESP8266 or ESP32 

## Scanning mode datasources

Scanning mode datasources are created dynamically by Kismet when reports are submitted; there is no need to define a specific scanning mode datasource prior to sending a report.

A scanning mode report must include:

1. A datasource UUID.  This ID must be unique within Kismet, and consistent between all reports from this scanning source.  Scanning software should cache this UUID for consistency.

2.  A human-readable name.  This will be assigned as the name of the datasource, and will be updated if it changes.  Scanning software should cache this name for consistency.

## Cache/burst mode reporting

Scanning mode assumes that the device doing scanning is not able to maintain a constant connection to the Kismet server.

Reports can be cached and send in groups using the report endpoint; each report can contain a timestamp, GPS location, and signal information, and multiple reports over time can be sent for a single AP.

## Scanning mode report

A scanning mode report consists of a [command dictionary](/docs/devel/webui_rest/commands/) holding an array of reports.  Virtual datasources for each new report are automatically created.

* API added \
    `2020-06`

* URL \
    /phy/phy80211/scan/scan_report.cmd

* Methods \
    `POST` 

* POST parameters \
    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key | Description |
    | --- | ----------- |
    | reports | Array containing multiple report objects |
    | source_name | A unique, consistent source name for the virtual datasource reporting this scan. |
    | source_uuid | A unique, consistent source UUID for the virtual datasource reporting this scan. |

    A report object should contain:

    | Key | Description |
    | --- | ----------- |
    | timestamp | (Optional) Unix timestamp at second precision.  If no timestamp is provided, the time of this message is used.  Due to general lack of precision of scanning mode, timestamp is second only. |
    | ssid | (Optional) SSID |
    | bssid | BSSID |
    | capabilities | (Optional) An Android or Wigle style string of encryption options, such as `[WPS]`, `[WPA-PSK-TKIP+CCMP]`, `[WEP]`, and so on. |
    | channel | (Optional) Quoted string channel, such as `"6"`, `"42HT40P"` |
    | freqkhz | (Optional) Frequency of AP, in KHz |
    | signal | Signal, in dBm |
    | lat | (Optional) GPS latitude |
    | lon | (Optional) GPS longitude |
    | alt | (Optional) GPS altitude |
    | speed | (Optional) GPS speed |

* Results \
    `HTTP 200` on success
    HTTP error on failure

