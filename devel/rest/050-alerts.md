---
title: "Alerts"
permalink: /docs/devel/webui_rest/alerts/
toc: true
docgroup: "devel-rest"
excerpt: "The alerts API allows for fetching raised alerts, defining new custom alerts purely via the API interface, and raising alerts via the API interface, allowing external tools to tie into the Kismet alert subsystem."
---
Kismet alerts notify the user of critical Kismet events and wireless intrusion events.  Alerts are generated as messages (sent via [the messagebus](/docs/devel/webui_rest/messages/)) and as alert records.

For real-time monitoring of alerts, see the [eventbus](/docs/devel/webui_rest/eventbus/).

## Alert severities

Alerts severities are categorized by numerical value; a higher number is more severe.

| Severity | Defintion | Use                                                                                            |
| -------: | --------- | ---                                                                                            |
| 0        | INFO      | Informational alerts, such as datasource errors, Kismet state changes, etc                     |
| 5        | LOW       | Low-risk events such as probe fingerprints                                                     |
| 10       | MEDIUM    | Medium-risk events such as denial of service attempts                                          |
| 15       | HIGH      | High-risk events such as fingerprinted watched devices, denial of service attacks, and similar |
| 20       | CRITICAL  | Critical errors such as fingerprinted known exploits                                           |

## Alert types

Alerts are categorized by type; alert types are free-form strings, but include:

| Type    | Use                                                                |
| ----    | ---                                                                |
| DENIAL  | Possible denial of service attack                                  |
| EXPLOIT | Known fingerprinted exploit attempt against a vulnerability        |
| OTHER   | General category for alerts which don't fit in any existing bucket |
| PROBE   | Probe by known tools                                               |
| SPOOF   | Attempt to spoof an existing device                                |
| SYSTEM  | System events, such as log changes, datasource errors, etc         |

## Alert configuration

Kismet exposes the full alert system configuration, including currently supported alert types, full descriptions of alert content, and time and burst-rate delivery limiting.

* URL

    /alerts/definitions.json

    /alerts/definitions.ekjson

    /alerts/definitions.itjson

* Methods

    `GET`

* Role

    `readonly`

* Result 

    Array of all alert records.

## All alerts

Kismet retains the past *N* alerts, as defined by `alertbacklog` in `kismet_memory.conf`.  By default, Kismet retains 50 alert records.

* URL 

    /alerts/all_alerts.json

    /alerts/all_alerts.ekjson

    /alerts/all_alerts.itjson

* Methods

    `GET`

* Role

    `readonly`

* Result

    Array of all currently stored alerts

## Recent alerts

Alerts can be fetched by timestamp, returning only new alerts.  This API takes a specialized timestamp value which includes microsecond precision.

* URL

    /alerts/last-time/*[TIMESTAMP.UTIMESTAMP]*/alerts.json

    /alerts/last-time/*[TIMESTAMP.UTIMESTAMP]*/alerts.ekjson

    /alerts/last-time/*[TIMESTAMP.UTIMESTAMP]*/alerts.itjson

* Methods

    `GET`

* Role

    `readonly`

* URL parameters

    | Key                      | Description                                                                                                           |
    | ---                      | -----------                                                                                                           |
    | *[TIMESTAMP.UTIMESTAMP]* | A double-precision timestamp of the Unix epochal second timestamp *and* a microsecond precision sub-second timestamp. |

* Result

    An array containing an array of alerts since *TIMESTAMP.UTIMESTAMP*.

## Recent alerts (wrapped)

Alerts can be fetched by timestamp, returning only new alerts.  This API takes a specialized timestamp value which includes microsecond precision.

This endpoint returns the exact timestamp, with microsecond precision, of the returned alerts; this allows a client UI to accurately display only the new alerts.

* URL

    /alerts/last-time/*[TIMESTAMP.UTIMESTAMP]*/alerts.json

* Methods

    `GET`

* Role

    `readonly`

* URL parameters

    | Key                      | Description                                                                                                           |
    | ---                      | -----------                                                                                                           |
    | *[TIMESTAMP.UTIMESTAMP]* | A double-precision timestamp of the Unix epochal second timestamp *and* a microsecond precision sub-second timestamp. |

