---
title: "Remote capture"
permalink: /docs/readme/datasources_remote_capture/
excerpt: "Remote network capture allows Kismet to receive packets from distributed sensors installed on other hardware, such as OpenWRT routers."
docgroup: "readme"
toc: true
---

## Remote Packet Capture

Kismet can capture from a remote source over a TCP connection (legacy) or over websockets (2020-10 and newer).

## Websockets

Kismet has previously provided a remote packet capture API over TCP sockets on port 3501.  

As of 2020-10, Kismet now supports websockets and using a websocket endpoint in the standard Kismet webserver on port 2501.  

To use websockets for remote capture, you need to be sure the capture tools are compiled with libwebsockets support.  Some older distributions have not updated libwebsockets and are too old for Kismet to use; remote capture is still possible with these distributions using the TCP mode (TCP mode does not support authentication or proxy, but can be protected via SSH tunnels or similar, more info below).

Websockets add:

* Single port

    Websockets are endpoints on the standard Kismet webserver.  There is no need to forward additional ports for websockets mode.

* Authentication

    Websockets remote capture is behind either the Kismet username and password, or an API token with the 'datasource' role.

* Proxyable

    Websockets can be proxied via standard web proxies, such as nginx.

* Encryptable

    When proxied by nginx, websocket remote capture can be wrapped in SSL.

Generally all new deploys of remote capture should use the websockets based system.

## Launching remote capture

Kismet remote capture is initiated by the same binaries Kismet uses for capturing locally and offers the same control and features as local capture.

When launching remote capture, you provide the host and port of the Kismet server, the username and password of your Kismet user, and a local source definition:

```bash
# kismet_cap_linux_wifi --connect 192.168.1.2:2501 --user kismetuser --password kismetpass --source=wlan1:name=remote-wlan1
```

Specifically, this uses the `kismet_cap_linux_wifi` tool, which is by default installed in `/usr/local/bin/`, to connect to the IP `192.168.1.2` port 2501.

The --source=... parameter is the same as you would use in a `source=` Kismet configuration file entry, or as `-c` to Kismet itself. 

Source definitions of a remote capture are controlled the same way as `source=` definitions in the Kismet config, and can take the same options.  It's not uncommon to have many remote captures using `wlan0` as the capture interface, but by passing the `name=foo` option to the source, it's easy to differentiate:

```bash
# kismet_cap_linux_wifi --connect 192.168.1.2:2501 --user kismetuser --password kismtepass --source=wlan1:name=sensor_foo_wlan1,add_channels=\"6W5,36W5\",info_antenna_type=omni
```

### Any-to-Any

Kismet is designed to allow any source to function as a remote capture, and to function cross-platform.  It is completely reasonable, for instance, to use an OpenWRT sensor running Linux to provide packets to a Kismet server running on OSX or Windows 10 under the WSL.

### Controlling access to remote capture (websockets)

Websocket-based remote capture can be authenticated by the Kismet user and password, or can be authenticated with a provisioned API token with the role 'datasource'.

Publicly accessible Kismet servers should always be protected by SSL.

To connect Kismet to a public nginx proxy, consider using a tool like `autossh` (shown below), and a configuration in nginx like the following:

```
location /kismet/ {
        proxy_pass http://localhost:2501;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Proxy-Dir /kismet;
        proxy_http_version 1.1;
        client_max_body_size 0;
}
```

When proxying, there are two options - proxying via a specific directory, or proxying an entire domain name.   When proxying by a single directory, you will need to:

1.  Tell Kismet to handle the directory in the path
    
    In `kismet_httpd.conf` (or preferably, `kismet_site.conf`), set the `httpd_uri_prefix` option.  For the example above, `httpd_uri_prefix=/kismet`

2.  Provide the altered URI and port to the remote capture tool

    Use the `--endpoint` argument to set the appropriate endpoint, and use the standard https port in the `--connect` host.

    `kismet_cap_linux_wifi --connect some-public-host:443 --user kismet --password password --endpoint /kismet/datasource/remote/remotesource.ws --source wlan0:name=websocket --ssl`

### Controlling access to remote capture (legacy TCP)

For security reasons, by default Kismet only accepts remote sensor connections from the localhost IP (`127.0.0.1`).  To connect remote sensors, you must either:

1. Set up a tunnel from the remote sensor to your Kismet server, for example using SSH port forwarding.  This is very simple to do, and adds encryption transparently to the remote packet stream.  This can be done as simply as:

   ```bash
   # ssh someuser@192.168.1.2 -L 3501:localhost:3501
   ```

   This sets up a SSH tunnel from `localhost` port 3501 to `192.168.1.2` port 3501.  Then in a second terminal running the Kismet remote capture, using `localost:3501` as the destination:

   ```
   # /usr/local/bin/kismet_cap_linux_wifi --connect localhost:3501 --source=wlan1
   ```

   Other, more elegant solutions exist for building the SSH tunnel, such as `autossh` which can be used to automatically maintain the tunnel and start it on boot.

2.  Kismet can be configured to accept connections on a specific interface, or from all IP addresses, by changing the `remote_capture_listen=` line in `kismet.conf` or `kismet_site.conf` as an override.  To enable listening on ALL network interfaces:

   ```
   remote_capture_listen=0.0.0.0
   ```

   Or a single specific network interface:
   ```
   remote_capture_listen=192.168.1.2
   ```

   Remote capture *should only be enabled on interfaces on a protected LAN*.

