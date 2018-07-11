This document refers to the necessary configuration changes that will enable users to transmit TURN credentials from a server to a client during connection. This removes the need for configured credentials on the client.

## How it works

When operating in this mode, the server will attach it's turn credentials into the initial session establishment message that is sent to the client. As this message is sent over the signaling channel it is crucial that you only rely on this feature when using `https` communication with your signaling server.

When enabled, we'll append the following information (as-per [this commit](https://github.com/3DStreamingToolkit/3DStreamingToolkit/commit/2045e63e80cc1277b900caf0ba36cfaa4562b077)) to the session establishment message:

```
{
    ...
    "uri": "turnServerUriValue",
    "username": "turnServerUsernameValue",
    "password": "turnServerPasswordValue"
}
```

The client will then be responsible for parsing this information, and configuring TURN.

## Supported clients

+ Win32 DirectX
+ Chrome
+ Firefox

For more details, see [the feature matrix](./feature-matrix.md).