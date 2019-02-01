---
title: "Keys and MAC addresses"
permalink: /docs/devel/webui_rest/keys_and_macs/
toc: true
---

## MAC addresses
The MAC address is a theoretically unique identifier given to a device at manufacture time.  For Ethernet and Wi-Fi devices, this is assigned by the IEEE, and must be unique.

Typically a MAC address is 6 bytes.  Kismet supports MAC addresses up to 8 bytes, however no PHYs currently use these.

Kismet will attempt to synthesize MAC addresses for PHYs which do not present traditional MAC addresses.

## Keys
Kismet uses a unique key for each device.  The key is *derived from* the MAC address, but contains additional information about the PHY type.  This allows multiple devices to have the same MAC address under different PHY types without overlap; this can be of particular importance when using non-traditional PHY types which do not use strict device identifiers.

## MAC address masking
On queries and filters affecting MAC addresses, Kismet supports complete addresses or partial addresses with masking.

A masked address resembles the syntax typically used for IP network masking: `[MAC]/[MASK]`.

For instance, to match only the OUI, a masked MAC of:

```json
"aa:bb:cc:00:00:00/ff:ff:ff:00:00:00"
```

This would match any MAC address where the OUI, or first three bytes, are "aa:bb:cc".  A similar feature to match on the first *four* bytes would be:

```json
"aa:bb:cc:dd:00:00/ff:ff:ff:ff:00:00"
```


