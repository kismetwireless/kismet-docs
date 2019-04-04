
---
title: "Kismetdb Statistics"
permalink: /docs/readme/kismetdb_statistics/
excerpt: "Quick analysis tool for KismetDB"
toc: true
---

This tool is available as part of Kismet when built from source, or in the kismet-logtools package, as of `2019-04`.

## Statistics / summary

The kismetdb stores the packets, interfaces, devices, and other data about a Kismet session.  The `kismetdb_statistics` tool is an easy interface to analyze a capture and return information about it such as time, device counts, location coverage, etc.

```bash
$ kismetdb_statistics --in some-kismet-file.kismet
```

### Example output
```
  KismetDB version: 5

  Packets: 447
  Non-packet data: 8

  Devices: 133
  Devices seen between: 2019-04-02 17:10:05 (1554239405) to 2019-04-02 17:20:46 (1554240046)
  2 datasources
    carnuc-rtl433    rtl433-0         5E600813-0000-0000-0000-00005DBB0805 rtl433
      Hardware: Generic RTL2832U OEM
      Packets: 8
    carnuc-mediatek  wlx000e8e5c8866  5FE308BD-0000-0000-0000-000E8E5C8866 linuxwifi
      Hardware: mt76x2u
      Packets: 424
      Hop rate: 5.000000/second
      Hop channels: 1, 1HT40+, 2, 3, 4, 5, 6, 6HT40-, 6HT40+, 7, 8, 9, 10, 11, 11HT40-, 12, 13, 14, 36, 36HT40+, 36VHT80, 40, 40HT40-, 44, 44HT40+, 48, 48HT40-, 52, 52HT40+, 52VHT80, 56, 56HT40-, 60, 60HT40+, 64, 64HT40-, 100, 100HT40+, 100VHT80, 104, 104HT40-, 108, 108HT40+, 112, 112HT40-, 116, 116HT40+, 116VHT80, 120, 120HT40-, 124, 124HT40+, 128, 128HT40-, 132, 132HT40+, 136, 136HT40-, 140, 140HT40-, 149, 149HT40+, 149VHT80, 153, 153HT40-, 157, 157HT40+, 161, 161HT40-, 165, 165HT40-

  Bounding location: 48.000000,-70.000000 49.000000,-75.000000 (~11.006206 Km)
  Packets with location: 447
  Data with location: 8
```

## Outputting to JSON
`kismetdb_statistics` can also output a JSON record; this can be used to index log files programmatically based on log attributes.

```bash
$ kismetdb_statistics --in some_file.kismet --json 
```

### Example output
```json
{
  "data_packets": 8,
  "datasources": [
    {
      "definition": "rtl433-0:name=carnuc-rtl433",
      "hardware": "Generic RTL2832U OEM",
      "interface": "rtl433-0",
      "name": "carnuc-rtl433",
      "packets": 8,
      "type": "rtl433",
      "uuid": "5E600813-0000-0000-0000-00005DBB0805"
    },
    {
      "definition": "wlx000e8e5c8866:name=carnuc-mediatek",
      "hardware": "mt76x2u",
      "hop_channels": [                                                                                                            
        "1",                                                                                                                       
        "1HT40+",                                                                                                                  
        "2",                                                                                                                       
        "3",                                                                                                                       
        "4",                                                                                                                       
        "5",                                                                                                                       
        "6",                                                                                                                       
        "6HT40-",                                                                                                                  
        "6HT40+",                                                                                                                  
        "7",                                                                                                                       
        "8",                                                                                                                       
        "9",                                                                                                                       
        "10",                                                                                                                      
        "11",                                                                                                                      
        "11HT40-",                                                                                                                 
        "12",                                                                                                                      
        "13",                                                                                                                      
        "14",                                                                                                                      
        "36",                                                                                                                      
        "36HT40+",                                                                                                                 
        "36VHT80",                                                                                                                 
        "40",                                                                                                                      
        "40HT40-",                                                                                                                 
        "44",                                                                                                                      
        "44HT40+",                                                                                                                 
	  ],
      "hop_rate": 5,
      "interface": "wlx000e8e5c8866",
      "name": "carnuc-mediatek",
      "packets": 424,
      "type": "linuxwifi",
      "uuid": "5FE308BD-0000-0000-0000-000E8E5C8866"
    }
  ],
  "device_max_time": 1554240046,
  "device_min_time": 1554239405,
  "devices": 133,
  "diag_distance_km": 11.00620557382399,
  "file": "/home/dragorn/wavehack/carnuc-20190402-17-10-03-1.kismet",
  "kismetdb_version": 5,
  "max_lat": 40.000000000,
  "max_lon": -70.000000000,
  "min_lat": 45.000000000,
  "min_lon": -75.000000000,
  "packets": 447
}
```


## Options
There are several optional parameters you can use:

* `--skip-clean`
    By default, `kismetdb_strip_packets` runs a SQL Vacuum command to optimize the database and clean up any journal files.  Skipping this process will save time on larger captures.

* `--json`
    Output as a JSON file; useful when using the `kismetdb_statistics` tool to populate an index of log files or similar.
