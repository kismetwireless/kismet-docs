---
title: "Device views"
permalink: /docs/devel/webui_rest/device_views/
toc: true
docgroup: "devel-rest"
excerpt: "A common 'device view' API which is used by many components of Kismet to present different views of the device data while retaining identical API calls."
---

## Device views

Device views are optimized subsets of the global device list.  Device views can be defined by PHY handlers, plugins, as part of the base Kismet code, or user-supplied data.

All device views respond to the same common API; any code which access a specific device view should be portable across multiple views.

### View list

The view list shows all defined device views and a summary of the number of devices in each.

* URL 

    /devices/views/all_views.json

    /devices/views/all_views.ekjson

    /devices/views/all_views.itjson

* Methods

    `GET`

* Role

    `readonly`

* Results

    Array of device views and device counts per view.

### View-based summarization and display

Mirroring the [base summarization & display endpoint](/docs/devel/webui_rest/devices/#old-summarization--display) API, the view summarization endpoint is the primary interface for clients to access the device list and for scripts to retrieve lists of devices.

The device summarization is best utilized when applying a view window via the `start` and `length` variables.

* URL

    /devices/views/*[VIEWID]*/devices.json

* Methods

    `POST`

* Role

    `readonly`

* URL parameters

    | Key        | Description    |
    | ---        | -----------    |
    | *[VIEWID]* | Kismet view ID |

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

    Summarized array of devices.

### Devices by view & time

Mirroring the [Activity & timestamp](/docs/devel/webui_rest/devices/#activity--timestamp) API, fetches devices from a specified view which have been active since the supplied timestamp.  This endpoint is typically used by scripted clients to monitor active devices within a view.

* URL

    /devices/views/*[VIEWID]*/last-time/*[TIMESTAMP]*/devices.json

    /devices/views/*[VIEWID]*/last-time/*[TIMESTAMP]*/devices.ekjson

    /devices/views/*[VIEWID]*/last-time/*[TIMESTAMP]*/devices.itjson

* Methods

    `GET` `POST`

* Role

    `readonly`

* URL parameters

    | Key           | Description                                                                  |
    | ---           | -----------                                                                  |
    | *[VIEWID]*    | Kismet view ID                                                               |
    | *[TIMESTAMP]* | Relative or absolute [timestamp](/docs/devel/webui_rest/commands/#timestamp) |

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key    | Description                                                                             |
    | ---    | -----------                                                                             |
    | fields | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |
    | regex  | Optional, [regular expression filter](/docs/devel/webui_rest/commands/#regex-filters)   |

* Results

    Array of devices in view *VIEWID* with activity more recent than *TIMESTAMP* with optional field simplification.


### Realtime device monitoring by view

Mirroring the [Realtime device monitoring](/docs/devel/webui_rest/devices/#realtime-device-monitoring) API, provides a subscription-based realtime push API for monitoring devices within a view.

By subscribing to devices, or groups of devices, a client can receive a websocket push event of device data.  This data can be simplified by a standard field simplification system. 

* URL 

    /devices/views/*[VIEWID]*/monitor.ws

* API added

    `2021-01`

* Methods

    `WEBSOCKET` (HTTP Upgrade + Websocket handshake)

* Role

    `readonly`

* URI parameters

    | Key           | Description                                                                  |
    | ---           | -----------                                                                  |
    | *[VIEWID]*    | Kismet view ID                                                               |
    | user     | Kismet administrative username, as HTTP URI-encoded variable |
    | password | Kismet administrative password, as HTTP URI-encoded variable |
    | KISMET   | Kismet auth cookie, as HTTP URI-encoded variable             |

* Result

    A websocket session with a subscription-model API

    HTTP error on failure

* Notes

    Kismet websockets will accept authentication as HTTP basic auth headers, Kismet session token cookies, or HTTP URI-encoded GET parameters of the basic auth or session cookie.

    The subscription and result API is [identical to the device monitoring API](/docs/devel/webui_rest/devices/#realtime-device-monitoring) API

