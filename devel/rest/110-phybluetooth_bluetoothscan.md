---
title: "PhyBluetooth Scanning-mode Sources"
permalink: /docs/devel/webui_rest/phybluetooth_scansource/
toc: true
docgroup: "devel-rest"
excerpt: "A simple API for non-packet-capture bluetooth scanning results"
---

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

* URL 

    /phy/phybluetooth/scan/scan_report.cmd

* API added 

    `2020-07`

* Methods 

    `POST` 

* POST parameters 

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key         | Description                                                                      |
    | ---         | -----------                                                                      |
    | reports     | Array containing multiple report objects                                         |
    | source_name | A unique, consistent source name for the virtual datasource reporting this scan. |
    | source_uuid | A unique, consistent source UUID for the virtual datasource reporting this scan. |

    A report object should contain:

    | Key          | Description                                                                                                                                                                                  |
    | ---          | -----------                                                                                                                                                                                  |
    | timestamp    | (Optional) Unix timestamp at second precision.  If no timestamp is provided, the time of this message is used.  Due to general lack of precision of scanning mode, timestamp is second only. |
    | btaddr       | Bluetooth MAC address                                                                                                                                                                        |
    | name         | (Optional) Advertised device name                                                                                                                                                            |
    | devicetype   | (Optional) Device type, if known                                                                                                                                                             |
    | txpowerlevel | (Option) Device advertised powerlevel (Integer)                                                                                                                                              |
    | pathloss     | (Optional) Path loss (Integer)                                                                                                                                                               |
    | signal       | (Optional) Signal, in dBm (Integer)                                                                                                                                                          |
    | scan_data    | (Optional) Binary scan data, as hex string                                                                                                                                                   |
    | service_data | (Optional) Dictionary of service UUID to service scan data as hex strings                                                                                                                    |
    | lat          | (Optional) GPS latitude (Float)                                                                                                                                                              |
    | lon          | (Optional) GPS longitude (Float)                                                                                                                                                             |
    | alt          | (Optional) GPS altitude (Float)                                                                                                                                                              |
    | speed        | (Optional) GPS speed (Float)                                                                                                                                                                 |

* Results 

    `HTTP 200` on success

    HTTP error on failure

