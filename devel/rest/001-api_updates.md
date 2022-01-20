---
title: "API Changes and Updates"
permalink: /docs/devel/webui_rest/changes/
docgroup: "devel-rest"
excerpt: "Significant changes to the API endpoints"
---

Over time, the Kismet endpoint API will change - while efforts are made to retain compatibility whenever possible, some changes will require breaking older implementations.  These significant changes will be documented here.

* `2022-01-R2`

    Changes to the REST API:

    * Added regex to [device view by time](/docs/devel/webui_rest/device_views/#devices-by-view--time)

* `2022-01-R1`

    Changes to the REST API:
    * Added the option to [filter pcap-ng streams](/docs/devel/webui_rest/kismetdb/#filter-options) by packet tag
    * Added [additional events](/docs/devel/webui_rest/eventbus/#dot11_advertised_ssid) to the eventbus, including the first packet for advertised SSIDs, responding SSIDs, and probed SSIDs.

    Changes to the kismetdb database:
    * Updated to version 8 of kismetdb, which [adds additional packet metadata](/docs/devel/kismetdb/#version-8)

    Changes to pcapng logging:
    * Added pcapng style CRC32 hashes and packet identifier numbers for multi-interface captures

    Changes to ipc/remote:
    * Added v2 ipc protocol to optimize for fast construction / zero memcpy

* `2021-05-R1` 

    Changes to the REST API:
    * Added a [view-specific device subscription API](/docs/devel/webui_rest/device_views/#realtime-device-monitoring-by-view) under `/device/views/[VIEWID]/monitor.ws` 
    * Added a [datasource-specific ADSB hex API](/docs/devel/webui_rest/phyadsb/#adsb-raw-hex-wbesocket-per-source) under `/datasource/by-uuid/[uuid]/adsb_raw.ws`
    * Added `class` and `severity` to [alert definitions](/docs/devel/webui_rest/alerts/#defining-alerts) and returned alerts

* `2020-12-R1` - Major rewrites of HTTP and networking, memory optimizations

    This version introduces a complete rewrite of the internal webserver implementation, changing the core webserver library to Boost::Beast and rewriting how endpoints are processed and parsed.

    Changes to the REST API:
    * 802.11 handshake pcaps are now found on `/phy/phy80211/by-key/[key]/pcap/handshake.pcap` and `/phy/phy80211/by-key/[key]/pcap/handshake-pmkid.pcap`.  The original MAC-based filenames are passed with the `attachment; filename=...` header.
    * Per-uuid pcapng streams are now found at `/datasource/pcap/by-uuid/[uuid]/packets.pcapng`
    * Per-device pcapng streams are now found at `/devices/pcap/by-key/[key]/packets.pcapng`
    * Phy80211 per-bssid pcap streams are now found at `/phy/phy80211/pcap/by-bssid/[mac]/packets.pcapng`
    * All REST endpoints in the API now use `cmd` as the file extension for all commands, deprecating and removing the `jcmd` extension fully (which has not been a documented command extension for several releases already).
    * Websockets are now implemented in the Kismet webserver, with the Eventbus websocket being the largest user.
    * `wget` is now supported by detection of the user-agent field; a full HTTP 401 and WWW-Authenticate header is sent to accommodate `wget` not sending basic-auth until it fails an auth check.
    * HTTP GET variables are now properly supported 
    * [API keys and roles](/docs/devel/webui_rest/logins/#api-tokens-and-roles) are now supported
    * JSON data accepted as application/json as well as application/x-form-urlencoded
    * Authentication may be passed as `user`, `password`, or `KISMET` GET URL variables

    New endpoints:
    * Realtime message, alert, GPS, system status, and more via the [eventbus](/docs/devel/webui_rest/eventbus/) websocket at `/eventbus/events.ws`
    * [API auth token manipulation](/docs/devel/webui_rest/logins/#api-tokens-and-roles) endpoints on `/auth/apikey/generate.cmd`, `/auth/apikey/revoke.cmd`, and `/auth/apikey/list.json`.
    * Runtime [changing of the devicefound/deviceleft alert list](/docs/devel/webui_rest/devices/#alerts---device-presence--absence---changing) via `/devices/alerts/mac/[type]/add.cmd`, `/devices/alerts/mac/[type]/remove.cmd` and `/devices/alerts/mac/[type]/macs.json`
    * Live [ADSB data](/docs/devel/webui_rest/phyadsb/) in text/hex mode via `ws://.../phy/RTLADSB/raw.ws`, streams a text-based hex output of the ADSB data
    * Live [ADSB data](/docs/devel/webui_rest/phyadsb/) in binary/beast mode via `ws://.../phy/RTLADSB/beast.ws`, streams a binary beast-protocol ADSB dump
    * Subscription-style [live device monitoring](/docs/devel/webui_rest/devices/#realtime-device-monitoring) websocket endpoint at `ws://.../devices/monitor.ws`


    Changes to data:
    * Locational data no longer includes the `avg_lat`, `avg_lon`, or `avg_alt` and related fields that held the *raw* averages of location.  The average location *is still present* in average location object, this removal affects only the internal raw values which were exposed to serialization.
    * Locational data no longer includes the `kismet.common.location.valid` field, as it was redundant - this data is contained in `kismet.common.location.fix` when the fix is >= 2.
    * Location data no longer includes the 'history cloud', an attempt to provide a RRD-like history log; it used way too much RAM and was not used anywhere.
    * Some locations (such as min/max/average) no longer track speed and heading, saving more ram
    * Server UUID in device records is now a common shared field under the name `kismet.server.uuid`
    * Non-json command endpoints (mostly) now return text/plain
    * RRDs no longer include the aggregator string name

* `2020-08` several maps made optional/dynamic

    To save RAM, several maps and vectors are now flagged as optional; if there is no content in those fields, they will not be present in the generated JSON.  Consumers should always check for presence of the map in the returned data before trying to use it.

    Fields made dynamic:
    * `kismet.device.base.tags`
    * `dot11.device/dot11.device.client_map`
    * `dot11.device/dot11.device.advertised_ssid_map`
    * `dot11.device/dot11.device.probed_ssid_map`
    * `dot11.device/dot11.device.associated_client_map`
    * `dot11.device/dot11.device.wpa_nonce_list`
    * `dot11.device/dot11.device.wpa_anonce_list`

    To save RAM, the `kismet.device.base.tags` sub-map is now optional; if there are no tags in a device, this field will not exist in the serialized JSON data.  Consumers of this data should check that `kismet.device.base.tags` is present in the device map.

* `2019-10` geopoint

    To more cleanly support ELK, all location records now use `geopoint` formats.  A geopoint is an array containing `[lon, lat]`.  `kismet.common.location.lat` and `kismet.common.location.lon` are now `kismet.common.location.geopoint`.

* `2019-10` advertised_ssid and probe_ssid as arrays

    To more cleanly support ELK the advertised ssid and probed ssid components of dot11 devices are now serialized as vectors instead of maps.  Previously these were serialized as maps with a key of the hash of the SSID and attributes (an essentialy meaningless number in the export).  Now these are sent as an array of advertised or probed SSID objects.


* `2019-10` ekjson and itjson

    To more cleanly support ELK EKJSON format, the `ekjson` serialization now permutes the field names to transform all `.` to `_`.  This brings it in line with the ELK interpretation that a `.` is a field separator.  To access the old implemention, where the field names are unmodified, use the new `itjson` (or 'iterative json') format; it will return results which are a vector of objects as an object per line, suitable for serialized parsing and processing. 

* `2019-04` Mandatory authentication

    Kismet now requires authentication on *ALL* endpoints, with the exclusion of `/system/user_status`, `/session/check_login`, `/session/check_session`, and `/session/check_setup_ok`.

