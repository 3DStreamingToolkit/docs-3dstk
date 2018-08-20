# DirectX spinning cube server sample
The DirectX spinning cube sample is a server sample built in DirectX that supports mono and stereo rendering of a simple 3D cube.

## Native Plugin

DirectX spinning cube consumes a native plugin that is produced by the 3D Streaming Toolkit build pipeline. As such it's critical to first build 3D Streaming Toolkit solution before attempting to use the sample (see [Building](#building) below). 

The native plugin is responsible for negotiating with clients to configure a stream, and for encoding and sending visual frame data from DirectX to a connected client. All of our code is designed to abstract and facilitate communication with this native module.

## Building

+ Follow all steps in [How to build the toolkit](https://3dstreamingtoolkit.github.io/docs-3dstk/#how-to-build-the-toolkit)
+ Open the 3DStreamingToolkit solution in Visual Studio
+ Build the project `SpinningCubeServer` in the desired configuration (Build -> Configuration Manager -> Dropdowns at the top). We encourage using `Release` and `x64`.

## Configuration

3DSTK’s sample server applications make use of an external JSON configuration file (webrtcConfig.json) to manage the connections to the Signaling and TURN services. Below is an example `webrtcConfig.json` file. This file contains placeholders for server addresses from your server setup in step 1 and 2 of [Getting Started](https://3dstreamingtoolkit.github.io/docs-3dstk/#getting-started). This file is found in the executable directory.
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

Once the config settings are changed, the server will require a client to connect to. In this toolkit, we have a few [Sample Clients](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Client). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

You can initiate the connection using the client or server, simply select the peer from the list and press join. That will start the video streaming and you can use the controls for that specific client to interact with the scene. 


