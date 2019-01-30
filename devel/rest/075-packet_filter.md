---
title: "Packet filters"
permalink: /docs/devel/webui_rest/packet_filters/
toc: true
---

## Packet filters
Packet filtering is used by Kismet to limit the packets in some fashion; typically to restrict the packets being logged, returned in packet streams, and similar functions.

The packet filtering system, like [device views](/docs/devel/webui_rest/device_views), functions as a common layer which is mapped by different components.

### Filter logic

Kismet filters activate to *block* packets:  A *positive* filter type *filters* or *excludes* the packet.  When represented as boolean or integer values, a filter type of `true` or any non-zero value will *exclude* that match.

Filter terms may match on packet attributes, dependent on the type of filter.  Matches can operate as `filter` or `pass` to explicitly allow or block a match.

Packets which do not match any filter terms are handled by the filter `default` behavior, which can be used to accept or reject all non-matching packets.

## Common filter status
* URL \\
        /packetfilters/*[FILTERID]*/filter.json

* Methods \\
`GET` `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result \\
Dictionary of filter status, including description, default behavior, and type, optionally simplified.  Additionally, the filter results may contain the filter terms, depending on the type of filter (for instance, `mac_addr` filters contain the mac address filtering tables).


