---
title: "KismetDB file source"
permalink: /docs/readme/datasources_kismetdb/
excerpt: "Kismetdb capture file replay"
toc: true
---

## Replaying data
Added: 2019-03

Kismet can replay recorded data in the `kismetdb` format, the unified log created by Kismet. 

Kismet can replay a pcapfile for testing, debugging, demo, or reprocessing.

A `kismetdb` file can contain packets and device data from any source Kismet handles.

### Datasource - kismetdb

The kismetdb datasource will auto-detect kismetdb files and paths to files:
```bash
$ kismet -c /tmp/foo.kismet
```

It can be manually specified with `type=kismetdb`, as in:

```
source=/tmp/foo.kismet:type=kismetdb
```

The kismetdb capture uses the 'kismet_cap_kismetdb' tool which does not need special privileges.

When replying a kismetdb file, data from multiple interfaces will appear as though it is from a single capture source - the kismetdb file.

### Kismetdb Options
In addition to the normal options supported by all sources (name, information elements, UUID, etc) the kismetdb source can also support:

* `pps=rate`
   Normally, kismetdb files are replayed as quickly as possible.  On larger logs this can lead to CPU and RAM contention, and dropped packets.  Specifying a packets-per-second rate throttles processing of the packet to a more sustainable speed.
   This option cannot be combined with the `realtime` option.

* `realtime=true | false`
   Normally, kismetdb files are replayed as quickly as possible.  On larger logs this can lead to CPU and RAM contention, and dropped packets.  Specifying `realtime=true` in your source definition will reduce the packet processing rate to the original capture rate, and the packets will be processed with real-time delays equal to how they were received.
   
 These options can be used in the kismet.conf and kismet_site.conf, as in:
 
 ```
 source=/tmp/foo.kismet:type=kismetdb,name=a_meaningful_name,realtime=true
 ```
 
 Or from the command line:
 
 ```bash
$ kismet -c /tmp/foo.kismet:name=a_meaningful_name,pps=1000
```
 
 

