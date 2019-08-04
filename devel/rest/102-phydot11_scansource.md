---
title: "Phy802.11 Scanning-mode Sources"
permalink: /docs/devel/webui_rest/phy80211_scansource/
toc: true
docgroup: "devel-rest"
excerpt: "A simple API for non-packet-capture 802.11 devices to report scanning results to Kismet"
---

## Scanning mode

Properly capturing packets in Wi-Fi requires monitor mode and either a local Wi-Fi device or a high bandwidth connection.  Scanning mode allows devices without special drivers to report networks to Kismet, but with some severe limitations:

* Clients will not be visible.  It is not possible to derive the number of clients on a network from scanning mode.
* The scanning mode device may transmit probe requests while scanning.
* Enhanced information from the beacon such as max speed, etc, will likely not be available.
* Actual packet data will not be available.

Scanning mode is really only appropriate for specific configurations, such as:

* Mobile devices like Android or IOS reporting scan results to a central Kismet server
* Embedded devices such as the ESP8266 or ESP32 

## Scanning mode datasources

A scanning report can include two different ways to identify the datasource:

1. A unique and consistent name.  If no other datasource with that name exists, a new virtual datasource will be created and all future reports with the same name will be attached to it.  This can be used for data from devices with a unique ID or serial number, but no code for generating a UUID.
2. A datasource UUID.  A virtual datasource with a matching UUID will be created.  

For each supplied name and/or UUID, Kismet will create a virtual datasource.  This datasource is used to track seen-by and reports.

## Scanning mode report

A scanning mode report consists of a [command dictionary](/docs/devel/webui_rest/commands/) holding an array of reports.  Virtual datasources for each new report are automatically created.

* API added \
    `2019-08`

* URL \
    /phy/phy80211/scan/scan_report.cmd

* Methods \
    `POST` 

* POST parameters \
    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key | Description |
    | --- | ----------- |
    | reports | Array containing multiple report objects |
    | source_name | (Optional) A unique, consistent source name for the virtual datasource reporting this scan.  A `source_uuid` may also be specified.  A `source_name` or `source_uuid` MUST be specified. |
    | source_uuid | (Optional) A unique, consistent source UUID for the virtual datasource reporting this scan.  A `source_name` may also be specified.  A `source_name` or `source_uuid` MUST be specified. |

    A report object should contain:

    | Key | Description |
    | --- | ----------- |
    | timestamp | (Optional) Unix timestamp at second precision.  If no timestamp is provided, the time of this message is used. |
    | ssid | (Optional) SSID |
    | bssid | BSSID |
    | encryption | (Optional) String array of encryption types |
    | signal | Signal, in dBm |
    | count | (Optional) Number of times this result has been seen since the last scan report |
    | lat | (Optional) GPS latitude |
    | lon | (Optional) GPS longitude |
    | alt | (Optional) GPS altitude |
    | speed | (Optional) GPS speed |

* Results \
    `HTTP 200` on success
    HTTP error on failure

