# iOS Swift Client Sample

An iOS client written in swift :iphone: :cloud:

## Building the client

> Note: If a linking error occurs, set the flag `Enable Bitcode` to `NO` under the build settings tab for the project.

+ Install the required WebRTC library by executing `pod install` from the directory containing the `Podfile`.
+ Edit the file [Config.swift](./SwiftClient/Config.swift) to enter your proper server configuration.
+ Build the project

## Controls
+ To pan: Use one finger or use accelerometer control (optional)
+ To move forward/backward: Keep one finger down and slide second finger up/down to move.

## Configuration

The current configuration values are as follows:

```
static let signalingServer = "YOUR SIGNALING SERVER"
static let turnServer = "YOUR TURN SERVER"
static let username = "YOUR USERNAME"
static let credential = "YOUR CREDENTIAL"
static let tlsCertPolicy = RTCTlsCertPolicy.secure
```

With the exception of `tlsCertPolicy` these map directly to [webrtcConfig](../webrtc-config.md) options, as follows:

+ `signalingServer` => [webrtcConfig#server](../webrtc-config.md#server) 
+ `turnServer` => [webrtcConfig#turnServer#uri](../webrtc-config.md#uri)
+ `username` => [webrtcConfig#turnServer#username](../webrtc-config.md#username)
+ `credential` => [webrtcConfig#turnServer#password](../webrtc-config.md#password)

uniquely for `tlsCertPolicy`, we allow the caller to configure how the underlying tls library will verify certificates. This supports
the following values:

+ `RTCTlsCertPolicy.insecureNoCheck` - disables certificate and chain of trust checks
+ `RTCTlsCertPolicy.secure` - (default) ensures certificate integrity and chain of trust are valid

## How it works

Once the config settings are changed, the client will require a server to connect to. In this toolkit, we have a few [Sample Servers](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Server). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

You can initiate the connection using the client or server, simply select the peer from the list and press join. That will start the video streaming and you can use the above controls to move the camera around the scene.
