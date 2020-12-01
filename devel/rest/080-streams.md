---
title: "Streams"
permalink: /docs/devel/webui_rest/streams/
toc: true
docgroup: "devel-rest"
excerpt: "Logging and long-running live exports of data are classified as streams and can be observed and manipulated via the stream API."
---
A Kismet stream is linked to an export of data of prolonged length; for instance, packet capture logs to disk or streamed over the web API.

Streams can be monitored and managed; a privileged user can close existing streams.

## Streams list

* URL

    /streams/all_streams.json

    /streams/all_streams.ekjson

    /streams/all_streams.itjson

* Methods

    `GET`

* Role

    `readonly`

* Result

    Array of active streams

## Stream details

* URL

    /streams/by-id/*[STREAMID]*/stream_info.json

* Methods 

    `GET`

* Role

    `readonly`

* URL parameters 

    | Key          | Description             |
    | ---          | -----------             |
    | *[STREAMID]* | ID of stream to examine |

* Results

    Returns detailed stream information object

## Closing a stream

Closing a stream cancels any data being transferred or stored.

* URL

    /streams/by-id/*[STREAMID]*/close_stream.cmd

* Methods

    `GET`

* Role

    `admin`

* URL parameters

    | Key          | Description           |
    | ---          | -----------           |
    | *[STREAMID]* | ID of stream to close |

* Results

    `HTTP 200` on successful stream closure

    HTTP error on failure

