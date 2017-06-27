![logo](./tokbox-logo.png)

# OpenTokRTC v2
[![Build Status](https://travis-ci.com/opentok/OpenTokRTC-V2.svg?token=qPpN1jG8Wftsn1cafKif&branch=master)](https://travis-ci.com/opentok/OpenTokRTC-V2)

OpenTokRTC is your private web-based video conferencing solution. It is based on the TokBox
[OpenTok platform](https://tokbox.com/developer/) and uses the OpenTok SDKs and API. You can deploy
OpenTokRTC on your servers to get your own Google Hangouts alternative running on WebRTC.

This repository contains a NodeJS server and a web client application.

## Table of Contents

- [Installation](#installation)
  - [Requirements](#requirements)
  - [Setting up](#setting-up)
- [Running](#running)
- [Configuration options](#configuration-options)
- [Screen sharing](#screen-sharing)
- [Troubleshooting](#troubleshooting)

## Installation
If you want to install OpenTokRTC on your own server, read on. If you want to deploy OpenTokRTC to Heroku, see [`INSTALL-heroku.md`](INSTALL-heroku.md).

### Requirements

You will need these dependencies installed on your machine:

- [NodeJS v4+](https://nodejs.org): This version of OpenTokRTC is tested with NodeJS v4 LTS.
- [Redis](https://redis.io): A `redis` server running on `localhost`. Redis is used for storing session data.
- [Grunt](http://gruntjs.com): Used for bundling assets and running tests.

You will also need these API subscriptions:

- [OpenTok](https://tokbox.com): An OpenTok API key and secret. You can obtain these by signing up with [TokBox](https://tokbox.com).
- [Firebase](https://firebase.google.com) (Optional): A Firebase app and secret. Firebase is used for storing archive data of video conferences. You will need this only if you want to enable Archive Management (In app playback and download of recordings) of conference rooms.

### Setting up

Once all the dependencies are in place, you will need to set some configuration options and install the applications dependencies.

First, change directory to where you have downloaded OpenTokRTC.
Then create create the file `config.json` in the `config` folder.

```sh
$ cd <path-to-OpenTokRTC>
$ touch config/config.json
```

Copy and paste the OpenTok config, Replacing `<key>` and `<secret>` with your OpenTok API key and the corresponding API secret:

```js
{
    "OpenTok": {
        "apiKey": "<key>",
        "apiSecret": "<secret>"
    }
}
```

If you want to use archive management, set up Firebase configuration. Copy and paste the following json after the OpenTok configuration (but still within outer {}) and replace `<appurl>` with your Firebase application URL and `<appsecret>` with the secret for that Firebase app:

```js
,   
"Firebase": {
    "dataUrl": "<appurl>",
    "authSecret": "<appsecret>"
}
"Archiving": {
    "archiveManager": {
        "enabled": true
    }
}
```

For more configuration options, see [detailed configuration options](#configuration-options) below:

Next, set up the dependencies for the server:

```sh
$ npm install
```

## Running

Ensure that Redis server is running on `localhost` (run `redis-server`).
In a development environment, you can start the application by running:

```sh
$ npm run dev
```
This will start the node server on port `8123` and start watching `.less` files for changes.

For production environments, use `node server`.
To specify a custom port number, use the `-p` flag when calling `node server`, e.g., to run the application on port `8080`:

```sh
$ node server -p 8080
```

Additionally, you can start the application as a daemon by passing `-d` flag, which starts the application and keeps it running in the background so that your terminal is not blocked, e.g.:

```sh
$ node server -d
```

## Configuration options

Configuration can be done using the config JSON file, or environment variables which overwrite any JSON value read. The default JSON file is `config/config.json`. This path can be overwritten using the Environment Variable `DEFAULT_JSON_CONFIG_PATH`.
These are the detailed configuration options:

### OpenTok configuration

Environment Variable Names and Description:
- `TB_API_KEY` (Required): Your OpenTok API key.
- `TB_API_SECRET` (Required): Your OpenTok API Secret.
- `TB_MAX_SESSION_AGE` (Optional, default value 2):  Sessions should not live forever. So we'll store
   the last time a session was used and if when we fetch it from Redis we determine it's older than
   this max age (in days). This is the key where that value (in days) should be stored.
   By default, sessions live two days.

JSON example:
```js
"OpenTok": {
	"apiKey": "<key>",
	"apiSecret": "<secret>",
	"maxSessionAge": 2,
},
   ```


### Firebase configuration

- `FB_DATA_URL`: Firebase data URL. This should be the root of the archives section of your Firebase app URL, which isn't necessarily the root of the app.
- `FB_AUTH_SECRET`: Firebase secret to generate auth tokens.

```js
"Firebase": {
    "dataUrl": "<appurl>",
    "authSecret": "<appsecret>"
}
```
### Web client configuration

Web client allows to be configured in some of its features. You can enable or disable using their `enabled` field in JSON or `ENABLE_<FEATURE>` environment variable.

#### Archiving
- `ENABLE_ARCHIVING`:(Optional, default value: true) Enable Archiving (Recording)
- `ARCHIVE_ALWAYS`:(Optional, default value: false) Record all sessions.
- `ARCHIVE_TIMEOUT`: (Optional, default value: 5000): The initial polling timeout (in milliseconds) for archive status change updates. Set this to 0 to disable polling.
- `TIMEOUT_MULTIPLIER` (Optional, default value: 1.5) : Timeout multiplier. If the first archive status update polling fails, subsequent polling intervals will apply this multiplier
   successively. Set to a lower number to poll more often.

##### Archive Manager
- `ENABLE_ARCHIVE_MANAGER`: (Optional, default value: false) Enable Archive Manager. Only meaningful if `archiving` is not disabled (Manage Recordings, requires firebase to be configured)
- `EMPTY_ROOM_LIFETIME`: (Optional, default value 3): Maximum time, in minutes,  an empty room

```js
"Archiving": {
    "enabled": true,
    "archiveAlways": false,
    "pollingInitialTimeout": 5000,
    "pollingTimeoutMultiplier": 1.5,
    "archiveManager": {
        "enabled": false,
        "emptyRoomMaxLifetime": 3
    }
},
```

#### Screensharing
- `ENABLE_SCREENSHARING`:(Optional, default value: false) Enable Screen sharing.
- `CHROME_EXTENSION_ID` (Optional, default value: 'null'): Chrome AddOn extension ID for screen sharing. Note that while the default value allows the server to run, doesn't actually enable screen sharing in Chrome. See [Screen sharing](#screen-sharing).
- `ENABLE_ANNOTATIONS`: (Optional, default value: true) Enable Annotations in Screen Sharing. Only meaningful if `screensharing` is not disabled.

```js
"Screensharing": {
    "enabled": false,
    "chromeExtensionId": null,
    "annotations": {
        "enabled": true
    }
}
```
#### Feedback
 `ENABLE_FEEDBACK`: Enable the "Give Demo Feedback" form.
 ```js
 "Feedback": {
     "enabled": false
 },
 ```

### Additional configuration options


* `ALLOW_IFRAMING` (Optional, default value: 'never'): Controls the server-side restriction on
   allowing content to load inside an iframe. The allowed values are:

   - 'always': Allow iframing unconditionally (note that rtcApp.js should also be changed
     to reflect this, this option only changes what the server allows)

   - 'never': Set X-Frame-Options to 'DENY' (Deny loading content in any iframe)

   - 'sameorigin': Set X-Frame-Options to 'SAMEORIGIN' (Only allow iframe content to be loaded
     from pages in the same origin)

   We don't allow restricting iframe loading to specific URIs because it doesn't work on Chrome

### Firebase security measure

If you want to ensure that the archive list is kept secure (as in only the actual people using a room can see it, and nobody can see the list of archives of other rooms) then you will need to configure additional security parameters to your Firebase application. To do this, log in to Firebase and set this security rule in the "Security & Rules" section:


```js
{
    "rules": {
        ".read": false,
        ".write": false,
        "sessions": {
          ".read": "auth != null && auth.role == 'server'",
          ".write": "auth != null && auth.role == 'server'",
          "$sessionId": {
            ".read": "auth != null && (auth.role == 'server' || auth.sessionId == $sessionId)",
            ".write": "auth != null && auth.role == 'server'",
            "archives": {
            },
            "connections": {
              ".read": "auth != null && auth.role == 'server'",
              ".write": "auth != null && (auth.role == 'server' || auth.sessionId == $sessionId)",
              "$connectionId": {
              }
            }
          }
        }
    }
}
```

Replace 'sessions' with the root where you want to store the archive data (the actual URL that you set as `fb_data_url` configuration parameter.


## Screen sharing

The screen-sharing-extension-chrome directory includes sample files for developing a
Chrome extension for enabling screen-sharing for this app. See the
[OpenTok screen sharing developer guide](https://tokbox.com/developer/guides/screen-sharing/js/)
for more information.

Follow these steps to use the chrome extension included in this repository.

1. Edit the `manifest.json` file:

    * Set the `matches` property to match only your web domains. (When developing in
      the localhost environment, you can use ```"matches": ["http://localhost/*"]```).

    * Change the `name` and `author` settings

    * Replace the icon files (logo16.png, logo48.png, logo128.png, and logo128transparent.png)
    with your own website logos.

    * Change the `version` setting with each new version of your extension.

    * You may want to change the `description`.

    For more information, see the [Chrome extension manifest
    documentation](https://developer.chrome.com/extensions/manifest).

2. Load the extension into Chrome:

    Open [chrome://extensions](chrome://extensions) and drag the screen-sharing-extension-chrome
    directory onto the page, or click 'Load unpacked extension...'. For more information see
    [Chrome's documentation on loading unpacked
    extensions](https://developer.chrome.com/extensions/getstarted#unpacked).

3. Add the `extensionId` to application configuration:

   You can get the ID of the extension in the [chrome://extensions](chrome://extensions) page.
   (It looks like `ffngmcfincpecmcgfdpacbdbdlfeeokh`).
   Add the value to the configuration, see [Configuration Options: Screen sharing](#Screensharing)


For more information and how to use your extension in production see the documentation at the
[opentok/screensharing-extensions](https://github.com/opentok/screensharing-extensions/blob/master/chrome/ScreenSharing/README.md#customizing-the-extension-for-your-website)
repo on GitHub.

## Troubleshooting

**"ServerPersistence: Timeout while connecting to the Persistence Provider! Is Redis running?**

Ensure Redis server is running on localhost (run `redis-server` in the command line)
and restart OpenTokRTC.

*** OpenTokRTC does not work on when served over HTTP.***

Browser security policies require HTTPS for WebRTC video communications. You will need to set up
the app to be served over HTTPS. You can set up a
[secure reverse-proxy](https://www.nginx.com/resources/admin-guide/nginx-https-upstreams/)
to your OpenTokRTC port using nginx. For details, read
[this post](https://tokbox.com/blog/the-impact-of-googles-new-chrome-security-policy-on-webrtc/).
