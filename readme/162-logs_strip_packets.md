---
title: "Stripping Kismetdb packet data"
permalink: /docs/readme/kismetdb_strip_packets/
excerpt: "Kismetdb logs typically contain packet data; sometimes you may wish to strip the packet contents while keeping the device records."
docgroup: "readme"
toc: true
---

This tool is available as part of Kismet when built from source, or in the kismet-logtools package, as of `2019-02`.

## Packet data
Kismet stores packets as binary data in the [kismetdb log file](/docs/readme/logging).

Packet data can be invaluable for processing log files - but it can also take up space, and reveal sensitive information.  Before sharing a packet log (for instance with future sites which accept kismetdb logs directly), you should strip the packet content.

The `kismetdb_strip_packets` tool will retain all metadata - MAC addresses, signal, and location - but will erase the contents of the packets.

```bash
$ kismetdb_strip_packets --in some-kismet-file.kismet --out some-other-file.kismet
```

## Export options
There are several optional parameters you can use when exporting a JSON file:

* `--verbose`
    Add more status output to the console while `kismetdb_strip_packets` runs.

* `--force`
    By default, `kismetdb_strip_packets` will not overwrite the target file if it exists already.  `--force` will cause it to clobber the destination.

* `--skip-clean`
    By default, `kismetdb_strip_packets` runs a SQL Vacuum command to optimize the database and clean up any journal files.  Skipping this process will save time on larger captures.

