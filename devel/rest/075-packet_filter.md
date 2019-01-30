---
title: "Packet filters"
permalink: /docs/devel/webui_rest/packet_filters/
toc: true
---

## Packet filters
Packet filtering is used by Kismet to limit the packets in some fashion; typically to restrict the packets being logged, returned in packet streams, and similar functions.

The packet filtering system, like [device views](/docs/devel/webui_rest/device_views), functions as a common layer which is mapped by different components.

## System status
* URL \\

* Methods \\
`GET` `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result \\
Dictionary of Kismet system-level status, including update, battery, memory, and thermal data, optionally simplified.

