---
title: "Pcap logging"
permalink: /docs/readme/logging_pcap/
excerpt: "Pcap logging format"
docgroup: "readme"
toc: true
---

## Pcap logging

Pcap (and pcapng) are standard formats for logging packets, which most tools which manipulate packets accept, including Tcpdump and Wireshark.  A kismetdb log can be converted to pcapng, or Kismet can write pcap logs directly.

There are two main flavors of pcap supported by Kismet:

1. `pcapppi`

    `pcapppi` is the legacy pcap format, with the PPI headers.  This version of the pcap log can only handle Wi-Fi packets, and manipulates the packet headers to fit the PPI standard (only one antenna signal value and other limitations).  Generally the `pcapng` format should be used, but not all tools understand the new format.

2. `pcapng`

    `pcapng` is the modern pcap standard supported by wireshark, tshark, and other tools.

    Pcap-ng allows for mixing packet types (such as Wi-Fi, BTLE, and Zigbee) as well as retaining the original capture interface (which datasource in Kismet saw the packet), the original headers (pure radiotap packet headers with more complete signal info, timestamps, etc).  It also allows Kismet to log the packet without manipulation (such as header translation to PPI), allows for annotations, other data, and more.

### Log type

```
log_types=pcapppi
```

and

```
log_types=pcapng
```


## Filtering

### Duplicate packets

Kismet can record duplicate packets when multiple datasources capture the same data.  In some instances, keeping duplicate packets is desirable (such as remote captures used for device location), while in others, logging duplicates may be a waste of space.

```
pcapng_log_duplicate_packets=false
```

and

```
ppi_log_duplicate_packets=false
```

### Data packets

Kismet typically logs all types of packets.  To discard data packets, retaining only management frames:

```
pcapng_log_data_packets=false
```

and

```
ppi_log_data_packets=false
```

