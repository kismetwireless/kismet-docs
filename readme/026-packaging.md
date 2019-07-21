---
title: "Packaging"
permalink: /docs/readme/packaging/
excerpt: "Recommendations for package maintainers"
docgroup: "readme"
toc: true
---

## Packaging Kismet

Every distribution is different, and has different idioms for configuration location and style, but we recommend the following:

### Provide a suid (or setcap) install option

The Kismet capture binaries (`kismet_cap_linux_wifi`, `kismet_cap_linux_bluetooth`, and so on) require root privileges to control the interfaces, however generally Kismet itself should not be run as root.  Kismet supports two mechanisms for this:

1. Installing the binaries setuid root, and restricting access to them by limiting them to a specific group (`kismet`, by preference).
2. Installing the binaries normally, but using `setcap` to give them authority to change network settings: `setcap cap_net_raw,cap_net_admin=eip`, and restricting them to a single group (`kismet` again, by preference).

When installed suid-root, the Kismet helper utilities will drop capabilities and pivot into a read-only mount namespace.

### Provide multiple packages for capture tools

Kismet capture tools can be packaged independently.  By preference, the capture tools should be independent packages so that a user building a remote capture system only needs the specific capture packages.  A meta-package which references all of the capture tools can provide a simple installation point for users.

### Default prefix

The Kismet configure defaults to `/usr/local/`; typically a distribution package would target `/usr` as the prefix.

### Config files

Kismet looks for config files in the directory specified in `configure` via the `--sysconfdir` directive.  This location may vary per distribution, however typically `/etc/kismet` would be the appropriate default, and is the location used by the packages provided on the Kismet site.

### Never override `kismet_site.conf`

Kismet provides an override mechanism for local configurations which take precedence over all other config options; these are kept in `kismet_site.conf` by default.  A package should never provide this file, as changes made by the user will likely go here.

### Protobuf-lite

Some distributions (like OpenWRT) use libprotobuf-lite to save space.  Kismet does not use any of the options removed in the lite version, but *should* be configured with the `--enable-protobufite` option to `./configure`.

