---
title: "UAV / Drones"
permalink: /docs/devel/webui_rest/phyuav/
toc: true
docgroup: "devel-rest"
excerpt: "Kismet can track additional information about UAV/Drone/Quadcopter devices based on manufacturer, SSID, and packet contents."
---
The UAV/Drone phy defines extra endpoints for matching UAVs based on manufacturer and SSID.

## UAV manufacturers

* URL

    /phy/phyuav/manuf_matchers.json

    /phy/phyuav/manuf_matchers.ekjson

    /phy/phyuav/manuf_matchers.itjson

* Methods

    `GET` `POST`


* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key    | Description                                                                             |
    | ---    | -----------                                                                             |
    | fields | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result

    Array of manufacturer match records

