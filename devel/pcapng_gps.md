---
title: "Pcap-NG GPS"
permalink: /docs/devel/pcapng-gps/
toc: true
docgroup: "devel"
excerpt: "Embedded GPS in pcap-ng"
---

## GPS data and Pcap-NG

The pcap-ng format does not currently define a standard block or option for GPS data; in the meantime, Kismet uses a standard pcap-ng custom extension under the Kismet IANA PEN to encode this data.

You can follow the development of the GPS standard at [the pcap-ng github issue](https://github.com/pcapng/pcapng/issues/48)

Once the standard is adopted, Kismet will transition to using the standard options.

The Kismet GPS implementation is directly modeled on the PPI GPS implementation by Jon Ellch and Harris.

## Kismet inclusion of GPS data

As of `2021-03` Kismet can include GPS data in pcap-ng when using the `kismetdb_to_pcap` tool;  ultimately this data will be encoded in streamed and directly logged pcap-ng files however while it is under development it is only available when converting from kismetdb logfiles.

## Kismet IANA PEN

The [IANA](https://pen.iana.org/) maintains a registry of *Private Enterprise Numbers*, which are unique identifiers.  

The Kismet PEN used for custom options and blocks in the pcap-ng file is:

`55922`

## Kismet GPS block

The Kismet GPS block consists of the following:

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | GPS Magic(0x47)| Version      |            Length             |
     +---------------------------------------------------------------+
     |                  GPS Fields Presence Bitmask                  |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     /                            GPS Data                           /  
     /              variable length, padded to 32 bits               /
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The GPS magic field must always be `0x47` (ASCII `G`).  This indicates to parsers that this custom tag contains Kismet GPS data.

The GPS version field indicates the version of this standard; currently this version is `0x1`.

The length field should be the length of the GPS data, as dictated by the options set in the GPS fields presence bitmask.

Available fields and sizes are noted below.

The GPS data record must be padded to 32 bits to comply with the pcap-ng framing standards.

Multi-byte values are encoded as the endian format of the pcap-ng file; parsers should detect this via the pcap-ng endian magic field and process accordingly.

### GPS fields

| Field               | ID    | Size (bytes) | Units       | Description                                                                                         |
| -----               | --    | ----         | -----       | -----------                                                                                         |
| Longitude           | 0x2   | 4            | Degrees     | Longitude, encoded as a fixed 3_7 value                                                             |
| Latitude            | 0x4   | 4            | Degrees     | Latitude, encoded as a fixed 3_7 value                                                              |
| Altitude            | 0x8   | 4            | Meters      | Altitude in meters encoded as a fixed 6_4 value                                                     |
| Altitude_G          | 0x10  | 4            | Meters      | Altitude *from ground*, encoded as a fixed 6_4 value (currently unused by Kismet)                   |
| GPS time            | 0x20  | 4            | Seconds     | Seconds since unix epoch UTC (currently unused by Kismet)                                           |
| GPS fractional time | 0x40  | 4            | Nanoseconds | Unsigned counter, nanosecond resolution.  Should not exceed one second (currently unused by Kismet) |
| EPH                 | 0x80  | 4            | Meters      | Estimated horizontal error as a fixed 6_4 value (currently unused by Kismet)                        |
| EPV                 | 0x100 | 4            | Meters      | Estimated vertical error as a fixed 6_4 value (currently unused by Kismet)                          |

## Number encoding

To facilitate cross-platform encoding of variable precision floating and double-precision floating numbers, all floating-point data is represented as one of the following:

| Encoding | Range                         | Precision | Use                                                                                                |
| -------- | -----                         | --------- | ---                                                                                                |
| fixed3_6 | 000.000000 to +999.999999     | 3.6       | Positional error estimates, angular rotations and error estimates, antenna beamwidth, antenna gain |
| fixed3_7 | -180.0000001 to +180.0000001  | 3.7       | Latitude and longitude                                                                             |
| fixed6_4 | -180000.0001 to + 180000.0001 | 6.4       | Altitude, positional offsets, velocity, acceleration                                               |

### Fixed3_6

An example implementation of encoding and decoding fixed3_6 would be:

```C++
uint32_t float_to_fixed3_6(float flt) {
    if (flt <= -000.000001)
        throw std::runtime_error("invalid value");
    if (flt > +999.999999)
        throw std::runtime_error("invalid value");

    return (uint32_t) flt * 1000000;
}

float fixed3_6_to_float(uint32_t fixed) {
    if (fixed > 1000000000)
        throw std::runtime_error("invalid value");

    return (double) (fixed / 1000000.0);
}
```

#### Fixed3_6 encoding examples

| Float       | Encoded    | Comments            |
| -----       | -------    | --------            |
| 000.000000  | 0000000000 | Zero                |
| 000.000001  | 0000000001 | One millionth       |
| 001.000000  | 0001000000 | One                 |
| 123.123456  | 0123123456 |                     |
| 360.000000  | 0360000000 |                     |
| 999.999999  | 0999999999 | Largest legal value |
| 1000.000000 | 1000000000 | Illegal value       |

### Fixed3_7

Latitude and longitude are typically represented as double-precision floating point values between -180.0000000 and +180.0000000.  In order to eliminate storing a signed value, these are mapped to the range (0, 3600000000) with a fixed decimal point at the third digit.  This allows any longitude and latitude to be represented in 4 bytes with 7 digits of precision to the right of the decimal.  Any values between 3600000001 (0x0xD693A401) and 4294967295 (0xFFFFFFFF) are invalid

An example implementation of encoding and decoding fixed3_7 would be:

```C++
uint32_t float_to_fixed3_7(double flt) {
    if (flt <= -180.0000001) 
        throw std::runtime_error("invalid value");
    if (flt >= +180.0000001)
        throw std::runtime_error("invalid value");
 
    int32_t scaled = (int32_t) ((flt) * (double) 10000000);
    return = (u_int32_t) (scaled + ((int32_t) 180 * 10000000));
}

double fixed3_7_to_float(uint32_t fixed) {
    if (fixed > 3600000000)
        throw std::runtime_error("invalid value");
 
    int32_t remapped = fixed – (180 * 10000000);
    return (double) ((double) remapped / 10000000);
}
```

#### Fixed3_7 encoding examples

| Double       | Encoded    | Comments             |
| -----        | -------    | --------             |
| -180.0000001 | -          | Illegal value        |
| -180.0000000 | 0000000000 | Smallest legal value |
| -179.9999999 | 0000000001 |                      |
| 000.0000000  | 1800000000 | Zero                 |
| +123.1234567 | 3031234567 |                      |
| +179.9999999 | 3599999999 |                      |
| +180.0000000 | 3600000000 | Largest legal value  |
| +180.0000001 | 3600000001 | Illegal value        |

### Fixed6_4

Most other data such as altitude does not need high precision, but may need larger range; the fixed6_4 encoding can express -180000.0000 to 180000.0000.

An example implementation of encoding and decoding fixed3_7 would be:

```C++
uint32_t float_to_fixed3_7(double flt) {
    if (flt <= -180000.0001) 
        throw std::runtime_error("invalid value");
    if (flt >= +180000.0001)
        throw std::runtime_error("invalid value");
        
    int32_t scaled_l = (int32_t) ((flt) * (double) 10000);
    return (u_int32_t) (scaled_l + ((int32_t) 180000 * 10000)); 
}

double fixed3_7_to_float(uint32_t fixed) {
    if (fixed > 3600000000)
        throw std::runtime_error("invalid value");

    int32_t remapped = fixed – (180000 * 10000);
    return (double) ((double) remapped / 10000);
}
```

#### Fixed6_4 encoding examples

| Double       | Encoded       | Comments                        |
| -----        | -------       | --------                        |
| -180000.0001 | Illegal value |
| -180000.0000 | 0000000000    | Most negative expressible value |
| -179999.9999 | 0000000001    |                                 |
| -010000.0000 | 0800000000    | Marianas trench (approx)        |
| 000000.0000  | 1800000000    | Sea level                       |
| +000000.0001 | 1800000001    | Sea level plus .0001 meters     |
| +021000.0123 | 2010000123    | Very high altitude flight       |
| +179999.9999 | 3599999999    |                                 |
| +180000.0000 | 3600000000    | Most positive expressible value |
| +180000.0001 | Illegal value |                                 |

## Role in pcap-ng

The Kismet GPS block can be contained in pcap-ng as an option on an enhanced packet EPB block, or as a custom block type.  When found as an option in the EPB, it is the GPS information for that specific packet, and when found as a custom block type, it is the track data or location of the system at that time, but not otherwise tied to a specific packet.

### GPS custom option

The Kismet GPS data follows the rules of the pcap-ng custom options:  It must be identified as a custom binary option, using the Kismet IANA PEN, and padded to 32 bits.

An EPB with the Kismet GPS data would look like:

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +---------------------------------------------------------------+
    0 |                    Block Type = 0x00000006                    |
      +---------------------------------------------------------------+
    4 |                      Block Total Length                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    8 |                         Interface ID                          |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   12 |                        Timestamp (High)                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   16 |                        Timestamp (Low)                        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   20 |                    Captured Packet Length                     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   24 |                    Original Packet Length                     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   28 /                                                               /
      /                          Packet Data                          /
      /              variable length, padded to 32 bits               /
      /                                                               /
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      /                                                               /
      /                      Options (variable)                       /
      /                                                               /
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |     Custom Option Code (2989) |         Option Length         |
      +---------------------------------------------------------------+
      |                Private Enterprise Number (55922)              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | GPS Magic(0x47)| Version      |            Length             |
      +---------------------------------------------------------------+
      |                  GPS Fields Presence Bitmask                  |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      /                            GPS Data                           /  
      /              variable length, padded to 32 bits               /
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                      End-of-options block                     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                      Block Total Length                       |
      +---------------------------------------------------------------+
```

### GPS custom block

A custom pcap-ng Kismet GPS block would look like:

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +---------------------------------------------------------------+
    0 |                    Block Type = 0x00000BAD                    |
      +---------------------------------------------------------------+
    4 |                      Block Total Length                       |
      +---------------------------------------------------------------+
    8 |                Private Enterprise Number (55922)              |
      +---------------------------------------------------------------+
      | GPS Magic(0x47)| Version      |            Length             |
      +---------------------------------------------------------------+
      |                  GPS Fields Presence Bitmask                  |
      +---------------------------------------------------------------+
      /                            GPS Data                           /  
      /              variable length, padded to 32 bits               /
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      /                      Options (variable)                       /
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                      Block Total Length                       |
      +---------------------------------------------------------------+
```

## Current implementation

Currently, no tools exist to read the custom fields from the pcapng; any extraction of the data must be done with custom tools.

Any tool capable of parsing pcap-ng files should be able to process a GPS tagged file without incident, but will not read the data.

