The nature of a 3DStreamingToolkit deployment implies a complex authentication setup. This page captures the data needed to understand what components are in play, and how to configure them. It is based off of [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/177). 🔐 

![Authentication Architecture Overview](https://user-images.githubusercontent.com/1167891/39771759-48355a84-52c1-11e8-98fd-dd0198aac0ed.png)

## Prework:

1. [Create an AAD Tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-howto-tenant)
2. [Create an AAD B2C Application](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-app-registration) (used for client login)
3. [Create a Normal AAD Application](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal) (used for server login)
4. Record tenant name and app id's for use below
5. (optional) setup [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) if you need to support advanced NAT traversal scenarios. We have an article about how we did this [here](https://www.microsoft.com/developerblog/2018/01/29/orchestrating-turn-servers-cloud-deployment)
6. (optional) record TURN postgres credential information

## Setup Signaling Authentication:

> This secures transmission of [sdp](https://webrtchacks.com/sdp-anatomy/) messages, which are the session establishment messages. It's how our clients find servers, and vice versa.

1. deploy [signaling](https://github.com/3DStreamingToolkit/signal-3dstk) to azure. This is simple, just click the deploy button. 
2. in [App Settings](https://docs.microsoft.com/en-us/azure/app-service/web-sites-configure) set the following (note: these are defined in [the readme](https://github.com/3DStreamingToolkit/signal-3dstk/blob/master/README.md)):
    + `AAD_TENANT_ID` your aad tenant info. ex: `3dtoolkit.onmicrosoft.com`
    + `AAD_B2C_CLIENT_APPLICATION_ID` your add client app id. ex: `aacf1b7a-104c-4efe-9ca7-9f4916d6b66a`
    + `AAD_B2C_POLICY_NAME` your aad b2c policy name. ex: `b2c_1_signup` 
    + `AAD_RENDERING_SERVER_APPLICATION_ID` your aad server app id. ex: `5b4df04b-e3bb-4710-92ca-e875d38171b3`

## Setup Identity Management:

> This secures [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) such that short lived temporary credentials are used, and further ensures that only AAD authenticated users can retrieve said credentials. __If you aren't using TURN in production, you don't need this__.

1. deploy [identity management](https://github.com/anastasiia-zolochevska/3dtoolkit-identity-management) to azure.
2. in [App Settings](https://docs.microsoft.com/en-us/azure/app-service/web-sites-configure) set the following:
    + `PGUSER` your TURN postgres username
    + `PGDATABASE` your TURN postgres database name
    + `PGPASSWORD` your TURN postgres password
    + `PGHOST` your TURN postgres hostname/fqdn
    + `PGPORT` your TURN postgres port
    + `AAD_TENANT_ID` your aad tenant info
    + `AAD_B2C_CLIENT_APPLICATION_ID` your add client app id
    + `AAD_B2C_POLICY_NAME` your aad b2c policy name
    + `AAD_RENDERING_SERVER_APPLICATION_ID` your aad server app id
    + `AAD_B2C_CLIENT_APPLICATION_SECRET` your aad client app secret
    + `AAD_B2C_REDIRECT_URI` your aad client app redirect uri (should be identity management service site with a specific path. ex: `<siteName>.azurewebsites.net/device/login`)

## Setup OAuth24D:

> This hands out credentials to clients, using short-lived device tokens and a user device to login. We have an article about how we did this, and why it exists [here](https://www.microsoft.com/developerblog/2018/04/23/oauth-2-0-mixed-reality-applications/).

1. modify [app.js](https://github.com/bengreenier/oauth24d/blob/master/examples/server/node/app.js) to use a real [passportjs](http://www.passportjs.org/) strategy, likely [passport-azure-ad](https://github.com/azuread/passport-azure-ad) (__no public example yet, see https://github.com/bengreenier/oauth24d/issues/2__).
2. deploy the modified [oauth24d](https://github.com/bengreenier/oauth24d/tree/master/examples/server/node) to azure.
3. in [App Settings]() set the following:
    + `AUTH_URI` the user facing authentication url (should be oauth24d service site with a specific path. ex: `<siteName>.azurewebsites.net/login`)
    + Optional settings exist, please see [the readme](https://github.com/bengreenier/oauth24d/blob/master/examples/server/node/README.md) for more info

## Configuration:

> See [webrtcConfig](./webrtc-config.md) for more info

For the client, we need to point it to the oauth24d service. This requires two urls in the `authentication` config section in `webrtcConfig.json`:

```
"authentication": {
    "codeUri": "<siteName>.azurewebsites.net/new",
    "pollUri": "<siteName>.azurewebsites.net/poll"
}
```

This tells the client to use the oauth24d authentication flow to direct a user to a uri to login, and give the application a token it can use to authenticate to the other services.

For the server, we need to point it to azure ad. This requires a few values in the `authentication` config section in `webrtcConfig.json`:

```
"authentication": {
    "authority": "<an oauth2 authority uri, tested against aadv1>",
    "resource": "<the oauth2 resource, commonly the same as clientId>",
    "clientId": "<the oauth2 authority application id>",
    "clientSecret": "<the oauth2 authority application secret>"
}
```

These are the same values as observed in setup above, with one caveat - authority maps to `https://login.microsoftonline.com/<AAD_TENANT_ID>.onmicrosoft.com/oauth2/token`. For instance, `https://login.microsoftonline.com/3dtoolkit.onmicrosoft.com/oauth2/token`. This tells the server to use client_credentials aad grant to get a token it can use to authenticate to other services.

## Realtime data:

Realtime data channel messaging is encrypted using DTLS [by default](https://github.com/3DStreamingToolkit/3DStreamingToolkit/blob/master/Plugins/NativeServerPlugin/src/peer_conductor.cpp#L195). Please note that if you're using TURN, data will be decrypted on the TURN node, while it is handed off to the other peer (effectively exposing [MITM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) potential on that node). Realtime video channel communications are encrypted using SRTP [by default](https://github.com/3DStreamingToolkit/3DStreamingToolkit/blob/master/Plugins/NativeServerPlugin/src/peer_conductor.cpp#L195). You can learn more [here](http://webrtc-security.github.io/#4.3.).

## Resources

* [signaling server](https://github.com/3DStreamingToolkit/signal-3dstk)
* [legacy signaling server](https://github.com/anastasiia-zolochevska/signaling-server-http-auth)
* [TURN configuration walkthrough](https://www.microsoft.com/developerblog/2018/01/29/orchestrating-turn-servers-cloud-deployment)
* [Identity management server](https://github.com/anastasiia-zolochevska/3dtoolkit-identity-management)
* [oauth24d spec](https://github.com/bengreenier/oauth24d)
* [oauth24d example](https://github.com/bengreenier/oauth24d/tree/master/examples/server/node)