---
title: "Webserver"
permalink: /docs/readme/webserver/
excerpt: "Kismet webserver configuration"
toc: true
---

## Kismet Webserver
Kismet now integrates a webserver which serves the web-based UI and data to external clients.

**THE FIRST TIME YOU RUN KISMET**, you must go to the Kismet web UI and create a login and password.  This password is stored in `~/.kismet/kismet_httpd.conf` which is in the home directory of **the user which started Kismet**.

You will need this password to log into Kismet in the future, and to give scripts and tools which need administrative access to Kismet the login.

The webserver is configured via the `kismet_httpd.conf` file.  These options may be included in the base kismet.conf file, but are broken out for clarity.  These options may be overridden in `kismet_site.conf` for pre-configured installs.

By default, Kismet does not run in SSL mode.  If you provide a certificate and key file in PEM format, Kismet supports standard SSL / HTTPS; alternately, Kismet can be run behind a SSL-enabling proxy.

HTTP configuration options:

* `httpd_username=username`
   Set a global, fixed username.  This overrides any per-user login information in `~/.kismet/kismet_httpd.conf`.  This can be used to set a fixed password during deployment, for instance via `kismet_site.conf` overriding.  If `httpd_username=` is specified, `httpd_password=` *must* also be provided.

* `httpd_password=password`
   Set a global, fixed password.  This overrides any per-user login information in `~/.kismet/kismet_httpd.conf`.  This can be used to set a fixed password during deployment, for instance via `kismet_site.conf` overriding.
   It is generally preferred to keep the username and password in the per-user configuration file, however they may also be set in the global config.

* `httpd_port=port`
   Sets the port for the webserver to listen to.  By default, this is port 2501, the port traditionally used by the Kismet client/server protocol.
   Kismet typically should not be started as root, so will not be able to bind to ports below 1024.  If you want to run Kismet on, for instance, port 80, this can be done with a proxy or a redirector, or via DNAT rewriting on the host.

* `httpd_uri_prefix=/prefix`
   Sets the URI prefix - this prefix is an optional lead-in value to all the existing URIs in Kismet which allows it to be run via a HTTP proxy such as nginx.  For example, setting a httpd_uri_prefix of `/kismet/` would allow proxying from the `/kismet/` directory of a nginx server.

* `httpd_ssl=true|false`
   Turn on SSL.  If this is turned on, you must provide a SSL certificate and key in PEM format with the `httpd_ssl_cert=` and `httpd_ssl_key=` configuration options.

* `httpd_ssl_cert=/path/to/cert.pem`
   Path to a PEM-format SSL certificate.

   This option is ignored if Kismet is not running in SSL mode.

   Logformat escapes can be used in this.  Specifically, "%S" will automatically expand to the system install data directory, and "%h" will expand to the home directory of the user running Kismet:
   ```
   httpd_ssl_cert=%h/.kismet/kismet.pem
   ```

* `httpd_ssl_key=/path/to/key.pem`
   Path to a PEM-format SSL key file.  This file should not have a password set as currently Kismet does not have a password prompt system.

   This option is ignored if Kismet is not running in SSL mode.

   Logformat escapes can be used in this.  Specifically, "%S" will automatically expand to the system install data directory, and "%h" will expand to the home directory of the user running Kismet:
   ```
   httpd_ssl_key=%h/.kismet/kismet.key
   ```

* `httpd_home=/path/to/httpd/data`
   Path to static content web data to be served by Kismet.  This is typically set automatically to the directory installed by Kismet in the installation prefix.
   Typically the only reason to change this directory is to replace the Kismet web UI with alternate code.

* `httpd_user_home=/path/to/user/httpd/data`
   Path to static content stored in the home directory of the user running Kismet.  This is typically set to the httpd directory inside the users .kismet directory.

   This allows plugins installed to the user directory to install web UI components.

   Typically there is no reason to change this directory.

   If you wish to disable serving content from the user directory entirely, comment this configuration option out.

* `httpd_session_db=/path/to/session/db`
   Path to save HTTP sessions to.  This allows Kismet to remember valid browser login sessions over restarts of kismet_server. 

   If you want to refresh the logins (and require browsers to log in again after each restart), comment this option.

   Typically there is no reason to change this option.
   
* `httpd_mime=extension:mimetype`
   Kismet supports MIME types for most standard file formats, however if you are serving custom content with a MIME type not correctly set, additional MIME types can be defined here.
   Multiple httpd_mime lines may be used to add multiple mime types:
   ```
   httpd_mime=html:text/html
   httpd_mime=svg:image/svg+xml
   ```
   Typically, MIME types do not need to be added.

## Kismet and SSL

### When should I use SSL?
SSL provides encryption between the Kismet server and a web browser communicating with it.

SSL is not usually necessary when using Kismet locally (like your laptop or a dedicated device), however if you plan to use Kismet remotely over a public network, or any other sort of remote network, you should strongly consider enabling SSL.

Without SSL enabled, your data (and your Kismet Web UI login and password) will be sent in plaintext, which is likely not something you want.

### Local SSL vs Proxy
Kismet can be run via a HTTPS proxy service such as nginx; under this configuration, the SSL certificates are managed by the proxy software, and Kismet can continue to operate normally with a local port.

### Enabling SSL

SSL is enabled in the kismet_httpd.conf configuration file by setting

```
httpd_ssl=true
```

and providing a PEM-formatted SSL certificate and key via:

```
httpd_ssl_cert=/path/to/cert
httpd_ssl_key=/path/to/key
```

### Getting a certificate

