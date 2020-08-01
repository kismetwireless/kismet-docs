---
title: "API Changes and Updates"
permalink: /docs/devel/webui_rest/changes/
docgroup: "devel-rest"
excerpt: "Significant changes to the API endpoints"
---

Over time, the Kismet endpoint API will change - while efforts are made to retain compatibility whenever possible, some changes will require breaking older implementations.  These significant changes will be documented here.

* `2020-08` kismet.device.base.tags now optional

    To save RAM, the `kismet.device.base.tags` sub-map is now optional; if there are no tags in a device, this field will not exist in the serialized JSON data.  Consumers of this data should check that `kismet.device.base.tags` is present in the device map.

* `2019-10` geopoint

    To more cleanly support ELK, all location records now use `geopoint` formats.  A geopoint is an array containing `[lon, lat]`.  `kismet.common.location.lat` and `kismet.common.location.lon` are now `kismet.common.location.geopoint`.

* `2019-10` advertised_ssid and probe_ssid as arrays

    To more cleanly support ELK the advertised ssid and probed ssid components of dot11 devices are now serialized as vectors instead of maps.  Previously these were serialized as maps with a key of the hash of the SSID and attributes (an essentialy meaningless number in the export).  Now these are sent as an array of advertised or probed SSID objects.
    

* `2019-10` ekjson and itjson

    To more cleanly support ELK EKJSON format, the `ekjson` serialization now permutes the field names to transform all `.` to `_`.  This brings it in line with the ELK interpretation that a `.` is a field separator.  To access the old implemention, where the field names are unmodified, use the new `itjson` (or 'iterative json') format; it will return results which are a vector of objects as an object per line, suitable for serialized parsing and processing. 

* `2019-04` Mandatory authentication

    Kismet now requires authentication on *ALL* endpoints, with the exclusion of `/system/user_status`, `/session/check_login`, `/session/check_session`, and `/session/check_setup_ok`.

