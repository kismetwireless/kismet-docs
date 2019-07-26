---
title: "Kismet and KML"
permalink: /docs/readme/kml/
excerpt: "Kismetdb logs can be easily exported to KML for use with Google Earth"
docgroup: "readme"
toc: true
---

This tool is available as part of Kismet when built from source, or in the kismet-logtools package, as of `2019-07-git`.

## KML
KML is an XML-based markup language for use with Google Earth.

## Converting

The simplest way to convert a kismetdb log to a KML is to simply run the conversion tool:

```bash
$ kismetdb_to_kml --in some-kismet-log-file.kismet --out some-kml-file.kml
```

## Conversion options

* `--verbose`

    Add more status output to the console while `kismetdb_to_kml` runs.

* `--skip-clean`

    By default, `kismetdb_to_kml` runs a SQL Vacuum command to optimize the database and clean up any journal files.  Skipping this process will save time on larger captures.

* `--basic-location`

    By default, `kismetdb_to_kml` computes a final average across all the packets seen; this can be more precise than the running average Kismet computes.  If packets were not logged, or to save time and processing, passing `--basic-location` will use the average location computed runtime, instead.

