---
title: "Kismetdb to JSON"
permalink: /docs/readme/kismetdb_devices_json/
excerpt: "Kismetdb logs can be exported to JSON records describing all seen devices, making it easy to process capture history."
docgroup: "readme"
toc: true
---

This tool is available as part of Kismet when built from source, or in the kismet-logtools package, as of `2019-02`.

## Device JSON
Kismet stores devices it has seen in the [kismetdb log file](/docs/readme/logging/) as JSON dumps containing *everything* Kismet knows about a device.

Extracting these devices can be done simply using the `kismetdb_dump_devices` tool:

```bash
$ kismetdb_dump_devices --in some-kismet-file.kismet --out some-json-file.json
```

## Export options
There are several optional parameters you can use when exporting a JSON file:

* `--verbose`
    Add more status output to the console while `kismetdb_dump_devices` runs.

* `--force`
    By default, `kismetdb_dump_devices` will not overwrite the target file if it exists already.  `--force` will cause it to clobber the destination.

* `--skip-clean`
    By default, `kismetdb_dump_devices` runs a SQL Vacuum command to optimize the database and clean up any journal files.  Skipping this process will save time on larger captures.

* `--ekjson`
    Export as an `ekjson` format; Instead of exporting a JSON array of the devices, instead export each device as an object on a single line.  While not technically valid JSON, this format can be used to stream processing or inserting into other tools (such as ELK), and can be processed line-by-line with far fewer resources than a single array of all options.

## Streaming via `stdout`

Like many other command line tools, specifying `-` as the output file will cause `kismetdb_dump_devices` to stream the output to the console, making it simple to pipe it to other tools:

```bash
$ kismetdb_dump_devices --in some-kismetdb.kismet --out - | python -mjson.tool
```

