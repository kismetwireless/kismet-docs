---
title: "Logins and sessions"
permalink: /docs/devel/webui_rest/logins/
toc: true
docgroup: "devel-rest"
excerpt: "Kismet uses a standard login and session cookie system which is easily supported by most HTTP libraries."
---

Kismet uses HTTP basic-auth to submit login information, and session cookies to retain login state.  

As of `2019-04-git`, all interaction with the Kismet server requires a login.

As of `2020-10-git`, all endpoints on the Kismet server support a role:  All login sessions made with the admin username and password are granted the `admin` role.  The `admin` role has access to all endpoints.  Additional sessions may be set by creating API keys with an assigned role which restricts the available endpoints of the session.

A session will automatically be created during authentication to any endpoint which requires login information, and returned in the `KISMET` session cookie.

Logins may be manually validated against the `/session/check_session` endpoint if validating user-supplied credentials.

## Providing logins

Kismet accepts logins via HTTP Basic authentication, session cookie, and GET URI parameters.

If the administrator username and password is provided via Basic auth or via get URI parameters, a session cookie is created (if one does not already exist) or found, and returned in the `KISMET` cookie parameter.

*Added 2020-10* API-token-only consumers of the API should provide *ONLY* the API token given, and supply it in the `KISMET` cookie or URI parameter.

## URI parameters

Some mechanisms, such as websockets, do not commonly support HTTP Basic Auth or cookie passing, and must use URI parameters:

| Key      | Value                             |
| ---      | -----                             |
| user     | Administrator username            |
| password | Administrator password            |
| KISMET   | Kismet session cookie / API token |

The same rules apply to the user and password and session token login process - if a valid username and password is provided, it will return a session token in the set-cookie parameter for future logins.

## Login roles

*Added 2020-10* Kismet has begun adding login roles to support an API key style access method for future expansion and tools.

An API token has a specific login role which it is permitted to access.

Roles are not inherited; a role limits the API token to those roles.

If a valid login is supplied, the session is assigned the role `admin`, which has unrestricted access to all APIs.  Providing both the Kismet admin username and password, and a session token with lesser permissions, will ignore the session token and apply the administrative role.

## First login

The first time Kismet is started, the user must set a password.  Until the password is set, endpoints which require a login will be disabled and will return an error.

External tools (and UI implementations) can check if the initial password has been set.  Typically this API should not be used unless you are implementing a new Kismet UI that needs to provision the first-use scenario.  If you wish to pre-configure a Kismet server username and login, refer to [the webserver configuration documentation](/docs/readme/webserver).

* URL 

    /session/check_setup_ok

* API added

    `2019-01`

* Methods

    `GET`

* Role

    n/a

* Result

    `HTTP 200` is returned if the initial setup has been completed.

    `HTTP 406` NOT ACCEPTABLE is returned if the credentials are hard-coded into the global configuration.  

    HTTP error is returned if the initial setup has not been completed and the user is required to set a password.

* Notes

    If the password is set in the global Kismet configuration files, such as `kismet_httpd.conf` or `kismet_site.conf`, this API will not be available and will return an error.

## Setting the login

The initial login must be set before the user can access any restricted endpoints.  This endpoint can also be used to set a new login.

* URL

    /session/set_password

* API added

    `2019-01`

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    Standard `HTTP POST` variables, including:

    | Key      | Description                                                                   |
    | ---      | -----------                                                                   |
    | username | Required, username for login.                                                 |
    | password | Required, new password.                                                       |

* Results

    `HTTP 200` is returned if the password set was successful. 
    `HTTP 406` NOT ACCEPTABLE is returned if the server password is set in the global configuration. 
    HTTP error is returned if the password set was unsuccessful. 

* Notes

    Setting the initial password does not require a login.  Subsequent attempts to change the login and password *will* require a valid login session for the current password value, and will not invalidate any current login sessions.

    If the password is set in the global Kismet configuration files, such as `kismet_httpd.conf` or `kismet_site.conf`, this API will not be available and will return an error.

## Checking sessions

A script can check for a valid session and prompt the user to take action if a session is no longer valid.

Login data may be provided; if the session is not valid, and valid login data is provided via basic auth, a session will be created.

* URL

    /session/check_session

* Methods

    `GET`

* Role

    n/a

* Result

    `HTTP 200` is returned if the session is valid.

    HTTP error returned if session is *not* valid and supplied login data, if any, is not valid.

## Checking logins

A script may need to check for a valid login and prompt the user to take action if the login credentials are not valid.

Session cookies will be ignored while checking logins.

* URL

    /session/check_login

* Methods
    
    `GET`

* Role

    n/a

* Result

    `HTTP 200` is returned if the login is valid.

    HTTP error returned if the login is not valid.

## API tokens and roles

As of `2020-11`, Kismet supports the use of API tokens and roles to restrict the actions of sessions.  Predefined roles include:

| Role       | Description                                                                                                      |
| ----       | -----------                                                                                                      |
| admin      | Main role with access to all endpoints.  Logins created via HTTP auth are automatically assigned the admin role. |
| readonly   | Read-only role with access to endpoints which do not modify any devices, state, or configuration                 |
| scanreport | Role able to submit device/network scan reports, via the Wi-Fi and Bluetooth scanning-mode API                   |
| datasource | Role for remote capture websocket sources                                                                        |

Roles are not inherited or cascading; for instance a `readonly` role does not have access to reporting scans or acting as remote datasources.  The only role with access to all endpoints is `admin`.

## Listing API tokens

* URL

    /auth/apikey/list.json

    /auth/apikey/list.ekjson

    /auth/apikey/list.itjson

* API Added

    `2020-11`

* Methods

    `GET`

* Role

    `admin`

* Notes

    If `httpd_allow_auth_view` is not set to `true` in the Kismet configuration, the response will not include then HTTP auth token.

* Results

    `HTTP 200` on success and a JSON array of provisioned logins.

    HTTP error on failure


## Creating API tokens

A new API token can be generated only if the `httpd_allow_auth_creation` option in the Kismet configuration is set to `true`.

If `httpd_allow_auth_view` is not set to true, API tokens may only be viewed when they are created, or by inspecting the session JSON file on the filesystem.

* URL

    /auth/apikey/generate.cmd

* API Added

    `2020-11`

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key         | Description                                                                     |
    | ----------- | ----------------------------------------                                        |
    | name        | Name of API token, must be unique                                               |
    | role        | Role for API token                                                              |
    | duration    | Duration, in seconds, of API token validity.  May be `0` for a permanent token. |

* Notes

    `httpd_allow_auth_creation` must be set to true or this API will return an error condition.

* Results

    `HTTP 200` on success and a plain-text response of the new token

    HTTP error on failure

## Revoking API tokens

An API token may be revoked only if the `httpd_allow_auth_creation` option in the Kismet configuration is set to `true`.

* URL

    /auth/apikey/revoke.cmd

* API Added

    `2020-11`

* Methods

    `POST`

* Role

    `admin`

* POST parameters

    A [command dictionary](/docs/devel/webui_rest/commands/) containing:

    | Key         | Description                              |
    | ----------- | ---------------------------------------- |
    | name        | Name of API token to revoke              |

* Notes

    `httpd_allow_auth_creation` must be set to true or this API will return an error condition.

* Results

    `HTTP 200` on success

    HTTP error on failure


