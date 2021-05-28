# OpenTokRTC

<img src="https://assets.tokbox.com/img/vonage/Vonage_VideoAPI_black.svg" height="48px" alt="Tokbox is now known as Vonage" />

[![Build Status](https://travis-ci.com/opentok/opentok-rtc.svg?token=qPpN1jG8Wftsn1cafKif&branch=master)](https://travis-ci.com/opentok/opentok-rtc)
[![codecov](https://codecov.io/gh/opentok/opentok-rtc/branch/master/graph/badge.svg)](https://codecov.io/gh/opentok/opentok-rtc/branch/master)

OpenTokRTC is your private web-based video conferencing solution. It is based on the
[OpenTok platform](https://tokbox.com/developer/) (now the Vonage Video API) and uses
the OpenTok SDKs and API. You can deploy the app on your servers to get your own
video conferencing app running on WebRTC.

This repository contains a Node.js server and a web client application.

## Table of Contents

* [Installation](#installation)
  - [Requirements](#requirements)
  - [Installing dependencies](#installing-dependencies)
  - [Basic configuration](#basic-configuration)
* [Runningthe app](#running-the-app)
* [Configuration options](#configuration-options)
  - [OpenTok configuration](#opentok-configuration)
  - [Screen sharing](#screen-sharing)
  - [Phone dial-out](#phone-dial-out)
  - [Google Authentication for Phone dial-out](#google-authentication-for-phone-dial-out)
  - [Archiving](#archiving)
  - [Screen sharing](#screen-sharing)
  - [Feedback](#feedback)
  - [Pre-call test](#pre-call-test)
  - [Adobe Analytics](#adobe-analytics)
  - [ICID tracking](#icid-tracking)
  - [Additional configuration options](#additional-configuration-options)
* [Customizing the UI](#customizing-the-ui)
* [Troubleshooting](#troubleshooting)
* [Health status check](#health-status-check)

---

## Installation

If you want to install OpenTokRTC on your own server, read on. If you want to deploy OpenTokRTC to Heroku, see [`INSTALL-heroku.md`](INSTALL-heroku.md).

### Requirements

You will need to install these dependencies on your machine:

- [NodeJS v12](https://nodejs.org): This version of OpenTokRTC is tested with NodeJS v12 LTS.
- [Redis](https://redis.io): A `redis` server running on `localhost`.

You will also need these an OpenTok API key and secret. You can obtain these by signing up with
[OpenTok/Vonage Video API](https://tokbox.com/account/user/signup).

### Installing dependencies

First, install the dependencies for the server.

If you use [nvm](https://github.com/nvm-sh/nvm), have it use the correct
version of Node (v12):

```sh
$ nvm use
```

Then install the Node module dependencies:

```sh
$ npm install
```

Note: You will need to run these commands as a non-root user, else `bower` will refuse to execute.

### Basic configuration

Once all the dependencies are in place, you will need to set some configuration options
and install the application's dependencies. At a minimum, you need to set options for
the OpenTok API key and secret to be used by the app. You can obtain these from
your [OpenTok account](https://tokbox.com/account).

Other features of the app are enabled and configured using more configuration options,
described in this README.

There are two ways to set configuration options for the application:

* Using a config.json file
* Setting environment variables

#### Using a config.json file

First create create a `config.json` file in the `config` folder.

```sh
$ cd <path-to-opentok-rtc>
$ cp config/example.json config/config.json
```

Edit the config.json file with the following, and replace `<key>` and `<secret>` with
your OpenTok API key and the corresponding API secret:

```js
{
    "OpenTok": {
        "apiKey": "<key>"
        "apiSecret": "<secret>"
    }
}
```

The [config/example.json](config/example.json) file includes settings for other options,
which are described in the [Configuration options](#configuration-options) section below.

#### Setting environment variables

You can set the `TB_API_KEY` and `TB_API_SECRET` to your OpenTok API key and secret.
For example, the following shell commands export these values for use by the app
(replace `<key>` and `<secret>` with your OpenTok API key and the corresponding API secret):

```sh
export TB_API_KEY=<key>
export TB_API_SECRET=<secret>
```

You can set other environment variables to enable and configure other options,
which are described in the [Configuration options](#configuration-options) section.

## Running the app

Ensure that Redis server is running on `localhost`. In a terminal, run:

```sh
redis-server
```

Then start the application in foreground by running:

```sh
$ node server
```

This will start the application on port `8123` by default.

To specify a custom port number, use the `-p` flag when calling `node server`.
For example, to run the application on port 8080, run:

```sh
$ node server -p 8080
```

Additionally, you can start the application as a daemon by passing `-d` flag,
which starts the application and keeps it running in the background so that your terminal
is not blocked:

```sh
$ node server -d
```

To start the server with HTTPS enabled, pass `-S` flag to launch a secure server
along with `-C <dir>` flag to specify a directory that holds SSL certificate files.
To quickly start a secure server, run:

```sh
$ node server -S -C sampleCerts
```

The server expects SSL certificate file to be named `serverCert.pem` and
an SSL private key file to be named `serverKey.pem`. There is a pre-generated,
self-signed SSL certificate pair in the `./sampleCerts` directory.

For detailed information on available options, run `$ node server -h`.

## Configuration options

You can enable and configure UI settings and other options using a config JSON file
or by setting environment variables.

Environment variable settings overwrite any value set with the config JSON file.

The default config JSON file location is config/config.json. This path can be
overwritten by setting the `DEFAULT_JSON_CONFIG_PATH` environment variable.

### OpenTok configuration

The required configuration settings for the OpenTok API key and secret are
described in the [basic configuration](#basic-configuration) section,
earlier in this README.

There are other OpenTok configuration settings (each of which are optional):

* **Publisher resolution** -- You can set the desired resolution of published video.
  The config.json setting is `OpenTok.publisherResolution`. The environment variable
  name is `PUBLISHER_RESOLUTION`.

  The format of the string is "widthxheight", where the width and height are represented
  in pixels. Valid values are "1280x720", "640x480", and "320x240". The published video
  will only use the desired resolution if the client configuration supports it.
  Some browsers and clients do not support each of these resolution settings.

  The default resolution for a stream (if you do not specify a resolution) is 640x480 pixels.
  If the client system cannot support the resolution you requested, the stream will use
  the next largest setting supported.

* **OpenTok.js URL** -- By default, the app uses the latest standard line version
  of OpenTok.js. You can change the OpenTok.js source URL. For example you may
  want to change this to the enterprise line version of OpenTok.js
  (https://static.opentok.com/v2/js/opentok.min.js).

  The config.json setting is `OpenTok.jsUrl`. The environment variable name is `TB_JS_URL`.

* **Session expiration length** -- To conserve Redis memory, the app limits how long
  how long room names are associated with OpenTok session IDs. The app stores the last time
  a session is used and if when we fetch it from Redis we determine it's older than
  this max age (in days). This is the key where that value (in days) should be stored.
  By default, sessions live two days.
  
  The config.json setting is `OpenTok.maxSessionAge`. The environment variable name is `TB_MAX_SESSION_AGE`.

* **OpenTok media mode** -- You can specify whether the OpenTok sessions will use
the OpenTok Media Router ('routed') or not ('relayed'). For more information, see
[The OpenTok Media Router and media modes](https://tokbox.com/developer/guides/create-session/#media-mode).
The default value is 'routed'.

The config.json setting is `mediaMode`. The environment variable name is `MEDIA_MODE`.


**Config.json example:**

```json
{
    "OpenTok": {
      "apiKey": "<key>",
      "apiSecret": "<secret>",
      "publisherResolution": "1280x720",
      "jsUrl": "https://static.opentok.com/v2/js/opentok.min.js",
      "maxSessionAge": 7
    },
    "mediaMode": "routed"
}
```

**Environment variable example:**

```sh
export PUBLISHER_RESOLUTION="1280x720";
export TB_JS_URL="https://static.opentok.com/v2/js/opentok.min.js";
export TB_MAX_SESSION_AGE="7";
```

### Phone dial-out

The app can dial out and add a phone-based end user to the OpenTok session, using the OpenTok
[SIP API](https://tokbox.com/developer/rest/#sip_call). This app uses
[Nexmo](https://www.nexmo.com/) as the SIP application that connects
to OpenTok. (You can also use the OpenTok SIP API to connect to other SIP endpoints.)

To enable this feature:

1. Sign up for a [Nexmo/Vonage](https://dashboard.nexmo.com/sign-up) account.

2. Edit options in the config/config.json file or set environment variables:

   * `SIP.enabled` (config.json) / `SIP_ENABLED` (environment variable) -- Set this to `true`.

   * `SIP.username` (config.json) / `SIP_USERNAME` (environment variable) -- Set this to
     the apiKey for the Nexmo account you created.

   * `SIP.password` (config.json) / `SIP_PASSWORD` (environment variable) -- Set this to
     the apiSecret for the Nexmo account you created.

   * `SIP.requireGoogleAuth` (config.json) / `SIP_REQUIRE_GOOGLE_AUTH` (environment variable) -- See
     [Google Authentication for Phone dial-out](#google-authentication-for-phone-dial-out)
     for instructions on how to limit this functionality to users authenticated by their Google account.

**Config.json example:**

```json
{
    "SIP": {
        "enabled" : true,
        "sipUsername" : "nexmoApiKey",
        "sipPassword" : "nexmoApiSecret",
        "requireGoogleAuth": false
    }
}
```

**Environment variable example:**

```sh
export SIP_ENABLED=true;
export SIP_USERNAME="nexmoApiKey";
export SIP_PASSWORD="nexmoApiSecret";
export SIP_REQUIRE_GOOGLE_AUTH=false;
```

#### Google Authentication for Phone dial-out

You can limit the ability to place outgoing calls to those authenticated by Google.
To enable this feature:

1. Create a Google API Console Project and client ID following the steps detailed here:
   https://developers.google.com/identity/sign-in/web/devconsole-project.

2. Edit options in the config/config.json file or set environment variables:

  * `Google.clientId` (config.json) / `GOOGLE_CLIENT_ID` (environment variable)  -- Set this to your client ID.

  * `Google.hostedDomain` (config.json) / `GOOGLE_HOSTED_DOMAIN` (environment variable)  -- If you wish to
    limit sign in to accounts associated with a hosted domain, set the domain here.

  * `Sip.requireGoogleAuth` (config.json) / `SIP_REQUIRE_GOOGLE_AUTH` (environment variable) -- Set this to`true`
    to require Google authentication for SIP dial-out as detailed in [Phone dial-out](#phone-dial-out).

**Config.json example:**

```json
{
    "Google": {
        "clientId": "yourClientId.apps.googleusercontent.com",
        "hostedDomain": "yourhosteddomain.com"
    },
    "SIP" : {
        "sipUri" : "sip:phoneumber@sip.nexmo.com",
      "sipUsername" : "nexmoApiKey",
      "sipPassword" : "nexmoApiSecret",
      "requireGoogleAuth": true
    }
}
```

**Environment variable example:**

```sh
export GOOGLE_CLIENT_ID=yourClientId.apps.googleusercontent.com;
export GOOGLE_HOSTED_DOMAIN=yourhosteddomain.com;
export SIP_REQUIRE_GOOGLE_AUTH=true;
```

### Archiving

To enable and configure archiving (recording), edit options in the config/config.json file
or set environment variables:

* `Archiving.enabled` (config.json) / `ENABLE_ARCHIVING` (environment variable) -- Set this
   to `true` to include the *Record* button in the UI, allowing the end user to record
   a session. If you also set the `Archiving.archiveAlways` / `ARCHIVE_ALWAYS` to `true`,
   each session is recorded automatically and the UI does not include the *Record* button.
   The default value is `false`.

* `Archiving.archiveAlways` (config.json) / `ARCHIVE_ALWAYS` (environment variable) -- Whether
  to record all sessions automatically (without the end user having to click the *Record* button
  (which is not displayed if this option is set to `true`). The default value is `false`.

 * `Archiving.pollingInitialTimeout` (config.json) / `ARCHIVE_TIMEOUT` (environment variable -- The initial
   polling timeout (in milliseconds) for archive status change updates. Set this to 0 to disable polling.
   The default value is 5000.

* `Archiving.pollingTimeoutMultiplier` (config.json) / `TIMEOUT_MULTIPLIER`
  (environment variable -- The timeout multiplier for archive status updates. If the first
  archive status update polling fails, subsequent polling intervals will apply this multiplier
  successively. Set to a lower number to poll more often. The default value is 1.5.

* `Archiving.archivingManager.enabled` (config.json) / `ENABLE_ARCHIVE_MANAGER`
  (environment variable) -- Whether the UI will include a list of archive recordings for the session.
  The end user can click an archive to view the recording. This setting is only meaningful
  if `Archiving.enabled` / `ENABLE_ARCHIVING` is set to `true`. The default value is `false`.
  
  *Note:* The OpenTokRTC app retrieves the archive recordings from the OpenTok cloud storage. However,
  you may choose to have archives uploaded to an Amazon S3 bucket or an Azure container. If you do, you
  will need to modify the OpenTokRTC code to obtain archive recordings from the chosen storage location.
  For more information, see [Archive storage](https://tokbox.com/developer/guides/archiving/#storage)
  in the OpenTok developer guides.

**Config.json example:**

```json
{
    "Archiving": {
        "enabled": true,
        "archiveAlways": true,
        "pollingInitialTimeout": 10000,
        "pollingTimeoutMultiplier": 1,
        "archiveManager": {
            "enabled": true,
        }
    }
}
```

**Environment variable example:**

```sh
export ENABLE_ARCHIVING=true;
export ARCHIVE_ALWAYS=true;
export ARCHIVE_TIMEOUT=10000;
export TIMEOUT_MULTIPLIER=1;
export ENABLE_ARCHIVE_MANAGER=true;
```

### Screen sharing

To enable and configure screen sharing, edit options in the config/config.json file
or set environment variables:

* `Screensharing.enabled` (config.json) / `ENABLE_SCREENSHARING` (environment variable) -- Whether
  to enable screen sharing. The default value is `false`.

* `Screensharing.chromeExtensionId` (config.json) / `CHROME_EXTENSION_ID`
  (environment variable) -- The Chrome add-on extension ID for screen sharing.
  Note: an extension is no longer required for screen sharing in Chrome 72+ and Opera 59+.
  The browser prompts the end user for access to the screen as it would for access to the camera.
  Use a screen-sharing extension only if you need to support older versions of Chrome and Opera.
  The default value is `null`.

* `Screensharing.annotations` (config.json) / `ENABLE_ANNOTATIONS` (environment variable) -- Whether
  to enable annotations in screen sharing. With annotations enabled, the room toolbar includes
  an *Annotate* button is This is only meaningful if screen sharing is enabled.  The default value
  is `true`.

To learn more about how screen sharing works in OpenTok, see the [guide on screen
sharing](tokbox.com/developer/guides/screen-sharing/).

**Config.json example:**

```json
{
    "Screensharing": {
        "enabled": true,
        "chromeExtensionId": "cfhdojbkjhnklbpkdaibdccddilifddb",
        "annotations": {
            "enabled": true
        }
    } 
}
```
**Environment variable example:**

```sh
export ENABLE_SCREENSHARING=true;
export CHROME_EXTENSION_ID=cfhdojbkjhnklbpkdaibdccddilifddb;
export ENABLE_ANNOTATIONS=true;
```

### Feedback

The app lets the developer POST feedback data to an endpoint on your HTTP server. To enable
and configure this, edit options in the config/config.json file or set environment variables:

* `Feedback.url` (config.json) / `FEEDBACK_URL` (environment variable) -- The URL to send a POST
  request with feedback data. Leave this as an empty string or undefined to disable issue reporting.
  The default value is ''.

* `Feedback.reportIssueLevel` (config.json) / `REPORT_ISSUE_LEVEL` (environment variable) -- The audio
  and video scores in the feedback form are between 1 (awful) and 5 (excellent). When the feedback form
  is submitted, if an audio or video score is less than or equal to the report issue level, the app calls
  `OT.reportIssue()`. This reports an issue, which you can view in OpenTok Inspector. For more information,
  see [Reporting an issue](https://tokbox.com/developer/guides/debugging/js/#report-issue) in the OpenTok
  developer guides. Set this to 0 to disable issue reporting. The default value is 3.

**Config.json example:**

```json
{
    "Feedback": {
        "url": "https://my-app.com/feedback-endpoint/",
        "reportIssueLevel": 3
    }
}
```

**Environment variable example:**

```sh
export FEEDBACK_URL="https://my-app.com/feedback-endpoint/";
export REPORT_ISSUE_LEVEL=3;
```

Instead of posting feedback to an endpoint on your HTTP server, you can use
[Hotjar](https://www.hotjar.com/) to collect feedback. Edit the following options
in the config/config.json file or set environment variables:

* `Feedback.hotjarId` (config.json) / `HOTJAR_ID`: (Optional, default value: null) The unique site ID
  for the application. This helps Hotjar tracking code to collect feedback data.

* `Feedback.hotjarVersion` (config.json) / `HOTJAR_VERSION`: (Optional, default value: null) Version of
  the Tracking Code using with hjsv.

* `Feedback.enableFeedback` (config.json) / `ENABLE_FEEDBACK` (environment variable) -- Set this
  to `true` when using Hotjar for feedback.

* `Feedback.reportIssueLevel` (config.json) / `REPORT_ISSUE_LEVEL` (environment variable) -- This setting
  is ignored when using Hotjar for feedback.

* `Feedback.url` (config.json) / `FEEDBACK_URL` (environment variable) -- This setting
  is ignored when using Hotjar for feedback.

**Config.json example:**

 ```json
{
      "Feedback": {
          "hotjarId": "your-hotjar-id",
          "hotjarVersion": "your-hotjar-version",
          "enableFeedback": true,
      }
}
```

**Environment variable example:**

```sh
export HOTJAR_ID=your-hotjar-id;
export HOTJAR_VERSION=your-hotjar-version;
export ENABLE_FEEDBACK=true;
```

### Pre-call test

Set the the `TB_PRECALL_API_KEY` and `TB_PRECALL_API_SECRET` environment variables
to the the OpenTok API key and secret to use for the test session used by
the precall-test. Or set these in the config.json file:

**Config.json example:**

```json
{
    "precallTest": {
        "apiKey": "46049502",
        "apiSecret": "0f4a63f629cec64ebdc5552974fe2566d2eb2835"
    }
}
```

These are optional. If you do not set these, the pre-call test will use the same
API key and secret that is used for the main OpenTok session used in the room.

You can disable the pre-call test by setting the `ENABLE_PRECALL_TEST`
environment variable to `false`. Or you can disable it using the config file:

**Config.json example:**

```json
{
    "precallTest": {
        "enabled": false
    }
}
```

### Adobe Analytics

The app lets the developer configure Adobe Analytics to track user information.
To configure this, edit options in the config/config.json file or set environment variables:

* `adobeTracking.url` (config.json) / `ADOBE_TRACKING_URL` (environment variable) -- The URL
  to download the custom embed code. The default value is ''.

* `adobeTracking.primaryCategory` (config.json) / `ADOBE_TRACKING_PRIMARY_CATEGORY`
  (environment variable) -- The value that will be included in Adobe Analytics
  object field: `digitalData.page.pageInfo.primaryCategory`. The default value is ''.

* `adobeTracking.siteIdentifier` (config.json) / `ADOBE_TRACKING_SITE_IDENTIFIER`
  (environment variable) -- The value that will be included in Adobe Analytics
  object field: `digitalData.page.pageInfo.siteIdentifier`. The default value is ''.

* `adobeTracking.functionDept` (config.json) / `ADOBE_TRACKING_FUNCTION_DEPT`
  (environment variable) -- The value that will be included in Adobe Analytics
  object field: `digitalData.page.pageInfo.functionDept`. The default value is ''.

### ICID tracking

You can set ICID query string values for the end (/thanks) page URLs with these settings:

* `icid.startBuiliding` (config.json) / `START_BUILDING_ICID` (environment variable) -- The ICID
  for the "How to make this app better" and "Start building for free" links. The default value is ''.

* `icid.contactUs` (config.json) / `CONTACT_US_ICID` (environment variable) -- The ICID for
  the "Contact us" link. The default value is ''.

### Additional configuration options

* `appName` (config.json) / `APP_NAME` (environment variable) -- The name of the application
  displayed in the precall widget, in the top of the room page, and in the end (/thanks) page.
  The default value is 'Vonage Video Conferencing'.

* `introText` (config.json) / `INTRO_TEXT` (environment variable) -- The text displayed under
  the application name in the precall widget. The default value is 'Welcome to Video Conferencing'.

* `introFooterLinkText` (config.json) / `INTRO_FOOTER_LINK_TEXT` (environment variable) -- The text
  for the optional link displayed under the intro text in the precall widget. If no value is
  specified (or if it is set the an empty string), no link is displayed. The default value is ''.

* `introFooterLinkUrl` (config.json) / `INTRO_FOOTER_LINK_URL` (environment variable) -- The URL
  for the optional link displayed under the intro text in the precall widget. The default value is ''.

* `helpLinkText1` (config.json) / `HELP_LINK_TEXT_1` (environment variable) -- The text
  for the first help link displayed after "Need help?" at the top of the precall widget.
  If you do not set this option, the "Need help" section is omitted.

* `helpLinkUrl1` (config.json) / `HELP_LINK_URL_1` (environment variable) -- The URL for
  the first help link the precall widget.

* `helpLinkText2` (config.json) / `HELP_LINK_TEXT_2` (environment variable) -- The text
  for the second help link displayed after "Need help?" at the top of the precall widget.
  If you do not set this option, second help link is omitted.

* `helpLinkUrl2` (config.json) / `HELP_LINK_URL_2` (environment variable) -- The URL for
  the second help link the precall widget.

* `showTos` (config.json) / `SHOW_TOS` (environment variable) -- Whether the app will display the terms of service
  dialog box and require the user to agree to the terms before joining a room. The default value is `false`.

* `meetingsRatePerMinute` (config.json) / `MEETINGS_RATE_PER_MINUTE` (environment variable) -- Determines the maximum amount of new meetings that
  can be created in a minute. Users will be allowed to join a meeting that already exists. Otherwise a message
  will appear telling them that the service is not available at the moment. If the value is set to any negative
  number, rate limiting will be turned off and all meetings will be allowed. If this value is set to 0, all new
  meetings will be rejected. The default value is -1.

* `minMeetingNameLength` (config.json) / `MIN_MEETING_NAME_LENGTH` (environment variable) -- The minimum length of
  meeting names created. The default value, 0, indicates that there is no minimum
  length. (You can set this in the config file using the `minMeetingNameLength` setting.)
  The default value is 0.

* `allowIframing` (config.json) / `ALLOW_IFRAMING` (environment variable) -- Controls the server-side restriction on
  allowing content to load inside an iframe. The allowed values are:

  - 'always' -- Allow iframing unconditionally (note that rtcApp.js should also be changed
     to reflect this, this option only changes what the server allows)

  - 'never'  Set X-Frame-Options to 'DENY' (Deny loading content in any iframe)

  - 'sameorigin': Set X-Frame-Options to 'SAMEORIGIN' (Only allow iframe content to be loaded
    from pages in the same origin)

  We don't allow restricting iframe loading to specific URIs because it doesn't work on Chrome.
  The default value is 'never'.

* `useGoogleFonts` (config.json) / `USE_GOOGLE_FONTS` (environment variable) -- Whether the client app will load
   the Open Sans font (the main font used in the user interface) from the Google font library
   (fonts.googleapis.com) or not. The default value is `true`.

* `jqueryUrl` (config.json) / `JQUERY_URL` (environment variable) --
  Route of the CDN that will be used to load JQuery scripts. The default value is
  'https://ajax.googleapis.com/ajax/libs/jquery/'.

* `oneTrustCookieConsentUrl` (config.json) / `ONE_TRUST_COOKIE_CONSENT_URL` (environment variable) -- Route of the OneTrust cookie consent URL.
   Leave blank or unset to turn the feature off. This setting is unset by default.

* `enableMuteAll` (config.json) / `ENABLE_MUTE_ALL` (environment variable) -- Whether to show the Mute All
  control in the top menu of the room. (You can set this in the config file
  using the `enableMuteAll` setting.) The default value is `true`.

* `enableStopReceivingVideo` (config.json) / `ENABLE_STOP_RECEIVING_VIDEO` (environment variable) -- Whether to show
  the Stop Receiving Video control in the top menu of the room. The default value is `true`.
 
* `maxUsersPerRoom` (config.json) / `MAX_USERS_PER_ROOM` (environment variable) -- The maximum number
  of users allowed in a room at the same time. Set this to 0, the default, to allow
  any number of users. The default value is 0.

* `enableRoomLocking` (config.json) / `ENABLE_ROOM_LOCKING` (environment variable) -- Whether or not
  to include the Lock Meeting command to users in the options menu. This command allows users
  to prevent new participants from joining a meeting. The default value is `true`.

* `autoGenerateRoomName` (config.json) / `AUTO_GENERATE_ROOM_NAME` (environment variable) -- Whether or not to
  auto-generate the room name on behalf of the user. If this setting is turned on, we will use haikunator to generate
  room names for new rooms. If turned off, users will be prompted to enter a room/meeting name when they visit
  the landing page and won't be allowed to move forward until they do so. The default value is `true`.

* `enableEmoji` (config.json) / `ENABLE_EMOJI` (environment variable) -- Whether or not to
  enable emoji support in the text chat widget.

## Customizing the UI

For information on how to customize the OpenTokRTC UI, see [CUSTOMIZING-UI.md](CUSTOMIZING-UI.md).

## Troubleshooting

**"ServerPersistence: Timeout while connecting to the Persistence Provider! Is Redis running?**

Ensure Redis server is running on localhost (run `redis-server` in the command line)
and restart OpenTokRTC.

**OpenTokRTC does not work when served over HTTP.**

Browser security policies require HTTPS for WebRTC video communications. You will need to set up
the app to be served over HTTPS. You can set up a
[secure reverse-proxy](https://www.nginx.com/resources/admin-guide/nginx-https-upstreams/)
to your OpenTokRTC port using Nginx. For details, read
[this post](https://tokbox.com/blog/the-impact-of-googles-new-chrome-security-policy-on-webrtc/).

**UI looks broken**

UI assets are compiled as part of the build process when installing application dependencies using `npm install`. If the web application UI still looks broken, run the following commands in the root directory of the application:

```
$ bower install
$ npm run clientBuild
```

We recommend that you run the application as a non-root user. However, if you are running the application as the `root` user, you will additionally need to tell `bower` to allow the root user to install dependencies, else bower will refuse to work:

```
$ bower install --allow-root
```

### Health status check

There is a health status check endpoint at /server/health. You can load this URL to check whether
the app is able to connect to all required external services. On success, this health check
endpoint sends a response with the HTTP status code set to 200 and the JSON like the following:

```json
{
  "name": "opentok-rtc",
  "version": "4.1.1",
  "gitHash": "312903cd043d5267bc11639718c47a9b313c1663",
  "opentok": true,
  "googleAuth": true,
  "status": "pass"
}
```

The JSON includes the following properties:

- `name` -- `"ot-embed"`

- `version` -- The version number deployed (from package.json)

- `git_hash` -- The git commit deployed

- `opentok` -- Whether the OpenTok API check passed. The app uses the OpenTok Node.js SDK,
  which connects to the OpenTok API server to create OpenTok sessions.

- `googleAuth` -- Whether the Google Authentication check passed. This check is only run if the app
  uses Google Authentication for making outbound SIP calls. (See [Google Authentication for
  Phone dial-out](#google-authentication-for-phone-dial-out).)

- `status` -- "pass" (if all checks pass) or "fail" (if any check fails)

On failure, the health status check endpoint returns a response with
the HTTP status code set 400 and JSON like the following:

```json
{
  "name": "opentok-rtc",
  "version": "4.1.1",
  "git_hash": "312903cd043d5267bc11639718c47a9b313c1663",
  "opentok": false,
  "error": "OpenTok API server timeout exceeded.",
  "status": "fail"
}
```

Note that upon failure, the `status` property is set to `"fail"` and the `error` property
is set to an error message. Also, the property for the failing test, such as `opentok`,
will be set to `false`. If a test fails, the health check will not run subsequent tests.

## Development and Contributing

Interested in contributing? We :heart: pull requests! See the
[Contribution](CONTRIBUTING.md) guidelines.

## Getting Help

We love to hear from you. If you have questions, comments or find a bug, let us know:

* Open an issue on this repository <https://github.com/opentok/opentok-rtc/issues>.

* See <https://support.tokbox.com/> for support options.

* Tweet at us! We're [@VonageDev](https://twitter.com/VonageDev) on Twitter.

* Or [join the Vonage Developer Community Slack](https://developer.nexmo.com/community/slack).

## Further Reading

Check out the OpenTok documentation at <https://tokbox.com/developer/>.
