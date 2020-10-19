---
title: "Eventbus socket"
permalink: /docs/devel/webui_rest/eventbus/
toc: true
docgroup: "devel-rest"
excerpt: "Event bus socket"
---

## Eventbus

The `event bus` is a push/publish system inside Kismet where events are transmitted.  Events contain a `string:object` dictionary of values and content, and can contain simple data or complete Kismet objects.

The eventbus API is a websocket endpoint with a subscription model for attaching an UI to the eventbus.  Events are sent over the socket automatically from the Kismet server.

In the standard web UI, Kismet combines the poll APIs to populate the initial pre-existing data (such as alerts, message text, and so on), then subscribes to the eventbus socket to receive updates as push events.

Eventbus topics are case-sensitive, and for topics which contain Kismet objects, the returned JSON is formatted using Kismet field names and types and is identical to the records returned via polling APIs.

* URL

    /eventbus/events.ws

* API Added

    `2020-10`

* Methods

    `WEBSOCKET` (HTTP Upgrade + Websocket handshake)

* URI parameters

    | Key      | Description                                                  |
    | ---      | -----------                                                  |
    | user     | Kismet administrative username, as HTTP URI-encoded variable |
    | password | Kismet administrative password, as HTTP URI-encoded variable |
    | KISMET   | Kismet auth cookie, as HTTP URI-encoded variable             |

* Result

    A websocket session with a subscription-model API

* Notes

    Kismet websockets will accept authentication as HTTP basic auth headers, Kismet session token cookies, or HTTP URI-encoded GET parameters of the basic auth or session cookie.

    Some tools (websocat, others) allow sending HTTP basic auth as part of the URI (`ws://user:pass@host:port/eventbus/events.ws`).  However, modern browser implementations do *not* support this, and websockets must be constructed using GET parameters, such as `ws://host:port/eventbus/events.ws?user=username&password=userpass`.

    Websocket URIs *must* match the content type of the calling page; a `https` page must use the `wss` URI and a `http` page must use the `ws` URI.  The Kismet UI detects this via javascript.

## Eventbus subscription API

The eventbus subscription API accepts JSON data requesting a subscription topic and optional [field simplification](/docs/devel/webui_rest/commands/#field-specifications).

Currently consumers are limited to *one* subscription *per topic*; it is not possible for a single connection to subscribe to the same topic twice with different field selections.

An eventbus JSON command contains:

| Key         | Content                                                                                                   |
| ---         | -----------                                                                                               |
| SUBSCRIBE   | Topic to subscribe to, case sensitive                                                                     |
| UNSUBSCRIBE | Topic to unsubscribe from, case sensitive                                                                 |
| fields      | If subscribing, a standard [field simplification](/docs/devel/webui_rest/commands/#field-specificiations) |

Either a `SUBSCRIBE` or `UNSUBSCRIBE` must be provided with each command.

Once subscribed to a topic, an eventbus receiver will be sent a websocket text message containing the JSON record for each event which matches that subscription, optionally simplified.

For instance:

```bash
dragorn@lithium ~ % websocat 'ws://host:2501/eventbus/events.ws?user=username&password=password'
{"SUBSCRIBE": "TIMESTAMP"}
{"TIMESTAMP": {"kismet.system.timestamp.usec": 671986,"kismet.system.timestamp.sec": 1603120458}}
{"TIMESTAMP": {"kismet.system.timestamp.usec": 672945,"kismet.system.timestamp.sec": 1603120459}}
```

Or in Javascript:

```javascript
var ws = new Websocket('ws://host:2501/eventbus/events.ws?user=username&password=password');
ws.onmessage = function(msg) {
    var json = JSON.parse(msg.data);
    console.log(json);
}
ws.onopen = function(event) {
    var req = {
        "SUBSCRIBE": "TIMESTAMP"
    }
    ws.send(JSON.stringify(req));
}
```

## Eventbus topics

Eventbus topics may be dynamically expanded by plugins, external helper tools, and more.  Some topics are used predominately as internal messaging mechanisms between components of Kismet, while others are generated at regular intervals specifically for consumption by external UI and monitoring systems.

Event topics include:

### ALERT

* Content

    JSON object containing a Kismet alert record

* Generation

    Published whenever an alert is raised in the Kismet WIDS/Alert system

### BATTERY

* Content

    JSON object containing the battery presence, charge, and rate data, as per the system statistics API

* Generation

    Published once per second by the Kismet server

### DATASOURCE_PAUSED

* Content


* Generation

    Published when a running datasource is paused

### DATASOURCE_RESUMED

* Content

    JSON object containing a Kismet datasource record

* Generation

    Published when a paused datasource is resumed

### DATASOURCE_ERROR

* Content

    The UUID of a Kismet datasource

* Generation

    Published when a datasource experiences an error

### DATASOURCE_OPENED

* Content

    The UUID of a Kismet datasource

* Generation

    Published when a datasource is opened (or re-opened)

### DATASOURCE_CLOSED

* Content

    The UUID of a Kismet datasource

* Generation

    Published when a datasource is closed

### DOT11_WPA_HANDSHAKE

* Content

    Contains two keys, DOT11_WPA_HANDSHAKE_BASE and DOT11_WPA_HANDSHAKE_DOT11, which contain, respectively, the base Kismet device and the 802.11-specific device sub-record.

* Generation

    Published when a complete (or estimated to be complete) WPA handshake is captured

### GPS_LOCATION

* Content

    JSON object containing the current 'best' GPS location, as per the GPS location API

* Generation

    Published once per second by the Kismet server

### KISMETDB_LOG_OPEN

* Content

    None

* Generation

    Published when the kismetdb log is successfully opened

### MESSAGE

* Content

    JSON object containing a Kismet messagebus message (text, flags, and timestamp)

* Generation

    Published when a new message is generated in Kismet

### NEW_DATASOURCE

* Content

    JSON object containing a Kismet datasource record

* Generation

    Published when a new datasource is defined

### PACKETCHAIN_STATS

* Content

    JSON object containing the packet chain statistics RRDs, as per the packetchain stats API

* Generation

    Published once per second by the Kismet server

### TIMESTAMP

* Content

    JSON object containing the second and usecond timestamp

* Generation

    Published automatically once per second by the Kismet server

