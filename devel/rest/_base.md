---
title: "xyz"
permalink: /docs/devel/webui_rest/xyz/
toc: true
---

## System status
* URL \\

* Methods \\
`GET` `POST`

* POST parameters \\
A [command dictionary](/docs/devel/webui_rest/commands/) containing:

| Key | Description |
| --- | ----------- |
| fields  | Optional, [field simplification](/docs/devel/webui_rest/commands/#field-specifications) |

* Result \\
Dictionary of Kismet system-level status, including update, battery, memory, and thermal data, optionally simplified.

