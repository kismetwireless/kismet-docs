---
title: "System status"
permalink: /docs/devel/webui_rest/system_status/
toc: true
docgroup: "devel-rest"
excerpt: "Basic system status and health reporting."
---

## System status

* URL

    /system/status.json

* Methods

    `GET` `POST`

* Role

    `readonly`

    Errata:  Incorrectly marked as `logon` in 2020-R1

* POST parameters

    | Key    | Description                                                                             |
    | ---    | -----------                                                                             |
    | fields | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result

    Dictionary of Kismet system-level status, including update, battery, memory, and thermal data, optionally simplified.

## Timestamp

* URL

    /system/timestamp.json

* Methods

    `GET`

* Role

    `readonly`

    Errata:  Incorrectly marked as `logon` in 2020-R1

* Result

    Dictionary of system timestamp as second, microsecond; can be used to synchronize timestamps and as a keep-alive check.

## Tracked fields

* URL

    /system/tracked_fields.html

* Methods

    `GET`

* Role

    `readonly`

* Result

    Human-readable table of all registered field names, types, and descriptions.  While it cannot represent the nested features of some data structures, it will describe every allocated field.  This endpoint returns a HTML document for ease of use.

## Packet counts

* URL

    /packetchain/packet_stats.json

* Methods

    `GET` `POST`

* Role

    `readonly`

* POST parameters

    | Key    | Description                                                                             |
    | ---    | -----------                                                                             |
    | fields | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result

    Dictionary of packet rate RRDs for various packet queues

## Dynamic javascript include

Kismet provides a dynamically generated javascript file which is loaded by the UI, which in turn loads the core system and plugin JS modules.

* URL

    /dynamic.js

* API Added

    `2022-01`

* Methods

    `GET`

* Role

    `readonly` 

* Result

    This returns a full javascript file which in turn loads the JS modules registered with Kismet and inserts them into the global namespace.  It should be included as a script file in the UI, such as:

    ```html
    <script src="dynamic.js"></script>
    ```

