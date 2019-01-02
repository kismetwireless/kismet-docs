---
title: "Extending device and data tracking"
permalink: /docs/devel/dot11_ie_parsers/
toc: true
---
Adding parsers for new IE tags in Kismet is relatively simple; there's a handful of files you need to modify and coding conventions you should follow, however.

## Kaitai
Kismet uses the [Kaitai](https://kaitai.io) library for processing stream data, however it does *not* (currently) use the Kaitai parser generator.  Kismet parsers are written manually, mostly because of the difficulty of generating long-lifecycle code safely from KaitaiStruct templates; errors encountered in the stream could leave memory leaks which were unrecoverable.

The Kaitai stream library contains safe mechanisms for accessing elements of various types in a stream.  If the stream is unable to satisfy the requested data, an exception is raised.  All interaction with tag parsers should be wrapped in `try/catch` blocks, and a tag parser is free to throw exceptions as necessary.

## Tags
IE tags are found in various management frames:  Beacons, probe responses, probe requests, association responses, and more.  They are a list of fields, each field containig a tag number, tag length, an the tag data.

The Kismet tag parser will break tags into independent values automatically, assigning a Kaitai stream to the data of each tag.

An IE tag parser needs to only worry about the data!

## Parser header
The tag (and other parsers) for 802.11 packets are kept in `dot11_parsers/`.

A theoretical parser, at its simplest, would be defined as:

```c++
class dot11_ie_12345_fake {
public:
    dot11_ie_12345_fake() { }
    ~dot11_ie_12345_fake() { }

    void parse(std::shared_ptr<kaitai::kstrem> p_io);

    constexpr17 uint8_t fake_value() const {
        return m_fake_value;
    }

    constexpr17 uint16_t fake_twobyte() const {
        return m_fake_twobyte;
    }

protected:
    uint8_t m_fake_value;
    uint16_t m_fake_twobyte;
};
```

IE field parsers are named after the IE tag and the common name:  `dot11_ie_*tagnum*_*tag name*`; for instance `dot11_ie_11_qbss` or `dot11_ie_221_vendor`.  While verbose, this makes reading the code which uses the parsers FAR simpler in the future.

The constructor and destructor are empty - a tag parser, typically, should not define any dynamic memory that is not managed by smart pointers such as `shared_ptr`.

The accessor functions are, typically, defined using the Kismet workaround for variable `constexpr` support.  `constexpr17` is defined by Kismet in `multi_constexpr.h` and evaluates to `constexpr` only if the compiler supports C++17 or newer.

Accessors should be named after the field they return, which should in turn be named after the field definition in the protocol specification.

The field variable names should follow the Kaitai auto-generated code convention of beginning with `m_` and being named after the field they are derived from.

Typically, the accessors are defined entirely in the header file, while the `parse` function is defined in the source file.

## Parser source
Generally, the `parse(...)` function is the only function defined in the source.

For our fake tag, we assume a tag contains an 8 bit unsigned value, and a 16 bit unsigned value.  The 16 bit value is encoded as little-endian.

```c++
dot11_ie_12345_fake::parse(std::shared_ptr<kaitai::kstream> p_io) {
    m_fake_value = p_io->read_u8();
    m_fake_twobyte = p_io->read_u2le();
}
```

Notice that we don't need to check the length of the fields; the Kaitai stream library handles this for us automatically.  Since the stream buffer is created with the tag data and the specified tag length, the parser does not need to worry about running into the next tag, nor running into the end of the packet headers.


## A more complex example

Let's look at the IE 11 QBSS source for a slightly more complex example:
```c++
#include "dot11_ie_11_qbss.h"
#include "fmt.h"

void dot11_ie_11_qbss::parse(std::shared_ptr<kaitai::kstream> p_io) {
    // V1
    if (p_io->size() == 4) {
        m_station_count = p_io->read_u2le();
        m_channel_utilization = p_io->read_u1();
        m_available_admissions = p_io->read_u1();
        return;
    } 

    // V2
    if (p_io->size() == 5) {
        m_station_count = p_io->read_u2le();
        m_channel_utilization = p_io->read_u1();
        m_available_admissions = p_io->read_u2le();
        return;
    }

    throw std::runtime_error(fmt::format("dot11_ie_11_qbss expected v1 (4 bytes) or v2 (5 bytes), "
                "got {} bytes", p_io->size()));
}
```

The QBSS tag may take 2 forms, depending on version; the only way to determine which form is in play is to check the size of the stream buffer.  For version 1, 4 bytes are read, while for version 2, 5 bytes are read.

The parser throws an exception if the buffer is not either 4 or 5 bytes long.  Typically, a parser may use Kaitai stream processing to handle raising the exception, but because the QBSS tag version is length-dependent, we explicitly raise an exception if an invalid length was found.

## 150 and 221 vendor tags
The IE150 and IE221 tags are somewhat special:  Both define vendor extensions, which are then nested under an OUI and sub-type record.  Typically IE150 holds Cisco-specific data about the access point configuration, while IE221 is a catch-all for WMM, QoS, encryption settings, unique vendor tags, and other unique vendor data.

The parsing of the IE150 and IE221 tags is handled by Kismet, breaking each tag into sub-tags, similar to how Kismet breaks the IE data into individual fields.  Kismet extracts the vendor OUI and subtype, and starts the sub-tag stream immediately *after* the OUI record.  Sub-tag parsers *must process the subtype byte*, as not all sub-types *use* a type byte.

Subtypes are defined the same way as IE parsers, with a few additional values; using the `dot11_ie_221_ms_wps` sub-type parser as an example:

```c++
    constexpr17 static uint32_t ms_wps_oui() {
        return 0x0050f2;
    }

    constexpr17 static uint8_t ms_wps_subtype() {
        return 0x04;
    }
```

Each subtype should export a static integer version of the vendor OUI, and if applicable, the subtype.  This allows the consuming code to quickly compare subtypes to apply the parser.

The parse function is implemented the same as before, with the additional consumption of the 1 byte value.  Using our previous example of parsing a 1 byte and 2 byte value, as a 221 subtype:

```c++
dot11_ie_221_fake_sub::parse(std::shared_ptr<kaitai::kstream> p_io) {
    // read and throw away the subtype
    p_io->read_u8();

    // Read the content
    m_fake_value = p_io->read_u8();
    m_fake_twobyte = p_io->read_u2le();
}
```

## Nested complexity
Some elements themselves include a list of data; for example the IE221 MS-WPS tag contains a list of sub-elements.

Typically these should be defined as sub-classes, each with their own `parse(...)` function.  This allows consistent naming and stream validation.

Nested vectors or maps of data *should always use smart pointers*.  This *ensures that memory is not lost in the parsing of tags*. 

Looking again at the relevant portions of `dot11_ie_221_ms_wps`:

```c++
class dot11_ie_221_ms_wps {
public:
    class wps_de_sub_element;

    typedef std::vector<std::shared_ptr<wps_de_sub_element> > shared_wps_de_sub_element_vector;

    dot11_ie_221_ms_wps() { }
    ~dot11_ie_221_ms_wps() { }

    void parse(std::shared_ptr<kaitai::kstream> p_io);

    ...

protected:
    uint8_t m_vendor_subtype;
    std::shared_ptr<shared_wps_de_sub_element_vector> m_wps_elements;

    ...
```

Later in `dot11_ie_221_ms_wps` we define the sub-element class:

```c++
public:
    class wps_de_sub_element {
    public:
        class wps_de_sub_common;
        class wps_de_sub_string;
        class wps_de_sub_rfband;
        class wps_de_sub_state;
        class wps_de_sub_uuid_e;
        class wps_de_sub_vendor_extension;
        class wps_de_sub_version;
        class wps_de_sub_primary_type;
        class wps_de_sub_ap_setup;
        class wps_de_sub_generic;

        ...

        wps_de_sub_element() {};
        ~wps_de_sub_element() {};

        void parse(std::shared_ptr<kaitai::kstream> p_io);

        constexpr17 wps_de_type_e wps_de_type() const {
            return (wps_de_type_e) m_wps_de_type;
        }

        ...
    }
```

The WPS definitions are very convoluted - the `de_sub_element` in turn defines a number of additional classes.  While verbose, this methodology is at least consistent, readable, and easier to validate.

Finally, the parser function:

```c++
void dot11_ie_221_ms_wps::parse(std::shared_ptr<kaitai::kstream> p_io) {
    m_vendor_subtype = p_io->read_u1();
    m_wps_elements.reset(new shared_wps_de_sub_element_vector());
    while (!p_io->is_eof()) {
        std::shared_ptr<wps_de_sub_element> e(new wps_de_sub_element());
        e->parse(p_io);
        m_wps_elements->push_back(e);
    }
}
```

The `parse` function does 3 things:

1. Read the subtype byte.  We store this instead of discarding it, in this parser.
2. Zero the vector
3. Iterate over the remaining tag data.  It is important to check the stream length here!  The data should match the length of the buffer.  By checking the length, we *prevent* a false failure when we hit the end of the buffer trying to read another record - but we *also* ensure that the entire buffer is read.

## Bitfields
Many tags define bitfields.  Generally, it's best to implement the bitfields so that the code using the API calls a function returning a boolean (or a shifted integer for multi-bit values); for instance from `dot11_ie_54_mobility`:

```c++
    constexpr17 uint8_t mobility_policy() const {
        return m_mobility_policy;
    }

    constexpr17 unsigned int policy_fastbss_over_ds() const {
        return mobility_policy() & 0x01;
    }

    constexpr17 unsigned int policy_resource_request_capability() const {
        return mobility_policy() & 0x02;
    }
```

The general goal is to shift as much of the tag-specific logic into the tag parser, and away from the caller.

## Parsing tags
Parsing tags is handled by `phy_80211_dissectors.cc`.  The values are put into the procesed `dot11_packinfo` defined in `phy_80211.h`.

Processed records can be added to `dot11_packinfo` as data, or as shared pointers to processed tags:

```c++
        ...
        std::shared_ptr<dot11_ie_61_ht_op> dot11ht;
        std::shared_ptr<dot11_ie_192_vht_op> dot11vht;
        ...
        unsigned int ccx_txpower;
        bool cisco_client_mfp;
        ...
```

Tag processing happenns in `phy_80211_dissectors.cc` in the `Kis_80211_Phy::PacketDot11IEDissector(...)` function.  This function builds the hash of IE tag content and processes the tags by number:

```c++
        ...
        // IE 11 QBSS
        if (ie_tag->tag_num() == 11) {
            try {
                auto qbss = std::make_shared<dot11_ie_11_qbss>();
                qbss->parse(ie_tag->tag_data_stream());
                packinfo->qbss = qbss;
            } catch (const std::exception& e) {
                fprintf(stderr, "debug - corrupt QBSS %s\n", e.what());
                packinfo->corrupt = 1;
                return -1;
            }
            continue;
        }
        ...
```

The parser structure breaks each tag apart and iterates over them in the `ie_tag` iterator.  To process the content of the tag using our parser, we define our tag and pass the tag stream to the parser.  For QBSS, we use smart pointers to keep a copy of the tag in the `packinfo` record.

## Adding tags to devices
To add tags to devices, a new `TrackedElement` field must be added to the appropriate object in the device record; these are generally defined in `phy_80211.h`.  You will need to define the proper `__Proxy`` functions and registration functions, as described in [Extending device and data records](/docs/devel/data_types):

```c++
class dot11_advertised_ssid : public tracker_component {
public:
    ...
    __Proxy(cisco_client_mfp, uint8_t, bool, bool, cisco_client_mfp);
    ...

protected:

    virtual void register_fields() override {
        ...
        RegisterField("dot11.advertisedssid.cisco_client_mfp",
                "Cisco client management frame protection", &cisco_client_mfp);
        ...
    }

    ...

    // Cisco frame protection
    std::shared_ptr<TrackerElementUInt8> cisco_client_mfp;
};
```

With the field defined, you will then need to extract the data from the `dot11_packinfo` record at the appropriate stage in `phy_80211.cc` and place it in the 802.11 device record.  Since IE tags happen in beacons, probes, and associations, there are several places you may wish to add code to handle tags:

* `Kis_80211_Phy::HandleSSID(...)` which handles beacons and probe and association responses.
* `Kis_80211_Phy::HandleProbedSSID(...)` which handles probe request and association request.

For instance:

```c++
void Kis_80211_Phy::HandleSSID(std::shared_ptr<kis_tracked_device_base> basedev,
        std::shared_ptr<dot11_tracked_device> dot11dev,
        kis_packet *in_pack,
        dot11_packinfo *dot11info,
        kis_gps_packinfo *pack_gpsinfo) {

    ...

        // Set the mobility
        if (dot11info->dot11r_mobility != NULL) {
            ssid->set_dot11r_mobility(true);
            ssid->set_dot11r_mobility_domain_id(dot11info->dot11r_mobility->mobility_domain());
        }

        // Set tx power
        ssid->set_ccx_txpower(dot11info->ccx_txpower);

        // Set client mfp
        ssid->set_cisco_client_mfp(dot11info->cisco_client_mfp);

    ...
}
```