* Result

    A *dictionary* containing an array of alerts since *TIMESTAMP.UTIMESTAMP* and a double-precision timestamp second and microsecond timestamp of the current server time.

## Alert View

Alerts can be accessed via a jquery-datatables compatible backend, which interprets datatables requests with searches, windowed views, and server-side sorting.

* URL

    /alerts/alerts_view.json

* API Added

    `2021-03`

* Methods

    `POST` `GET`

* Role

    `readonly`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key       | Description                                                                                                                                                 |
    | -------   | -----------------------------------------------------                                                                                                       |
    | fields    | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications)                                                                     |
    | regex     | Optional, [regular expression filter](/docs/devel/webui_rest/commands/#regex-filters)                                                                       |
    | colmap    | Optional, inserted by the Kismet Datatable UI for mapping column information for proper ordering and sorting.                                               |
    | datatable | Optional, inserted by the Kismet Datatable UI to enable datatable mode which wraps the output in a container suitable for consumption by jquery-datatables. |

    Additionally, when in datatables mode, the following HTTP POST variables are used:

    | Key                  | Description                                                 |
    | ---                  | ----                                                        |
    | start                | Data view window start position                             |
    | length               | Datatable window end                                        |
    | draw                 | Datatable draw value                                        |
    | search[value]        | Search term, applied to all fields in the summary vector    |
    | order\[0\]\[column\] | Display column number for sorting, indexed with colmap data |
    | order\[0\]\[dir\]    | Sort order direction from jquery-datatables                 |

* Results 

    Summarized array of devices and jquery-datatables metadata

## Alert details

Returns details on a specific alert, as identified by the alert identifier hash in the alert record.

* URL

    /alerts/by-id/*[ALERTID]*/alert.json

* Methods

    `GET` `POST`

* Role

    `readonly`

* URL parameters

    | Key         | Description     |
    | ---         | -----------     |
    | *[ALERTID]* | Kismet alert ID |

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key    | Description                                                                             |
    | ---    | -----------                                                                             |
    | fields | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Results

    Detailed information about specified alert, or HTTP error on failure / invalid alert ID.

## Defining alerts

New alerts can be defined runtime, and triggered by external tools via the REST API.

* URL

    /alerts/definitions/define_alert.cmd

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key         | Description                                                                                                                                                                                                                                          |
    | ----------- | ----------------------------------------                                                                                                                                                                                                             |
    | name        | Simple alert name/identifier                                                                                                                                                                                                                         |
    | class | Alert class (such as SYSTEM, FINGERPRINT, TREND, etc) |
    | severity | Alert severity (as number) |
    | description | Alert explanation / definition displayed to the user                                                                                                                                                                                                 |
    | phyname     | (Optional) name of phy this alert is associated with.  If not provided, alert will apply to all phy types.  If provided, the defined phy *must* be found or the alert will not be defined.                                                           |
    | throttle    | Maximum number of alerts per time period, as defined in kismet.conf.  Time period may be 'sec', 'min', 'hour', or 'day', for example '10/min'                                                                                                        |
    | burst       | Maximum number of sequential alerts per time period, as defined in kismet.conf.  Time period may be 'sec', 'min', 'hour', or 'day'.  Alerts will be throttled to this burst rate even when the overall limit has not been hit.  For example, '1/sec' |

* Results

    `HTTP 200` on success 

    HTTP error on failure

## Raising alerts

Alerts can be triggered by external tools; the alert must be defined, first.

* URL

    /alerts/raise_alerts.cmd

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key     | Description                                                                        |
    | ------- | ----------------------------------------                                           |
    | name    | Alert name/identifier.  Must be a defined alert name.                              |
    | text    | Human-readable text for alert                                                      |
    | bssid   | (optional) MAC address of the BSSID, if Wi-Fi, related to this alert               |
    | source  | (optional) MAC address the source device which triggered this alert                |
    | dest    | (optional) MAC address of the destination device which triggered this alert        |
    | other   | (optional) Related other MAC address of the event which triggered this alert       |
    | channel | (optional) Phy-specific channel definition of the event which triggered this alert |

* Result

    `HTTP 200` on success

    HTTP error on failure


