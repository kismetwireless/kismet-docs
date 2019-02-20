---
title: "Kismetdb to PCAP"
permalink: /docs/readme/kismetdb_to_pcap/
excerpt: "Converting Kismetdb to PCAP"
toc: true
---

## Packet data
Kismet stores packets as binary data in the [kismetdb log file](/docs/readme/logging).

Most tools like [Wireshark](https://www.wireshark.org), [tcpdump](https://www.tcpdump.org), [Aircrack-NG](https://www.aircrack-ng.org), and many more, use the PCAP format.

## Installing the Python log utility
The kismetdb-to-pcap conversion utility is part of the `python-kismet-db` package.  This can be installed automatically via `pip`:

```bash
$ pip install kismetdb
```

If pip is unavailable, you will need to install the pip package for your distribution.

Alternately, you can install the kismetdb Python module from git:

```bash
$ git clone https://www.kismetwireless.net/git/python-kismet-db.git
$ cd python-kismet-db
$ python ./setup.py build
$ sudo python ./setup.py install
```

You will need the python-setuptools package from your distribution.

## Converting packets
The simplest way to convert packets is to just call the `kismet_log_to_pcap` tool:

```bash
$ kismet_log_to_pcap --in some-kismet-log.kismet --out some-pcap-log.pcap
```

However, there are many options for filtering the packet stream as it is converted:

* `--outtitle [*filetitle*]` and `--limit-packets [*number*]`
    Instead of outputting to a single file, you can automatically split the packets into multiple files.  This can make processing the log easier with some tools.

    ```bash
    $ kismet_log_to_pcap --in some-kismet-log.kismet --outtitle some-log --limit-packets 1000
    ```

* `--source-uuid [*uuid*]`
    Limit packets by the capture UUID of the Kismet source that recieved the packet.

* `--start-time [*time*]` and `--end-time [*time*]`
    Limit packets by the supplied start and end time ranges.  Times can be human-readable like the format of the `date` command.

* `--min-signal [*value*]`
    Only include packets with a signal stronger than `[*value*]`.

