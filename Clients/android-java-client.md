# Android Java client sample

An Android client written in Java.

## Building the client
+ Requires Android Studio 2.3.3
+ Edit the file [Config.java](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Client/AndroidClient/app/src/main/java/microsoft/a3dtoolkitandroid/util/Config.java) to enter your proper server configuration.
+ Build the project

## Controls
+ To pan: Use one finger
+ To move forward/backward: Keep one finger down and slide second finger up/down to move.
+ (TODO): Accelerometer control.

## Configuration

The current configuration values are as follows:

```
    public static String signalingServer = "http://localhost:3001";
    public static String turnServer = "turn:turn:turnserveruri:5349";
    public static String username = "user";
    public static String credential = "password";
    public static PeerConnection.TlsCertPolicy tlsCertPolicy = PeerConnection.TlsCertPolicy.TLS_CERT_POLICY_SECURE;
```

With the exception of `tlsCertPolicy` these map directly to [webrtcConfig](../webrtc-config.md) options, as follows:

+ `signalingServer` => [webrtcConfig#server](../webrtc-config.md#server) 
+ `turnServer` => [webrtcConfig#turnServer#uri](../webrtc-config.md#uri)
+ `username` => [webrtcConfig#turnServer#username](../webrtc-config.md#username)
+ `credential` => [webrtcConfig#turnServer#password](../webrtc-config.md#password)

uniquely for `tlsCertPolicy`, we allow the caller to configure how the underlying tls library will verify certificates. This supports
the following values:

+ `PeerConnection.TlsCertPolicy.TLS_CERT_POLICY_INSECURE_NO_CHECK` - disables certificate and chain of trust checks
+ `PeerConnection.TlsCertPolicy.TLS_CERT_POLICY_SECURE` - (default) ensures certificate integrity and chain of trust are valid

## How it works

Once the config settings are changed, the client will require a server to connect to. In this toolkit, we have a few [Sample Servers](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Server). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

You can initiate the connection using the client or server, simply select the peer from the list and press join. That will start the video streaming and you can use the above controls to move the camera around the scene.
