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

* Methods

    `GET`

* Role

    `readonly`

* Result

    Streaming websocket output of ADSB data in BEAST format.

* Notes

    This can be transformed into a TCP socket for external tools which expect that via the `websocat` tool, for example:

    ```bash
    $ websocat ws://user:password@kismet-server-ip/phy/RTLADSB/beast.ws | nc -l 12345
    ```

