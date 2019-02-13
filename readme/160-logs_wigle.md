---
title: "Kismet and Wigle"
permalink: /docs/readme/wigle/
excerpt: "Kismet integration with Wigle"
toc: true
---

## Wigle
[Wigle](https://www.wigle.net) is a world-wide wardriving database which tracks wireless networks and their locations.

Contributions to Wigle come from the community.

You can contribute to Wigle by converting your logs to the Wigle CSV format, using the `kismetdb_to_wiglecsv` tool.

## Converting

The simplest way to convert a kismetdb log to a wiglecsv is to simply run the conversion tool:

```bash
$ kismetdb_to_wiglecsv --in some-kismet-log-file.kismet --out some-wigle-file.csv
```

This tool is available as part of Kismet when built from source, or in the kismet-logtools package, as of Feb 2019.

## Conversion options

Converting a kismetdb log can take a lot of space and time, because each packet is examined and the coordinates written to the CSV file.  This can be sped up with various options to the `kismetdb_to_wiglecsv` tool:

* `--verbose`
    Add more status output to the console while `kismetdb_to_wiglecsv` runs.

* `--skip-clean`
    By default, `kismetdb_to_wiglecsv` runs a SQL Vacuum command to optimize the database and clean up any journal files.  Skipping this process will save time on larger captures.

* `--rate-limit [rate]`
    Limit the export rate to `[rate]` seconds per device; an appropriate rate limit would depend on general speed you traveled during the capture.

    Even limiting to 1 second between updates can significantly reduce the size of the wiglecsv file.

* `--cache-limit [limit]`
    `kismetdb_to_wiglecsv` will cache device information; by default, it will cache 1000 devices at a time.  If you have a very large number of devices, and a lot of RAM, increasing this may make the conversion run faster.

## Uploading to Wigle
Once your log is converted, you can upload it to [Wigle](https://www.wigle.net) by creating an account there and choosing the file from your computer.

