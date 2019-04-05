---
title: "Phy802.11 SSID Scan module"
permalink: /docs/devel/webui_rest/phy80211_ssidscan/
toc: true
docgroup: "devel-rest"
excerpt: "Still under development, the ssidscan module will allow for targetting devices by SSID and automatically searching for behavior."
---

## SSIDScan status
Current configuration and status of the ssidscan module, including the target SSIDs and assigned datasources.

__LOGIN REQUIRED__

* API added \
    `2019-04`

* URL \
    /phy/phy80211/ssidscan/status.json

* Methods \
    `POST` `GET`

* POST parameters \
    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key | Description |
    | --- | ----------- |
    | fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Results \
    `HTTP 200` on success
    HTTP error on failure

## SSIDScan configuration
Push new configuration options to the ssidscan module, overriding any configuration present.

__LOGIN REQUIRED__

* API added \
    `2019-04`

* URL \
    /phy/phy80211/ssidscan/config.cmd

* Methods \
    `POST` 

* POST parameters \
    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key | Description |
    | --- | ----------- |
    | ssidscan_enabled | Optional, boolean, enable or disable the ssidscan module |
    | ignore_after_handshake | Optional, boolean, ignore a target device once a WPA handshake has been captured |
    | max_capture_seconds | Optional, unsigned integer, maximum seconds to capture before returning to hop |
    | min_scan_seconds | Optional, unsigned integer, minumum seconds to scan before entering capture mode |
    | restrict_log_filters | Optional, restrict the kismetdb log to log only devices and packets meeting ssidscan targets.  This will set filters but not remove existing logs. |
    | locking_datasources | Optional, vector of UUID strings of datasources assigned to the 'locking' pool to capture target devices |
    | hopping_datasources | Optional, vector of UUID strings of datasources assigned to the 'hopping' pool to scan for target devices |

* Results \
    `HTTP 200` on success
    HTTP error on failure

