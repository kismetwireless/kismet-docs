---
title: "Kismet External HTTP Helper Plugins"
permalink: /docs/devel/dev/plugin_httphelper/
toc: true
docgroup: "devel"
excerpt: "Extending Kismet via external helpers in Python and other languages"
---

## External Kismet helpers
Kismet implements a protobuf-based IPC system for communicating with external tools.  This system is used to manage datasources and capture data, but it can also be used to extend the Kismet functionality without writing C++ code loaded directly into Kismet.

### What helper plugins can do
Helper plugins are meant to provide additional functionality exposed via the Kismet web endpoints.  Generally they are useful for managing long-running external processes which operate on Kismet data; for example, running analysis on a pcap log or providing mapping from the Kismet UI to another tool.

Helper plugins can provide GET and POST endpoints, streaming data over an endpoint, and can also provide Javascript modules and HTTP to integrate with the web UI.  Helper plugins can automatically request a HTTP authentication key for accessing the Kismet REST endpoints.

### What helper plugins cannot do
Helper plugins do NOT run as part of Kismet - they are a separate process and communicate over IPC.  They can not directly implement internal parts of Kismet; they are limited to interacting with Kismet over the existing REST endpoints.

## Plugin manifest
The plugin manifest tells Kismet how to load the plugin.   Described more fully in [general plugin documentation](/docs/devel/plugins/), all plugins should define a name, author, description, and version:

```
name=externaldemo
author=Foo Bar <foo@bar.foo>
description=External API demo plugin
version=2019-00-00
```

To load an external API plugin, you also need the `httpexternal` option:

```
name=externaldemo
author=Foo Bar <foo@bar.foo>
description=External API demo plugin
version=2019-00-00
httpexternal=kismet_externaldemo
```

This tells Kismet to run `kismet_externaldemo` as an IPC command.

External API plugins can be combined with Javascript modules, as well; you can [make a Javascript module](/docs/devel/webui_basics/) and reference it in your manifest.  This lets you make a custom UI that interacts with your endpoints.

```
name=externaldemo
author=Foo Bar <foo@bar.foo>
description=External API demo plugin
version=2019-00-00
httpexternal=kismet_externaldemo
js=demo_web_module,/plugin/externaldemo/js/demo_web_module.js
```

## Helper executable
The external helper tool is run by Kismet and communicates over the [helper RPC interface](/docs/devel/exteral_helper_tools/).  The helper tool can be in any language which can talk the IPC protocol over a pipe(2) channel; for our examples we'll use Python and the KismetExternal module.

### Installing KismetExternal
KismetExternal has its own repository.  It will be migrated to a proper external module, installable via `pip`, but for now this guide will assume you're installing it via `setup.py`:

```bash
$ git clone https://www.kismetwireless.net/git/python-kismet-db.git
$ python-kismet-db
$ python2 ./setup.py build
$ sudo python2 ./setup.py install
```

### Python2 vs Python3
Currently protobufs appear to depend on python2; once there is a clean mechanism for running under python3 the KismetExternal module will be upgraded accordingly.

### Importing the basics
We need to import a few basic elements into our helper.  We also try to present a more meaningful message to the user if KismetExternal isn't available.

```python
#!/usr/bin/env python2

import argparse
import os
import sys
import threading
import time

try:
    import KismetExternal
except ImportError:
    print("ERROR:  We require the KismetExternal module")
    sys.exit(1)
```

### Parsing the arguments
Kismet will pass us two arguments when we get launched; `--in-fd` and `--out-fd`; these are the endpoints of the pipe(2) IPC channel.  We use the `argparse` module here, but you can use whatever you like to read the arguments.
  
```python
class ExternalDemo(object):
    def __init__(self):
        self.parser = argparse.ArgumentParser(description='Kismet external helper demo')
        self.parser.add_argument('--in-fd', action="store", type=int, dest="infd")
        self.parser.add_argument('--out-fd', action="store", type=int, dest="outfd")

        self.results = self.parser.parse_args()

        if self.results.infd is None or self.results.outfd is None:
            print("ERROR:  Kismet external helper tools are launched by Kismet itself; do not run them directly.")
            sys.exit(1)
```

### Initializing the external interface
KismetExternal provides a full threaded interface for handling the RPC communications. Once we initialize it, we can ask it for tokens and register callbacks.

```python
    self.kei = KismetExternal.ExternalInterface(self.results.infd, seld.results.outfd)
```