Kismet can use either a legitimate (signed by a trusted authority) certificate, or you can provide a self-signed cert.

If using a real certificate, follow the instructions from your certificate provider for creating a key and certificate request, and submit it for signing.

If you wish to use a self-signed certificate, follow the instructions below.

WARNING:  Self-signed certificates provide the same encryption, but DO NOT certify that a host is legitimate.  Because a self-signed certificate is not automatically trusted by your browser, you WILL receive a SSL error when you visit the Kismet Web UI.  You MUST confirm that the certificate you are seeing is the certificate you expect.  If you do not verify the certificate yourself, you may be susceptible to man-in-the-middle interception.

*WARNING:  ANY THIRD PARTY CLIENT which communicates with a SSL-enabled Kismet server MUST PROVIDE A METHOD FOR THE USER TO VALIDATE AND PIN THE CERTIFICATE.  Accepting all SSL certificates is NOT valid.*

###  Using LetsEncrypt certificates

LetsEncrypt (https://letsencrypt.org) provides free, signed SSL certificates which are accepted by most browsers.  These certificates can be used with Kismet and may offer a simpler and more secure option than self-signed certificates.

LetsEncrypt currently uses a set of python (or third-party) clients to automatically generate certificates.  In order to verify that you control the domain the certificate is issued for, you must run these scripts on the webserver with that domain.

Once the certificates are generated, they must be provided to Kismet.  You must provide the key file and the full chain certificate (by default named "full-chain.cer").  These files are already in PEM format.

To use a letsencrypt certificate without errors, Kismet must be running on the server with that public domain name, or the connection to Kismet must be forwarded, such as port forwarding over SSH.

###  Generating a self-signed certificate

When using a self-signed certificate, keep in mind the warnings above in "Getting a certificate".

To generate a self-signed certificate on Linux using the OpenSSL command line tools:

1.  Generate a RSA key.  This is the private key used to encrypt your certificate.  You will need to generate a keyfile without an embedded password.

    ```bash
    $ openssl genrsa -out kismet.key 4096
    ```

    This will generate a 4096-bit RSA keyfile.

2.  Generate a CSR (certificate request).  This is the request which defines the public data in your certificate, and links it to the key file.
    ```bash
    $ openssl req -new -key kismet.key -out kismet.csr
    ```

    You will be prompted to enter information to put in the certificate (Country name, State, etc).  If you were generating a legitimate certificate, this information would matter, but on a self-signed cert, it is less relevant.  Enter information which is recognizable and makes sense to you.

    On the Common Name field, enter the public name of your server.

    When prompted to enter a challenge password, leave it blank.

3.  Create the self-signed certificate:
    ```bash
    $ openssl x509 -req -days 365 -in kismet.csr -signkey kismet.key -out kismet.pem
    ```

    This will generate a request for a certificate which is valid from the time you run the command until a year in the future.  To make a certificate valid for longer than a year, change the -days parameter.

4.  Verify your certificate is what you expected
    ```bash
    $ openssl x509 -in kismet.pem -noout -text
    ```

    This should print out information about your certificate, including the signature you will need to validate it in the future.

    Once you have generated your certificate, copy it and configure Kismet to use SSL and point at your certificate.  For example, if you copied your .pem and .key files into ~/.kismet/ssl/, you would configure Kismet as:

    ```
    httpd_ssl=true
    httpd_ssl_cert=%h/.kismet/ssl/kismet.pem
    httpd_ssl_key=%h/.kismet/ssl/kismet.key
    ```

### Converting certificates

Kismet requires that the certificate be in PEM format.  If you are using a certificate signed by a certificate authority, you may receive the certificate in DER format.

If you are unsure what format a certificate file is in, open it in a standard editor or use the 'cat' command.  A PEM format certificate will include '-----BEGIN CERTIFICATE----', while a DER format certificate will be pure binary.

To convert a DER to PEM certificate:
```bash
$ openssl x509 -inform der -in certificate.cer -out certificate.pem
```

### Intermediary certificates

Many certificate authorities use an intermediary signing certificate.

This certificate must be included in the chain presented by the server to be valid in all browsers (Typically, Chrome works without intermediary certificates, but command-line tools like 'curl' and other OpenSSL based code, such as embedded https clients in Python, Ruby, etc, does not.

If you are using an official certificate, and need to include the intermediary certificate from your provider, simply concatenate it with your server certificate:

```bash
$ cat kismet.pem intermediate.pem > combo.pem
```

And use the combined certificate in your kismet_httpd.conf:

```
httpd_ssl_cert=%h/.kismet/ssl/combo.pem
```

The order of certificate files is important:  Your server certificate must come before the intermediary certificate.

## Using proxies and forwarding

Kismet supports being proxied through a HTTP/HTTPS proxy such as nginx.  This allows using standard configuration of certs on the public server, instead of moving the configuration to Kismet.

1.  Set up Kismet to support a proxy URI.
    By setting the `httpd_uri_prefix` variable, Kismet can handle a proxied URI directory:
    ```
    httpd_uri_prefix=/kismet
    ```

2.  Set up the nginx proxy
    nginx proxy can connect HTTPS and HTTP endpoints.  In `/etc/nginx/sites-enabled/default`, a `location` stanza similar to:
    ```
    location /kismet/ {
        proxy_pass http://localhost:2501/;
    }
    ```

3.  Tunnel the Kismet server over SSH
    SSH supports port forwarding/tunneling; We can use this to bring a connection to the Kismet server to the proxy system:
    ```bash
    $ ssh -R *:2501:localhost:2501 user@host
    ```

