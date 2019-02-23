---
title: "Filters"
permalink: /docs/devel/webui_rest/filters/
toc: true
---

## Packet filters
Packet filtering is used by Kismet to limit the packets in some fashion; typically to restrict the packets being logged, returned in packet streams, and similar functions.

The packet filtering system, like [device views](/docs/devel/webui_rest/device_views), functions as a common layer which is mapped by different components.

### Filter logic

Kismet filters activate to *block* packets:  A *positive* filter type *filters* or *excludes* the packet.  When represented as boolean or integer values, a filter type of `true` or any non-zero value will *exclude* that match.

Filter terms may match on packet attributes, dependent on the type of filter.  Matches can operate as `filter` or `pass` to explicitly allow or block a match.

Packets which do not match any filter terms are handled by the filter `default` behavior, which can be used to accept or reject all non-matching packets.

The filter engine recognizes several terms when setting filtering:  `true`, `reject`, `deny`, `filter`, and `block` are equivalent terms for telling the filter to exclude a match.  `false`, `allow`, `pass`, and `accept` are equivalent terms for allowing a packet to pass a filter and be allowed.


## Common packet filter status
* URL \\
        /filters/packet/*[FILTERID]*/filter.json

* Methods \\
`GET` `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result \\
Dictionary of filter status, including description, default behavior, and type, optionally simplified.  Additionally, the filter results may contain the filter terms, depending on the type of filter (for instance, `mac_addr` filters contain the mac address filtering tables).

## Common packet filter defaults
The default method is applied to all packets which do not match any terms in the filter.  

__LOGIN REQUIRED__

* URL \\
        /filters/packet/*[FILTERID]*/set_default.cmd

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| default  | String value of the default behavior of the filter (`reject` or `allow` for instance) |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

## MAC address packet filter blocks
MAC address filters use the type `mac_filter`, and filter (perhaps obviously) by MAC address.

Address filters can be applied to:
* *source* - Original source device.  In Wi-Fi networks, equivalent to the source MAC; in other phy types, typically the originating device.
* *destination* - Target device.  In Wi-Fi networks, the destination MAC; in other phy types, if present, the equivalent destination address.
* *network* - Associated network.  In Wi-Fi, this is the BSSID.
* *other* - Other address; in Wi-Fi this is the fourth MAC found in WDS; in other phy types it represents some form of alternate address.
* *any* - Matching any of the address fields.

Address filters are applied in the above order:  `source`, `destination`, `network`, `other`, `any`, `default`.  If an address is accepted by the `source` stage and would be rejected by the `destination` stage, the filter will *accept* the packet, as this is the first operation.

### Defining filters
Filters can be added to any of the filter blocks; `source`, `destination`, `network`, `other`, or `any`.

__LOGIN REQUIRED__

* URL \\
        /filters/packet/*[FILTERID]*/*[BLOCKNAME]*/set_filter.cmd

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| filter | Dictionary object where the MAC address is the key and a boolean filter term is the value.  These filters will be added to the block identified by *[BLOCKNAME]*. A value of `true` indicates the matching MAC address *will be blocked*, while a value of `false` indicates the matching MAC address *will be passed*.  [Masked MAC addresses](/docs/devel/webui_rest/keys_and_macs/) can be supplied for bulk matching. |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

* Notes \\
Packets which do not match a filter term will be passed to the default behavior of the filter.  Setting a filter to `false` does *not remove the match*.  Use the *filter removal API* to remove matches.

### Removing filters
Filters can be removed from any of the filter blocks; `source`, `destination`, `network`, `other`, or `any`.

__LOGIN REQUIRED__

* URL \\
        /filters/packet/[*FILTERID]*/*[BLOCKNAME]*/remove_filter.cmd

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| addresses | Array of MAC addresses (or masked MAC addresses) to be removed from the target filter block identified by *[BLOCKNAME]*. |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

## Class filters
Class-based filtering is used by Kismet to limit logging events, devices, or other non-packet options.

The class filtering system, like [device views](/docs/devel/webui_rest/device_views), functions as a common layer which is mapped by different components.

### Filter logic

Kismet filters activate to *block* matches:  A *positive* filter type *filters* or *excludes* the match.  When represented as boolean or integer values, a filter type of `true` or any non-zero value will *exclude* that match.

Filter terms may match on many attributes, dependent on the type of filter.  Matches can operate as `filter` or `pass` to explicitly allow or block a match.

Objects which do not match any filter terms are handled by the filter `default` behavior, which can be used to accept or reject all non-matching objects.

The filter engine recognizes several terms when setting filtering:  `true`, `reject`, `deny`, `filter`, and `block` are equivalent terms for telling the filter to exclude a match.  `false`, `allow`, `pass`, and `accept` are equivalent terms for allowing an object to pass a filter and be allowed.

## Common class filter status
* URL \\
        /filters/class/*[FILTERID]*/filter.json

* Methods \\
`GET` `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result \\
Dictionary of filter status, including description, default behavior, and type, optionally simplified.  Additionally, the filter results may contain the filter terms, depending on the type of filter (for instance, `mac_addr` filters contain the mac address filtering tables).

## Common class filter defaults
The default method is applied to all packets which do not match any terms in the filter.  

__LOGIN REQUIRED__

* URL \\
        /filters/class/*[FILTERID]*/set_default.cmd

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| default  | String value of the default behavior of the filter (`reject` or `allow` for instance) |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

## MAC address class filter blocks
MAC address filters use the type `mac_filter`, and filter (perhaps obviously) by MAC address.

Address filters apply to a single record, depending on the use of the filter.

### Defining filters
__LOGIN REQUIRED__

* URL \\
        /classfilters/*[FILTERID]*/*[PHYNAME]*/set_filter.json

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| filter | Dictionary object where the MAC address is the key and a boolean filter term is the value.  These filters will be added to the block identified by *[BLOCKNAME]*. A value of `true` indicates the matching MAC address *will be blocked*, while a value of `false` indicates the matching MAC address *will be passed*.  [Masked MAC addresses](/docs/devel/webui_rest/keys_and_macs/) can be supplied for bulk matching. |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

* Notes \\
        Packets which do not match a filter term will be passed to the default behavior of the filter.  Setting a filter to `false` does *not remove the match*.  Use the *filter removal API* to remove matches. \\
        Phy names which do not exist at the time of the filter being defined will be matched if a compatible phy type is registered in the future.
        

### Removing filters
__LOGIN REQUIRED__

* URL \\
        /classfilters/[*FILTERID]*/*[PHYNAME]*/remove_filter.json

* Methods \\
        `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| addresses | Array of MAC addresses (or masked MAC addresses) to be removed from the target filter block identified by *[BLOCKNAME]*. |

* Result \\
        `HTTP 200` on success \\
        HTTP error on failure