### Getting an authentication token
Talking to the Kismet REST endpoints requires an authentication token; rather than expose the username and password, or require each external tool to be configured with a login, the external tool can request a HTTP authentication token.  This can be used as the `KISMET` session cookie in all HTTP requests to the Kismet server.

Because RPC is asynchronous, we need to provide a callback.

```python
    self.kei.request_http_auth(self.handle_web_auth_token)
```

and then the callback:

```python
    def handle_web_auth_token(self):
        self.auth_token = self.kei.auth_token
```

A more complete callback might set a variable that initialization is complete, or start a thread that communicates with the Kismet server.

### Providing a REST endpoint
Now for the most powerful part of the external API:  providing our own REST endpoint for communicating with the world.

Rest endpoints can process `GET` and `POST` methods, and like the auth method, call a callback function we provide.

```python
    self.kei.add_uri_handler("GET", "/externaldemo/python.html", self.handle_web_python)
```

Then for the callback itself, we return some basic HTML:

```python
    def handle_web_python(self, externalhandler, request):
        # Print some debug that will show up in the console Kismet is running in
        print("PYTHON - handle web req {} {} {}".format(request.req_id, request.method, request.uri))
        # Return some HTML
        externalhandler.send_http_response(request.req_id, "<html><body><b>This is from python!</b></body></html>")
```

### Providing a POST endpoint
To receive data on our endpoint, we support a POST method; this is done with the same callback methods, but we use the `variable_Data` field from the requests:

```python
    self.kei.add_uri_handler("POST", "/externaldemo/post", self.handle_web_post)
```

and the callback:

```python
    def self.handle_web_post(self, externalhandler, request):
        # Iterate the variables
        for i in request.variable_data:
            print("PYTHON - POST {}:{}".format(i.field, i.content))

        externalhandler.set_http_response(request.req_id, "<html><body><b>Python got a POST</b></body></html>")
```
   
### Sending status to the Kismet server
We can send arbitrary messages to the Kismet server via `send_message`:

```python
    self.kei.send_message("Status update from python!")
```

### The rest
There's a little more we need to do; we need to start the KismetExternal interface, and instantiate our object.  We also need to define a service loop that runs for as long as the KismetExternal interface remains up.

Our extremely basic, but complete, example would look like:

```python
#!/usr/bin/env python2

import argparse
import os
import sys
import threading
import time

try:
    import KismetExternal
except ImportError:
    print("ERROR:  We require the KismetExternal module")
    sys.exit(1)

class ExternalDemo(object):
    def __init__(self):
        self.parser = argparse.ArgumentParser(description='Kismet external helper demo')
        self.parser.add_argument('--in-fd', action="store", type=int, dest="infd")
        self.parser.add_argument('--out-fd', action="store", type=int, dest="outfd")

        self.results = self.parser.parse_args()

        if self.results.infd is None or self.results.outfd is None:
            print("ERROR:  Kismet external helper tools are launched by Kismet itself; do not run them directly.")
            sys.exit(1)

        self.kei.request_http_auth(self.handle_web_auth_token)
        self.kei.add_uri_handler("GET", "/externaldemo/python.html", self.handle_web_python)

        self.kei.send_message("Status update from python!")

        self.kei.start()

    def handle_web_auth_token(self):
        self.auth_token = self.kei.auth_token

    def handle_web_python(self, externalhandler, request):
        # Print some debug that will show up in the console Kismet is running in
        print("PYTHON - handle web req {} {} {}".format(request.req_id, request.method, request.uri))
        # Return some HTML
        externalhandler.send_http_response(request.req_id, "<html><body><b>This is from python!</b></body></html>")

    def handle_web_python(self, externalhandler, request):
        # Print some debug that will show up in the console Kismet is running in
        print("PYTHON - handle web req {} {} {}".format(request.req_id, request.method, request.uri))
        # Return some HTML
        externalhandler.send_http_response(request.req_id, "<html><body><b>This is from python!</b></body></html>")

    # Loop forever
    def loop(self):
        while self.kei.is_running:
            self.kei.send_ping()
            time.sleep(1)

        self.kei.kill()

if __name__ == "__main__":
    # Make a proxytest and loop forever
    pt = ExternalDemo()

    # Loop in a detached process
    pt.loop()
```

## More examples
You can find a similar implementation of this in the `plugin-httpproxytest/` directory in the Kismet git; this is a Python KismetExternal plugin which implements GET, POST, and streaming interfaces.


