# DirectX desktop client sample

## Building the client
+ Follow all steps in [How to build the toolkit](https://3dstreamingtoolkit.github.io/docs-3dstk/#how-to-build-the-toolkit)
+ Open the 3DStreamingToolkit solution in Visual Studio
+ Build the project `StreamingDirectxClient` in the desired configuration (Build -> Configuration Manager -> Dropdowns at the top). We encourage using `Release` and `x64`.

## Controls
+ To pan: Use left click and drag
+ To move: Use `W` `A` `S` `D` keys for forward, left, back and right. 

## Configuration

3DSTK’s sample client applications make use of an external JSON configuration file (webrtcConfig.json) to manage the connections to the Signaling and TURN services. Below is an example `webrtcConfig.json` file. This file contains placeholders for server addresses from your server setup in step 1 and 2 of [Getting Started](https://3dstreamingtoolkit.github.io/docs-3dstk/#getting-started). This file is found in the executable directory.
```
{
  "iceConfiguration": "relay",
  "turnServer": {
    "uri": "turn:<url>:5349",
    "username": "<username>",
    "password": "<password>"
  },
  "server": "<url>",
  "port": 80
  "heartbeat": 5000
}
```

## How it works

Once the config settings are changed, the client will require a server to connect to. In this toolkit, we have a few [Sample Servers](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Server). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

You can initiate the connection using the client or server, simply select the peer from the list and press join. That will start the video streaming and you can use the above controls to move the camera around the scene.
