# Unity HoloLens client sample

## Building HoloLens Unity client 

 > Note: Make sure you have Unity 2017.4.4f1 LTS release with UWP tools support. 

These steps are unique, and specific to producing a unity client application for a HoloLens device.

You can find the HoloLens Unity client sample in `\Samples\Client\Unity`. The HoloLens client is using a native dll library for video decoding and rendering that needs to be built before you open the sample in Unity. Follow these steps:
+ Open `3DStreamingToolKit.sln` from the root folder 
+ Switch the build configuration to Release mode and x86. 
+ Under `Plugins -> UnityClient`, find MediaEngineUWP and WebRtcWrapper projects.
+ Build MediaEngineUWP and WebRtcWrapper. This will copy the necessary dll's to the Unity project folder. 
+ Open Unity and open the HoloLens client folder `\Samples\Client\Unity`
+ Under scenes folder, open 'StereoSideBySideTexture' scene.
+ Under StreamingAssets, you will find 'webrtcConfig.json'. Modify the values to match your signaling server and TURN server (optional, needed for VPN/Proxy networks)
+ In the scene hierarchy, open the Control gameobject and look at the Control Script. 
    - Texture Width and Height: These settings are used to display the stereoscopic texture from the server. The client is expecting a left and right eye side-by-side texture and it will crop it for each eye. Since our servers are sending 720p quality, the default values are 2560 x 720 (1280 x 2 width for each eye). Change this if you are sending different resolutions.
    - Frame Rate: This is the expected frame rate from the server. The native engine will be optimized to maintain this framerate. 30 is the currently recommended value, going above this, can cause loss of quality and corrupt frames. 
 > Note: Make sure you set the same frame rate on the server side!
+ Go to `File -> Build Settings` and select Windows Store platform. If that is not available, you need to add UWP support to your Unity installation. 
+ Select 
    - SDK: Universal 10
    - Target Device: HoloLens
    - UWP Build type: D3D
    - UWP SDK: 10.0.14393
+ Build the project to an empty folder and open the generated UnityX86.sln.
+ Switch configuration to Release mode and x86. 
+ Deploy to your machine or HoloLens. Make sure the server is running, the app will automatically connect to the ip address that was set in the step above.

## How it works

Once the config settings are changed, the client will require a server to connect to. In this toolkit, we have a few [Sample Servers](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Server). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

With the Hololens client, you can only initiate the connection using the server, simply select the peer from the list and press join. That will start the video streaming and you can move around the room to explore the scene. 
