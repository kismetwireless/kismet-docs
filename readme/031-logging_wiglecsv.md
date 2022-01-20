---
title: "Wigle logging"
permalink: /docs/readme/logging_wigle/
excerpt: "Wigle logging format"
docgroup: "readme"
toc: true
---

## Wigle logging

Kismet can convert a `kismetdb` log to a CSV format suitable to upload to [Wigle](https://wigle.net), but can also directly write a `wiglecsv` file.

### Log type

```
log_types=wiglecsv
```

### Wiglecsv

A Wigle log does not contain packet data - it is a simplified CSV file of Wi-Fi access points and Bluetooth devices.

Since Wigle is centered around mapping wireless, Kismet will only create entries in the `wiglecsv` log when a GPS is connected and a location is available.
