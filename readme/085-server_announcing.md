---
title: "Server Announcements"
permalink: /docs/readme/server_announcements/
excerpt: "Automatic server discovery via announcement"
docgroup: "readme"
toc: true
---

## Server announcements

*(Added 2020-08)*

Kismet supports automatic discovery of the server by remote capture sources via server announcements.

This is *disabled* by default.

If configured to, Kismet will announce itself via a UDP broadcast packet on UDP port 2501.  This packet contains the server UUID, name, timestamp, and web and remote capture ports.

Remote capture tools can use the `--autodetect` option to find the first advertising Kismet server on the local network, or the `--autodetect [uuid]` option to wait for a broadcast from a specific server UUID.

Typically this would be used when building a local network of remote capture devices, where the server IP may not be known, may change, or simply for convenience - for instance using a dedicated network for capture nodes throughout a building, or using an embedded device which serves DHCP to a laptop running Kismet.

### Configuration options

Server announcement is controlled by the `server_announce` options:

* `server_announce=true|false`

    Enable the announcement subsystem.  If not enabled, no announcements will be sent.

* `server_announce_address=[ip]`

    Limit the announcements to a specific interface, as controlled by IP.  By default this is `0.0.0.0`, or all interfaces, and typically this would be the desired configuration.

* `server_announce_port=2501`

    Configure the port server announcements are sent to.  Generally there is no reason to change this, and changing it will prevent other devices from finding it with the default configurations.


