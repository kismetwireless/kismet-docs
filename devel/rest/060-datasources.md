---
title: "Datasources"
permalink: /docs/devel/webui_rest/datasources/
toc: true
docgroup: "devel-rest"
excerpt: "APIs for accessing defined, and potential, datasources, as well as controlling the behavior of datasources."
---

Datasources in Kismet capture information - typically packets - and return them to the Kismet server for processing.  Typically, a datasource is analogous to a network interface, but may encompass other capture options such as SDR.

Datasources are defined on startup in `kismet.conf` via `source=...` or on the command line with `-c source definition...`.

## Supported datasource types

Datasource types are defined by the Kismet server code and any plugins loaded.  Clients can use this API to determine if Kismet has been started with the required plugins or compiled-in optional datasources.

* URL

    /datasource/types.json

    /datasource/types.ekjson

    /datasource/types.itjson

* Methods

    `GET`

* Role

    `readonly`

* Result

    Array of all defined datasource types

## Defaults

Datasource defaults are defined in `kismet.conf` and include default hopping behavior, hopping speeds, and other generic options.  If a datasource is defined without these options specified, the defaults are used.

* URL

    /datasource/defaults.json

* Methods

    `GET`

* Role

    `readonly`

* Result

    Dictionary of default values

## Listing datasources

Datasource listings include the definition of the source, current operational status, driver and backend information, packet counts, and historical data.

* URL

    /datasource/all_sources.json

* Methods

    `GET`

* Role

    `readonly`

* Result

    Array of active data sources and all current information.

## Querying specific datasources

* URL

    /datasource/by-uuid/*[UUID]*/source.json

* Methods

    `GET`

* Role

    `readonly`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* Results

    Returns dictionary object of source specified by *UUID*

## Listing interfaces

A *datasource* is a Kismet capture source which provides data.  An *interface* is a potential device a datasource may capture from, such as `wlan0` or a RTLSDR USB device.  This API is used by the Kismet web interface to add new datasources runtime.

* URL

    /datasource/list_interfaces.json

    /datasource/list_interfaces.ekjson

    /datasource/list_interfaces.itjson

* Methods

    `GET`

* Role

    `admin`

* Result

    Array of auto-detected interfaces, currently attached datasources if any, and parameters.

## Adding sources

Datasources may be dynamically added at runtime.  New data sources require a full datasource definition, as if they were in a `source=` config line.

* URL

    /datasource/add_source.cmd

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key        | Description            |
    | ---        | -----------            |
    | definition | Source definition line |

* Result

    `HTTP 200` is returned *when the source is successfully added*.  A source *may be added which cannot be activated* if the definition is incorrect or the interface is not available.

    HTTP error is returned if the source could not be added.

* Notes

    This API will block until the source is added to Kismet.  This may be up to several seconds but typically is instant.


## Setting channels

Datasource channel configurations be configured to automatically hop over a list of channels, or remain fixed on a single channel.  Channels are defined as strings and may contain complex information, such as `11HT40-` for a Wi-Fi channel 11 using 40MHz 802.11n.  The exact format of a channel depends on the PHY type being configured.

* URL

    /datasource/by-uuid/*[UUID]*/set_channel.cmd

* Methods

    `POST`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key      | Desc                                                                                                                                                                                        |
    | -------- | ----------------------------------------                                                                                                                                                    |
    | channel  | Single channel; This disables channel hopping and sets a specific channel.  Channel format depends on the source.                                                                           |
    | hoprate  | Channel hopping speed, as channels per second.  For timing greater than a second, rate can be calculated with the formula `hoprate = 1 / (6 / N)` where N is the number of hops per minute. |
    | channels | The list of channel strings to use in hopping                                                                                                                                               |
    | shuffle  | Integer 0 or 1, treated as boolean, tells the source to shuffle the channel list                                                                                                            |

* Results

    `HTTP 200` is returned if the channel set command succeeded.  Individual channels may not be tunable by the source.

    HTTP error is returned if the channel set command is not successful.

