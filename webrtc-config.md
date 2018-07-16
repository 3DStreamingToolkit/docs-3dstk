Use [webrtcConfig.json](https://github.com/3DStreamingToolkit/3DStreamingToolkit/blob/master/Plugins/NativeServerPlugin/webrtcConfig.json) for TURN relay communication

# WebRTC Configuration (webrtcConfig.json)

3DStreamingToolkit's sample server and client applications make use of an external JSON configuration file (webrtcConfig.json) to manage the connections to the Signaling and TURN services.  Below is an example webrtcConfig.json file.  This file contains placeholders for server addresses and login credentials from your server setup.  This file should be placed in your executable directory.

    {
      "iceConfiguration": "relay",
      "turnServer": {
        "uri": "turn:<url>:5349",
        "username": "<username>",
        "password": "<password>"
      },
      "stunServer": {
        "uri": "<url>:<port>"
      },
      "server": "<url>",
      "port": 80
      "heartbeat": 5000
    }

## Detailed values

### iceConfiguration

> Supported values: `relay`, `stun`, `none`, `not defined`

Indicates the [ICE](https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment) configuration. `relay` indicates that we should use [TURN](https://tools.ietf.org/html/rfc5766), `stun` indicates we should use [STUN](https://en.wikipedia.org/wiki/STUN). `none` and `not defined` indicate we should use [STUN](https://en.wikipedia.org/wiki/STUN) with Google's public stun servers.

### turnServer

> Supported values: `json object`, `not defined`

Only valid when `iceConfiguration` is `relay`. Defines the [TURN](https://tools.ietf.org/html/rfc5766) configuration values for connection establishment. 

__These values can be transmitted from the server (and therefore not needed on the client) in some configurations. See [this post](https://github.com/3DStreamingToolkit/3DStreamingToolkit/wiki/Transmitting-TURN-credentials) for more info.__

#### uri

> Supported values: `string, uri`

The uri that we will communicate with using [TURN](https://tools.ietf.org/html/rfc5766) to exchange data.

#### username

> Supported values: `string`

Provides a username to use when authenticating to the [TURN](https://tools.ietf.org/html/rfc5766) server.

#### password

> Supported values: `string`

Provides a password to use when authenticating to the [TURN](https://tools.ietf.org/html/rfc5766) server.

### stunServer

> Supported values: `json object`, `not defined`

Only valid when `iceConfiguration` is `stun`. Defines the [STUN](https://en.wikipedia.org/wiki/STUN) configuration values for connection establishment.

#### uri

> Supported values: `string, uri`

The uri that we will communicate with to request [STUN](https://en.wikipedia.org/wiki/STUN) information for connection establishment.

### serverUri

> Supported values: `string, uri`

Indicates the signalling server (see our implementation [here](https://github.com/3DStreamingToolkit/signal-3dstk)) uri that we will connect to, and use for signalling.

### port

> Supported values: `int, port`

Indicates the signalling server (see our implementation [here](https://github.com/3DStreamingToolkit/signal-3dstk)) port that we will connect to, and use for signalling.

### heartbeat

> Supported values: `int, time in ms`

A value of `0` disables this. Indicates the interval at which a `HTTP GET /heartbeat` request will be sent to the signalling server throughout the duration of the session. This helps the signalling server determine when connections have gone stale, or failed.

## WebRTCConfig Extensions

These additional snippets of configuration enable features, and can be appended to the above configuration to configure and leverage these features.

### Client authentication

> Note: this config is only valid in client implementations.

```
"authentication": {
    "codeUri": "<url to an oauth24d code provider>",
    "pollUri": "<url to an oauth24d code provider polling endpoint>"
}
```

This configuration is only valid for client implementations, as it requires interactive login on behalf of a user. An OAuth24D provider is any service that implements the [oauth24d](https://github.com/bengreenier/oauth24d) RESTful api, much like https://developers.google.com/identity/protocols/OAuth2ForDevices.

### Server authentication

> Note: this config is only valid in server implementations.

```
"authentication": {
    "authorityUri": "<an oauth2 authority uri, tested against aadv1>",
    "resource": "<the oauth2 resource, commonly the same as clientId>",
    "clientId": "<the oauth2 authority application id>",
    "clientSecret": "<the oauth2 authority application secret>"
}
```

This configuration is only valid for server implementations, as it does not require interactive login, and is authenticated on behalf of an application, not a user. Our server authentication mechanism is effectively an aadv1 application, like https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service

### Temporary turn credentials

```
"turnServer": {
    "provider":"<url to an endpoint that serves json containing a username/password>"
}
```
This configuration enables temporary turn credential retrieval (and has a dependency on authentication, and therefore the authentication config values). The expectation is that such a provider endpoint should be secured.

# Server Configuration (serverConfig.json)

3DStreamingToolkit's sample server applications have server specific configuration that is stored in an external JSON configuration file (serverConfig.json). It can be used to configure encoding dimensions, system service status, system capacity, and system service installation settings.

```
{
  "serverConfig": {
    "width": 1280,
    "height": 720,
    "systemService": false,
    "systemCapacity": -1,
    "autoCall": false,
    "autoConnect": false
  },
  "serviceConfig": {
    "name": "3DStreamingRenderingService",
    "displayName": "3D Streaming Rendering Service",
    "serviceAccount": "NT AUTHORITY\\NetworkService",
    "servicePassword":  null
  }
}
```

### systemCapacity

Controls [multi-peer](./multi-peer.md) capacity.

### autoCall

When set to `true`, the server will auto call any peer that has signed in, connecting the session.

### autoConnect

When set to `true`, the server will auto connect to the signaling server.

# NVEncode Configuration (nvEncConfig.json)

3DStreamingToolkit's sample server applications make use of an external JSON configuration file (nvEncConfig.json) to manage the settings used for video encoding.  Below is an example nvEncConfig.json file. This file should be placed in your executable directory.
```
{
  /* Set the desired framerate for the renderer and the encoder. */
  "serverFrameCaptureFPS": 60,
  "NvencodeSettings": {
    /* Setup the average and min bitrate for the encoder.
    * WebRTC will modify the bitrate based on bandwidth but will never go below minBitrate.
    * Use the Kush gauge for best quality: width * height * framerate * 4 * 0.07 
    * Our mono samples are 1280x720 and stereo 2560x720 
    * WARNING: Setting a high minBitrate can cause latency. */
    "bitrate": 7741440,
    "minBitrate": 3870720,
    /* Setup an encoder that puts an IDR every 60 frames and the rest P-frames. 
    * Set intraRefreshEnableFlag to enable/disable I-frames. 
    * If flag is true, intraRefreshPeriod puts an I-frame every (n) number of frames. */
    "idrPeriod": 60,
    "intraRefreshPeriod": 30,
    "intraRefreshEnableFlag": false
  }
}
```