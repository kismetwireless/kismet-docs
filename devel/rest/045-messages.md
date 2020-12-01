---
title: "Messages"
permalink: /docs/devel/webui_rest/messages/
toc: true
docgroup: "devel-rest"
excerpt: "Kismet exposes the console messages via the messagebus API."
---

Kismet uses an internal `messagebus` system for communicating text messages from system components to the user.  The messagebus is used to pass error, state, and debug messages, as well as notifications to the user about detected devices, alerts, etc.

For real-time message data, see the [eventbus](/docs/devel/webui_rest/eventbus/).

## All messages

* URL

    /messagebus/all_messages.json

    /messagebus/all_messages.ekjson

    /messagebus/all_messages.itjson

* Methods

    `GET`

* Role

    `readonly`

* Result

    Array of the last 50 messages in the messagebus

## Recent messages

* URL 

    /messagebus/last-time/*[TIMESTAMP]*/messages.json

    /messagebus/last-time/*[TIMESTAMP]*/messages.ekjson

    /messagebus/last-time/*[TIMESTAMP]*/messages.itjson

* Methods

    `GET`

* Role

    `readonly`

* URL parameters

    | Key           | Description                                                                  |
    | ---           | -----------                                                                  |
    | *[TIMESTAMP]* | Relative or absolute [timestamp](/docs/devel/webui_rest/commands/#timestamp) |

* Result

    Array of messages since *TIMESTAMP*

