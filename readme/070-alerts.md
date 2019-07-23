---
title: "Alerts and WIDS"
permalink: /docs/readme/alerts_and_wids/
excerpt: "Kismet can also function as a WIDS (Wireless Intrusion Detection System) with configurable alerts."
docgroup: "readme"
---

## Alerts and WIDS
Kismet can function as a WIDS (Wireless Intrusion Detection System) with configurable alerts for both stateless and stateful fingerprint and trend based monitoring.  Kismet fingerprint alerts can trigger on known-hostile specific behavior such as attacks against wireless drivers, and trend-based monitors can detect unusual behavior over time, such as flooding and denial of service attacks.

Kismet can integrate with other tools via the live packet export REST API, the Kismet alert REST API, and via syslog and some SIEM tools.

Kismet is most effective as an IDS in stationary (ie, non-wardriving) capacity.  Kismet WIDS functionality *can* be used in mobile and channel-hopping installations, but accuracy and coverage may suffer.

### Configuring alerts

Alerts are configured with the `alert=` configuration option in `kismet_alerts.conf`.  Alerts are configured by name, with `throttle` and `burst` options.

The throttle option controls how many alerts are allowed per time unit (seconds and minutes), and burst controls how many alerts are allowed in quick succession.  For instance:

```
alert=NETSTUMBLER,5/min,1/sec
```

Will allow at most one alert per second, and at most 5 alerts per minute.

### Kismet Alerts

* `ADVCRYPTCHANGE` \
	Trend/Stateful 

	An SSID has changed the encryption options it advertises (not necessarily decreasing the encryption options, but advertising different options).  This can occur when a managed access point is reconfigured, but may also indicate an access point spoofing attack.

* `AIRJACKSSID` \
	Fingerprint, deprecated

	One of the original 802.11 hacking tools, Airjack, set the initial SSID to `airjack` when loading.  This alert is no longer relevant since the Airjack toolset has long been discontinued.

* `APSPOOF` \
	Fingerprint

	A list of valid MAC addresses for a given SSID can be configured via the `apspoof=` configuration option.  If a beacon or probe response for that SSID is seen from a MAC address not found in that list, an alert will be raised.  This can be used to detect spoofed or evil twin attacks and attacks like Karma, however it will not detect attacks which also spoof the MAC address.

	The `apspoof=` configuration can specify exact string matches for the SSID, regular expressions using PCRE syntax, and single, multiple, or masked MAC addresses:
	```
	apspoof=Foo1:ssidregex="(?:foobar)",validmacs=00:11:22:33:44:55
	apspoof=Foo2:ssid="Foobar",validmacs="00:11:22:33:44:55,AA:BB:CC:DD:EE:FF"
	```

	When providing multiple MAC addresses, they must be enclosed in quotes.

* `BEACONRATE` \
	Trend/Stateful

	The advertised beacon rate of a SSID has changed; in a managed enterprise environment this may indicate a normal configuration change, however it may also indicate a spoofing attack against an access point.

