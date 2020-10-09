---
title: "Phy80211 Wi-Fi"
permalink: /docs/devel/webui_rest/phy80211/
toc: true
docgroup: "devel-rest"
excerpt: "The 802.11 Wi-Fi subsystem defines a set of Wi-Fi specific APIs for accessing information about APs, related devices, and more."
---
The 802.11 Wi-Fi phy defines extra endpoints for manipulating Wi-Fi devices seen by Kismet, and for extracting packets of special types.

## WPA Handshakes
The WPA handshake is vital for extracting the WPA key of an encrypted WPA or WPA2 session.  Kismet will retain the handshake packets from an access point, and can provide them as a PCAP file.

* URL

    /phy/phy80211/by-key/*[DEVICEKEY]*/pcap/handshake.pcap

* API modified:

    `2020-10`

* Methods

    `GET`

* GET parameters

    | Key | Description |
    | --- | ---------- |
    | *[DEVICEKEY]* | Kismet device key of target device |

* Result

    On success: PCAP file of WPA handshake packets associated with the device, as well as a beacon packet.
    On error: HTTP error

## WPA PMKID
The WPA PMKID component of the handshake can be used to perform offline attacks against the WPA key using Aircrack or Hashcat.  Kismet will retain a packet with the RSN PMKID value, and can provide it as a PCAP file.

* URL

    /phy/phy80211/by-key/*[DEVICEKEY]*/pcap/handshake-pmkid.pcap

* API added:

    `2019-05`

* API modified:

    `2020-10`

* Methods

    `GET`

* GET parameters

    | Key | Description |
    | --- | ----------- |
    | *[DEVICEKY]* | Kismet device key of target device |

* Result

    On success: PCAP file of RSN PMKID packet, and a beacon packet.
    On error: HTTP error 

## Wi-Fi per-device pcap stream
Kismet can provide a streaming pcap-ng log of all packets, from all interfaces, associated with a given Wi-Fi BSSID.  Packets are streamed _starting when this endpoint is opened_, for past packtes, use the [KismetDB log API](/docs/devel/webui_rest/kismetdb/).

* URL

    /phy/phy80211/pcap/by-bssid/*[BSSID]*/packets.pcapng

* Methods

    `GET`

* URL parameters

    | Key | Description |
    | --- | ---- |
    | *[BSSID]* | BSSID retrieve packets from |

* Results

    A pcap-ng stream of packets which will stream indefinitely as packets are received.

* Notes

    See the [packet capture API](/docs/devel/webui-rest/packet_capture/) for more information about pcap-ng streams

## Wi-Fi clients
Kismet tracks client association with access points.  This information is available as a list of the device keys in the access point device record, but it is also available through the clients API which will return the complete device record of the associated client.

* URL \\
        /phy/phy80211/clients-of/*[DEVICEKEY]*/clients.json

* Methods \\
        `GET` `POST`

* URL parameters

| Key | Description |
| - | - |
| *[DEVICEKEY]* | Device to fetch clients of.  This should be an access point device; providing a non-access-point device will return an empty set. |

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Results \\
        An array of device records of associated clients.

## Access points only view
The 802.11 subsystem uses [device views](/docs/devel/webui_rest/device_views/) to provide a list of Wi-Fi access points.

* URL \\
        /devices/views/phydot11_accesspoints/...
        /devices/views/phydot11_accesspoints/devices.json
        /devices/views/phydot11_accesspoints/last-time/*[TIMESTAMP]*/devices.json

* Notes \\
        See the [views api](/docs/devel/webui_rest/device_views/) for more information

## Wi-Fi related devices
Kismet can provide a list of related devices.  Devices are related in 802.11 when they appear to be on the same physical network, or make up multiple BSSIDs in a roaming SSID.  This can be seen when multiple APs share the same SSID, common clients, and appear as clients of each other.

* URL \\
        /phy/phy80211/related-to/*[DEVICEKEY]*/devices.json

* Methods \\
        `GET` `POST`

* API added \\
        `2019-03`

* URL parameters

| Key | Description |
| - | - |
| *[DEVICEKEY]* | Device to fetch relationships for.  This device should be an access point.  Providing a non-access-point device will return an empty set. |

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Results \\
        An array of device records of related devices.