### Remote capture devices must be unique

Every datasource in Kismet must have a unique identifier, the source UUID.  Kismet calculates this using the MAC address (when available) of the capture device, or on some source types, the serial number or other identifying information.

Not all devices provide sufficient unique identifiable information; for instance the rtlsdr hardware often reports a serial number of "00000000".  This is not a problem when using a single device or capturing from a single remote capture, however if multiple remote capture devices advertise the same identity, problems occur.

This can be solved with some devices by using a device-specific tool; for rtlsdr, the device serial number can be set using the `rtlsdr_eeprom` tool.  Remember - the serial number should be unique across *all* devices connecting to a Kismet server, not just on a single capture platform!

Another option is to use the `uuid=` parameter in the source definition to set a unique, static UUID for each remote capture source.  A UUID can be generated via the `genuuid` tool on most platforms, and the only requirement is that it fits the standard UUID format and is unique.

### Additional remote capture options

Kismet capture tools also support the following options:

* `--connect=[host]:[port]`

    Connects to a remote Kismet server on [host] and port [port].  When using `--connect=...` you MUST specify a `--source=...` option.

* `--tcp`

    Use the older style TCP socket instead of the new websockets method

* `--user=[username]`

    Set the Kismet username, required for websockets mode

* `--password=[password]`

    Set the Kismet password, required for websockets mode

* `--apikey=[key]`

    Use an API key instead of username and password authentication in websockets mode

* `--ssl=[ca certificate]`

    use SSL in websockets mode.  Optionally provide a SSL CA certificate to authenticate the remote server, if the remote server does not use a publicly valid certificate.

* `--endpoint=[endpoint]`

    Use an alternate endpoint in websockets mode.  This should only be necessary when using a HTTP/HTTPS proxy and placing Kismet in a proxied directory.

* `--source=[source definition]`

    Define a source; this is used only in remote connection mode.  The source definition is the same as defining a local source for Kismet via `-c` or the `source=` config file option.

* `--disable-retry`

    By default, a remote source will attempt to reconnect if the connection to the Kismet server is lost.

* `--daemonize`

    Places the capture tool in the background and daemonizes it.

* `--fixed-gps [lat,lon,alt] or [lat,lon]`

    Set the GPS location of the remote capture source; this will tag any packets from this source with a static, fixed GPS location.

* `--gps-name [name]`

    Rename the virtual GPS reported on this source; otherwise the capture name "fixed-remote" is used.

### Remote capture and GPS

Remote capture is typically incompatible with a *moving* sensor, because of the bandwidth required to move a full packet stream to the Kismet server, however in some situations it may be possible or desired.  There are two options for GPS with remote capture:

1. Fixed GPS

    For a remote capture in a fixed location, using the `--fixed-gps` argument when setting up the remote capture will tag all packets with that location information.  This lets Kismet use them location averaging, device seen-by locations, etc.

2. Meta GPS

    *Added in Kismet 2022-01-R3*

    For a more dynamic remote gps, use the `metagps` option on the datasource definition:

    ```bash
    $ kismet_cap_linux_wifi --connect *foo* --source wlan0:name=remote1,metagps=remote1
    ```

    This creates a virtual GPS which can be updated [via the REST API](/docs/devel/webui_rest/gps/#meta-gps) to provide locational data for the remote source.

### Filtering remote-capture Wi-Fi

Remote capture is designed to be as lightweight as possible and does not process packet contents, however it can still support BPF filtering.  When doing a remote Wi-Fi capture on a dual-radio capture platform connected over Wi-Fi itself, the remote capture stream will often see itself transmitting packets to the Kismet server, leading to a huge bandwidth spike and loop.

By specifying the `filter_locals=true` source option, `kismet_cap_linux_wifi` will automatically build a BPF filter to exclude all local interfaces found on the system:

```bash
# kismet_cap_linux_wifi --connect some-kismet-server:3501 --source wlan0:name=some_remote_cap,filter_locals=true
```

Because the Linux kernel has a limited amount of space for BPF filter programs, only the first *eight* interfaces found on the system are included in the BPF filter, currently.

### Remote capture timestamps

By default, Kismet will override the packet timestamp from a remote capture with the timestamp of *when the packet is seen by the Kismet server*.  Otherwise, variance in the clocks of remote captures can cause problems with Kismet - such as invalid WIDS alerts - due to some packets arriving "in the past".

If your clocks are synced with NTP, or if you want to preserve the timestamp for some other reason, you can use the source option `timestamp=false`:

```bash
# kismet_cap_linux_wifi --connect some-kismet-server:3501 --source wlan0:name=some_remote_cap,timestamp=false
```

### Compiling Only the Capture Tools

Typically, you will want to compile all of Kismet.  If you're doing specific remote-capture installs, however, you may wish to compile only the binaries used by Kismet to enable capture mode and pass packets to a full Kismet server.

To compile ONLY the capture tools:

Tell configure to only configure the capture tools; this will allow you to configure without installing the server dependencies such as C++, microhttpd, protobuf-C++, etc.  You will still need to install dependencies such as protobuf-c, build-essentials, and similar:
```bash
$ ./configure --enable-capture-tools-only
```

Once configure has completed, compile the capture tools:
```bash
$ make datasources
```

You can now copy the compiled datasources to your target.