* `BCOM11KCHAN` \
    Fingerprint

    Invalid channels in 802.11k neighbor report frames can be used to exploit certain Broadcom HardMAC implementations, typically used in mobile devices, as described in [project zero 1289](https://bugs.chromium.org/p/project-zero/issues/detail?id=1289)

* `BSSTIMESTAMP` \
	Trend/Stateful

	Invalid or out-of-sequence BSS timestamps can indicate AP spoofing.  APs with fluctuating BSS timestamps could be an indication of spoofing or an "evil twin" attack.  Out-of-order packets from multiple datasources and some AP firmware can both lead to false positives.

* `CHANCHANGE` \
	Trend/Stateful

	A previously detected access point which changes channels may indicate a spoofing attack or simply a configuration change.  By spoofing an otherwise legitimate AP on a different channel, an attacker may try to lure clients to the spoofed AP.  Centrally managed enterprise access points may change channel if they reboot or if the current channels become congested, and some consumer APs may automatically change channe to a less-used part of the spectrum.

    See `PROBECHAN` for related channel state alerts.

* `CRYPTODROP` \
	Trend/Stateful

	Typically an acess point should not downgrade to a less effective security setting.   This may be an attempt to spoof the access point or a misconfigured access point.

* `DEAUTHFLOOD` and `BCASTDISCON` \
	Trend/Stateful

	By spoofing disassociate and deauthenticate packets, an attacker may disconnect clients from a network which does not support management frame protection (MFP); This can be used to cause a denial of service or to disconnect clients in an attempt to capture handshakes for attacking WPA.

* `DHCPCLIENTID` \
	Fingerprint

	A client which sends a DHCP DISCOVER packet containing a CLient-ID tag (Tag 61) which doesn't match the source MAC of the packet may be doing a DHCP denial-of-service attack to attempt to exhaust the DHCP pool.  To detect the DHCP client ID, the network must be unencrypted so that data may be observed.

* `DHCPCONFLICT` \
	Trend/Stateful

	Clients which request a DHCP address and receive a response, but operate on a different IP address, may be misconfigured, or may be spoofed by an attacker trying to gain access by simulating an existing client.

* `DISASSOCTRAFFIC` \
	Trend/Stateful

	A client which has been disassociated from a network legitimately should not immediately continue exchanging data, as it needs to reestablish the connection to the AP.  A client behaving oddly can indicate a spoofed client attempting to incorrectly inject data into a network, or a client which is the victim of a denial-of-service attack.

* `DISCONCODEINVALID` \
	Fingerprint

	The 802.11 specification defines valid reason codes for disconnect and deauthenticate events.  Various clients and access points have been reported to improerly handle invalid/undefined reason codes, and illegal reason codes are often an indicator of a spoofing attack.

* `DHCPNAMECHANGE` and `DNSOSCHANGE` \
	Trend/Stateful

	The DHCP configuration protocol allows clients to optionally include the desired hostname and the DHCP client vendor/operating system in the DHCP Discovery packet.  These values should only change if the client has changed drastically (such as a dual-boot system).  As these are typically rare, these values changing can indicate a likely client spoofing attack.

* `DOT11D` \
	Trend/Stateful

	While deprecated, 802.11d country codes are still accepted by many clients.  Conflicting 802.11d advertisements can indicate an AP spoofing attack or an attempted to cause a client denial of service.

* `KARMAOUI` \
	Fingerprint

	Some implementations of Karma set a predictable OUI of 00:13:37.

* `LONGSSID` \
	Fingerprint

	The 802.11 specification allows for a maximum of 32 bytes for the SSID.  Over-sized SSIDs are indicative of packet corruption or of an attack attempting a buffer overflow on the access point or client drivers.

* `LUCENTTEST` \
	Fingerprint, deprecated

	Very old 802.11b Lucent Orinico cards, in certain scanning test modes, generated indentifiable packets.  These cards are extremely outdated at this point.

* `MSFBCOMSSID` \
	Fingerprint, deprecated

	Very old versions of the Windows Broadcom wireless drivers did not properly handle SSID fields longer than 32 bytes, which could lead to code execution.  This vulnerability is implemented in the Metasploit framework, but these drivers are extremely old and were found on Windows XP systems.

* `MSFDLINKRATE` \
	Fingerprint, deprecated

	Very old versions of the Windows D-Link wireless drivers did not properly handle extremely long 802.11 rate fields, leading to code execution.  This vulnerability is implemented in the Metasploit framework, but these drivers are extremely old and were found on Windows XP systems.

* `MSFNETGEARBEACON` \
	Fingerprint, deprecated

	Very old versions of the Windows D-Link wireless drivers did not properly handle over-sized beacon frames, leading to code execution.  This vulnerability is implemented in the Metasploit framework, but these drivers are extremely old and were found on Windows XP systems.

* `NETSTUMBLER` \
	Fingerprint, deprecated

	Very old versions of Netstumbler (3.22, 3.23, 3.30) generate, in certain conditions, specific packets which are identifiable.

* `NONCEDEGRADE` \
    Trend/stateful

    A WPA handshake has attempted to re-use a previous nonce value; this may indicate an attack against the WPA keystream such as the [vanhoefm KRACK attack](https://www.krackattacks.com/).  This alert is disabled by default by it is difficult to distinguish replay attacks vs legitimate retransmission of data in a passive observer.


* `NULLPROBERESP` \
	Fingerprint, deprecated

	Probe response packets with a SSID response with a length of 0 could cause extremely old 802.11b cards (prism2, orinoco, and the original Apple Airport) to crash.

* `OVERPOWERED` \
	Fingerprint

	Signal levels are abnormally high.  This can be caused by using an external amplifier with too much power, or being too close to a network.  Over-amplified signals can be distorted, resulting in fewer overall packets; in the absolute worst case, using too strong an amplifier can damage your Wi-Fi cards.

* `PROBECHAN` \
    Trend/stateful

    Wi-Fi Access Points advertise a channel in the beacon packet; they may also include a channel in the probe response.  The probe response should match the advertised beacon channel; if it does not, this can indicate a spoofing or 'evil twin' attack, but may also indicate a misconfigured or misbehaving Access Point or repeater.

* `PROBENOJOIN` \
	Trend/stateful

	Active scanning tools constantly send network discovery probes but never join any of the networks which respond.  This alert can cause excessive false positives in busy environments, so is disabled by default.

* `RSNLOOP` \
    Fingerprint

    Invalid RSN (802.11i) tags in beacon frames can be used to cause loops in some Atheros drivers, as described in CVE-2017-9714 and [pleasestopnamingvulnerabilities.com](https://pleasestopnamingvulnerabilities.com/)

* `WMMOVERFLOW` \
	Fingerprint

	The Wi-Fi standard specifies 24 bytes for WMM IE tags.  Over-sized WMM fields may be an attempt to exploit bugs in Broadcom chipsets, such as the Broadpwn attack.

* `WPSBRUTE` \
	Trend/stateful

	Excessive WPS negotiations can indicate an attack against WPS, such as Reaver

* `WMMTSPEC` \
    Fingerprint

    Too many WMMTSPEC options were seen in a probe response; this may be triggered by CVE-2017-11013 as described at [pleasestopnamingvulnerabilities.com](https://pleasestopnamingvulnerabilities.com/)


