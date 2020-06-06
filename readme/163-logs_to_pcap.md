---
title: "Kismetdb to PCAP"
permalink: /docs/readme/kismetdb_to_pcap/
excerpt: "Kismetdb logs can be easily converted to pcap format"
docgroup: "readme"
toc: true
---

## Packet data
Kismet stores packets as binary data in the [kismetdb log file](/docs/readme/logging).

Most tools like [Wireshark](https://www.wireshark.org), [tcpdump](https://www.tcpdump.org), [Aircrack-NG](https://www.aircrack-ng.org), and many more, use the PCAP format, or the more modern variant, the PCAP-NG format.

The PCAP-NG format allows for mixing different types of data (for instance, Wi-Fi and Bluetooth) into one logfile, and preserves which capture source it was received on, but isn't well supported by all tools (Wireshark and tshark offer excellent support, however).

## Converting packets with `kismetdb_to_pcap` (Added 2020-06)

Installed automatically (from source) or as part of the kismet-logtools package (if installing from packages), `kismetdb_to_pcap` converts the Kismet logs to standard PCAP and PCAP-NG.

### Basic converting

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcapng
```

This convert the log to a standard pcapng logfile.  This file contains the most information and is most useful in tools like Wireshark.

If you have only one type of data - for instance, Wi-Fi packets captured from a single interface - this file will be usable with any tool which uses libpcap (such as aircrack, tcpdump, and almost all other tools.)

### Legacy PCAP

`kismetdb_to_pcap` can log to legacy PCAP files as well:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap --old-pcap
```

Legacy PCAP files are limited to one DLT, or link type; the link type is the type of packet, for instance raw 802.11, radiotap signal headers, Bluetooth, and so on.

Legacy PCAP files have no concept of interfaces or data sources, so if you have multiple datasources in Kismet, all the packets will be available, but it will be impossible to see what source originally captured each packet, unless you split by datasource (more on this in the next section).

If your kismetdb log has more than one link type, you can specify which one will be included in the legacy pcap using the `--dlt` option:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap --old-pcap --dlt 127
```

To see what linktypes are included in your kismetdb log, use the `--list-datasources` option (see the next section for more).

### Listing and selecting datasources

`kismetdb_to_pcap` can list the datasources and what link types each has captured:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --list-datasources
* Found KismetDB version 6
* Collecting info about datasources...
Datasource #0 (5FE308BD-0000-0000-0000-00C0CAA6846C xenon-mt2 wlx00c0caa6846c) 766980 packets
   DLT 127: IEEE802_11_RADIO 802.11 plus radiotap header
Datasource #1 (5FE308BD-0000-0000-0000-00C0CAA68473 xenon-mt1 wlx00c0caa68473) 704950 packets
   DLT 127: IEEE802_11_RADIO 802.11 plus radiotap header
Datasource #2 (5FE308BD-0000-0000-0000-00C0CAA68471 xenon-mt0 wlx00c0caa68471) 3656794 packets
   DLT 127: IEEE802_11_RADIO 802.11 plus radiotap header
Datasource #3 (689C0913-0000-0000-0000-0000865F0805 rtladsb-0 rtladsb-0) 0 packets
   No packets seen by this datasource
Datasource #4 (5FE308BD-0000-0000-0000-9CEFD5FDD05C xenon-rt28 wlx9cefd5fdd05c) 0 packets
   No packets seen by this datasource
```

Each datasource has a unique identifier, or UUID.  Because multiple datasources could have the same interface (for example when using remote capture), datasources must be referred to by UUID.

Logs can be extracted for one or more datasources:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap --old-pcap --datasource 5FE308BD-0000-0000-0000-00C0CAA6846C --datasource 5FE308BD-0000-0000-0000-00C0CAA68473
```

would generate a legacy PCAP log with only the first and second interfaces.


### Splitting logs

If you have multiple datasources and want to generate a log file for each, or extremely large log files and want to split the logs by packet count or by log size, `kismetdb_to_pcap` can do that, as well:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap --old-pcap --split-datasources 
```

will make a pcap for each datasource named `some-kismet-log.kismet-[uuid]`.

The `--split-packets [#]` and `--split-size [kb]` options allow splitting packets by count or by total packet size in Kb:

```bash
$ kismetdb_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap --old-pcap --split-packets 10000
```

will make a pcap every 10000 packets, named `some-pcap-log.pcap-[XXXXXX]`.

The `--split-datasources` option can be combined with the `--split-packets` or the `--split-size` options.


### More info

More information is available via the `--help` option:

```bash
$ kismetdb_to_pcap --help
Convert packet data from KismetDB logs to standard pcap or pcapng logs for use in
tools like Wireshark and tcpdump
usage: ./log_tools/kismetdb_to_pcap [OPTION]
 -i, --in [filename]            Input kismetdb file
 -o, --out [filename]           Output file name
 -f, --force                    Overwrite any existing output files
 -v, --verbose                  Verbose output
 -s, --skip-clean               Don't clean (sql vacuum) input database
     --old-pcap                 Create a traditional pcap file
                                Traditional PCAP files cannot have multiple link types.
     --dlt [linktype #]         Limit pcap to a single DLT (link type); necessary when
                                generating older traditional pcap instead of pcapng.
     --list-datasources         List datasources in kismetdb; do not create a pcap file
     --datasource [uuid]        Include packets from this datasource.  Multiple datasource
                                arguments can be given to include multiple datasources.
     --split-datasource         Split output into multiple files, with each file containing
                                packets from a single datasource.
     --split-packets [num]      Split output into multiple files, with each file containing
                                at most [num] packets
     --split-size [size-in-kb]  Split output into multiple files, with each file containing
                                at most [kb] bytes
```