* Notes 

    This API will block until the source channel configuration is complete.  This may be up to several seconds but typically is instant.

    The exact behavior of the source depends on the provided parameters:
        * If the `channel` option is present, `hoprate`, `channels`, and `shuffle` will be ignored if present.  The source will be locked to the single provided channel.
        * If the `channels` option is present, `hoprate`, `channels`, and `shuffle` are optional.  If they are not provided, the current values in the source are used.
        * If *only* `hoprate` is present, and `channel` or `channels` is not present, the channel hopping rate will be changed and the current hopping pattern and channels list will be retained.

* Examples

    `{'channel': "6HT40"}` will lock to Wi-Fi channel 6, HT40+ mode

    `{'channels': ["1", "2", "6HT40+", "6HT40-"]}` will change the channel hopping list but retain the hopping rate and shuffle

    `{'channels': ["1", "2", "3", "4", "5"], 'hoprate': 1}` will change the channel hopping rate to once per second over the given list

    `{'hoprate': 5}` will set the hop rate to 5 channels per second, using the existing channels list in the datasource 

## Set channel hopping

Turn on channel hopping for the specified source, without altering the channel parameters.  Typically used to enable hopping after inspecting a single channel.

* URL

    /datasource/by-uuid/*[UUID]*/set_hop.cmd

* Methods

    `POST`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* POST parameters:

    None

* Results

    `HTTP 200` on success

    HTTP error if unable to set hopping

## Closing sources

Sources can be closed and will no longer be processed.  A source will remain closed unless reopened, unless it is a remote capture source - remote capture sources are re-opened when the remote source reconnects.

* URL

    /datasource/by-uuid/*[UUID]*/close_source.cmd 

    /datasource/by-uuid/*[UUID]*/disable_source.cmd

* Methods

    `GET` `POST`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* Results

    `HTTP 200` is returned if the source is successfully stopped

    HTTP error is returned if the source could not be stopped

## Opening closed sources

Closed sources can be re-opened.  Re-opening a source uses the existing source definition.

* URL

    /datasource/by-uuid/*[UUID]*/open_source.cmd

    /datasource/by-uuid/*[UUID]*/enable_source.cmd

* Methods

    `GET` `POST`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* Results

    `HTTP 200` is returned if the open command succeeds.  The source may still encounter errors.

    HTTP error is returned if the source could not be opened.

## Pausing sources

Paused sources are not closed, but packets received from them will be discard.  This can be used to silence a source in some circumstances, but does not require shutting down the source and re-acquiring it later.

* URL

    /datasource/by-uuid/*[UUID]*/pause_source.cmd 

* Methods

    `GET` `POST`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* Results

    `HTTP 200` is returned if the pause command succeeds.

    HTTP error is returned if the source could not be paused.

## Resuming sources

A paused source can be resumed; upon resumption, new packets will be processed from this source.  Packets received while the source was paused are lost.

* URL 

    /datasource/by-uuid/*[UUID]*/resume_source.cmd 

* Methods 

    `GET`

* Role

    `admin`

* URL parameters 

    | Key      | Description |
    | ---      | ----------- |
    | *[UUID]* | Source UUID |

* Results

    `HTTP 200` is returned if the resume command succeeds.

    HTTP error is returned if the source could not be resumed.

## Remote capture websocket

Kismet remote capture tools can connect to Kismet via a websocket.  Contained within the websocket is the standard [Kismet remote capture](/docs/devel/datasources/) protocol, including the remote capture handshake.

* URL

    /datasource/remote/remotesource.ws

* API Added

    `2020-10`

* Methods

    `WEBSOCKET`

* Role

    `datasource`

* Result

    Interactive websocket speaking the remote capture protocol.

* Notes

    This websocket is designed for use with Kismet remote capture tools, implemented by the [capframework](/docs/devel/datasources_capframework/) library or the KismetExternal python3 library.

