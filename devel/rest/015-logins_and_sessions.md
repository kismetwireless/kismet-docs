---
title: "Logins and sessions"
permalink: /docs/devel/webui_rest/logins/
docgroup: "devel-rest"
excerpt: "Kismet uses a standard login and session cookie system which is easily supported by most HTTP libraries."
---

Kismet uses HTTP basic-auth to submit login information, and session cookies to retain login state.  

As of `2019-04-git`, all interaction with the Kismet server requires a login.

A session will automatically be created during authentication to any endpoint which requires login information, and returned in the `KISMET` session cookie.

Logins may be manually validated against the `/session/check_session` endpoint if validating user-supplied credentials.

### First login
The first time Kismet is started, the user must set a password.  Until the password is set, endpoints which require a login will be disabled and will return an error.

External tools (and UI implementations) can check if the initial password has been set.  Typically this API should not be used unless you are implementing a new Kismet UI that needs to provision the first-use scenario.  If you wish to pre-configure a Kismet server username and login, refer to [the webserver configuration documentation](/docs/readme/webserver).

* URL 

    /session/check_setup_ok

* API added

    `2019-01`

* Methods

    `GET`

* Result

    `HTTP 200` is returned if the initial setup has been completed.

    `HTTP 406` NOT ACCEPTABLE is returned if the credentials are hard-coded into the global configuration.  

    HTTP error is returned if the initial setup has not been completed and the user is required to set a password.

* Notes

    If the password is set in the global Kismet configuration files, such as `kismet_httpd.conf` or `kismet_site.conf`, this API will not be available and will return an error.

### Setting the login

The initial login must be set before the user can access any restricted endpoints.  This endpoint can also be used to set a new login.

* URL

    /session/set_password

* API added

    `2019-01`

* Methods

    `POST`

* POST parameters

    Standard `HTTP POST` variables, including:

    | Key      | Description                                                                   |
    | ---      | -----------                                                                   |
    | username | Optional, username for new login.  If not provided, will default to `kismet`. |
    | password | Required, new password.                                                       |

* Results

    `HTTP 200` is returned if the password set was successful. 
    `HTTP 406` NOT ACCEPTABLE is returned if the server password is set in the global configuration. 
    HTTP error is returned if the password set was unsuccessful. 

* Notes

    Setting the initial password does not require a login.  Subsequent attempts to change the login and password *will* require a valid login session for the current password value, and will not invalidate any current login sessions.

    If the password is set in the global Kismet configuration files, such as `kismet_httpd.conf` or `kismet_site.conf`, this API will not be available and will return an error.

### Checking sessions

A script can check for a valid session and prompt the user to take action if a session is no longer valid.

Login data may be provided; if the session is not valid, and valid login data is provided via basic auth, a session will be created.

* URL

    /session/check_session

* Methods

    `GET`

* Result

    `HTTP 200` is returned if the session is valid.

    HTTP error returned if session is *not* valid and supplied login data, if any, is not valid.

### Checking logins

A script may need to check for a valid login and prompt the user to take action if the login credentials are not valid.

Session cookies will be ignored while checking logins.

* URL

    /session/check_login

* Methods
    
    `GET`

* Result

    `HTTP 200` is returned if the login is valid.

    HTTP error returned if the login is not valid.

