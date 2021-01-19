---
title: "ADSB"
permalink: /docs/devel/webui_rest/phyadsb/
toc: true
docgroup: "devel-rest"
excerpt: "Kismet can track aircraft using the ADSB transponder protocol using additional hardware"
---
The ADSB PHY defines a websocket endpoint for obtaining a live stream of ADSB data in the binary BEAST format expected by some tools.

## ADSB BEAST websocket

* URL

    /phy/RTLADSB/beast.ws

* API Added

    `2020-11`

* Methods

    `WEBSOCKET` (HTTP Upgrade + Websocket handshake)

* Role

    `readonly`, `ADSB`

* Result

    Streaming websocket output of ADSB data in BEAST format.

* Notes

    This can be transformed into a TCP socket for external tools which expect that via the `websocat` tool, for example:

    ```bash
    $ websocat ws://user:password@kismet-server-ip/phy/RTLADSB/beast.ws | nc -l 12345
    ```

## ADSB raw hex websocket

* URL

    /phy/RTLADSB/raw.ws

* API Added

    `2020-11`

* Methods

    `WEBSOCKET` (HTTP Upgrade + Websocket handshake)

* Role

    `readonly`, `ADSB`

* Result

    Streaming websocket output of ADSB data in hex format which matches the output format of `dump1090 --raw`

* Notes

    This can be easily dumped to tools which process hex ADSB streams with the `websocat` tool, for example:

    ```bash
    $ websocat ws://user:password@kismet-server-ip/phy/RTLADSB/raw.ws | some_tool...
    ```

## ADSB raw hex websocket per-source

* URL

    /datasource/by-uuid/*[UUID]*/adsb_raw.ws

* API Added

    `2021-01`

* Methods

    `WEBSOCKET` (HTTP Upgrade + Websocket handshake)

* Role

    `readonly`, `ADSB`

* Result

    Streaming websocket output of ADSB data in hex format which matches the output format of `dump1090 --raw`.  The feed is restricted to the specified source.

* Notes

    This can be easily dumped to tools which process hex ADSB streams with the `websocat` tool, for example:

    ```bash
    $ websocat ws://user:password@kismet-server-ip/phy/RTLADSB/raw.ws | some_tool...
    ```
