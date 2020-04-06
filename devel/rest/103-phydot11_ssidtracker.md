---
title: "Device views"
permalink: /docs/devel/webui_rest/phy80211_ssid_tracker/
toc: true
docgroup: "devel-rest"
excerpt: "A dedicated SSID aggregator API"
---

## SSID tracker
The SSID tracker is a phy80211 specific mechanism for mapping SSID broadcast, probe, and response.

A unique identifier is generated from the SSID content, length, and encryption options.  

Each SSID is mapped to a list of device keys organized by probe, response, and advertisement.

### SSID-based summarization and display
Mirroring the [base summarization & display endpoint](/docs/devel/webui_rest/devices/#old-summarization--display) API, the SSID summarization endpoint is the primary interface for clients to access the SSID list and for scripts to retrieve lists of SSIDs.

The SSID summarization is best utilized when applying a view window via the `start` and `length` variables.

* URL \\
        /phy/phy80211/ssids/views/ssids.json

* Methods \\
`POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key     | Description                                           |
| ------- | ----------------------------------------------------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |
| regex   | Optional, [regular expression filter](/docs/devel/webui_rest/commands/#regex-filters) |
| colmap  | Optional, inserted by the Kismet Datatable UI for mapping column information for proper ordering and sorting. |
| datatable | Optional, inserted by the Kismet Datatable UI to enable datatable mode which wraps the output in a container suitable for consumption by jquery-datatables. |

Additionally, when in datatables mode, the following HTTP POST variables are used:

| Key | Description |
| --- | ---- |
| start  | Data view window start position |
| length | Datatable window end |
| draw   | Datatable draw value |
| search[value] | Search term, applied to all fields in the summary vector |
| order\[0\]\[column\] | Display column number for sorting, indexed with colmap data |
| order\[0\]\[dir\] | Sort order direction from jquery-datatables |

* Results \\
        Summarized array of SSIDs

