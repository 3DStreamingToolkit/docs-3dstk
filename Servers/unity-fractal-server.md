# Unity fractal server sample
The Unity-FractalStream sample is a server sample built in Unity that supports mono and stereo rendering.

### Native Plugin

Unity-FractalStream consumes a native plugin that is produced by the 3D Streaming Toolkit build pipeline. As such it's critical to first build 3dtoolkit before attempting to use the sample (see [Building](#building) below). 

The native plugin is responsible for negotiating with clients to configure a stream, and for encoding and sending visual frame data from Unity to a connected client. All of our scripting in Unity is designed to abstract and facilitate communication with this native module.

### MutableState

Mutable state is just what it sounds like - a lightweight abstraction around mutable data that is used to represent data surfaced from the native plugin within Unity. The component that powers this is `PeerListState`. `PeerListState` is a simple `ScriptableObject` that represents a collection of peers (potential clients) as well as an optional currently selected peer. To create an instance of a `PeerListState`, you simply navigate to `Assets -> Create -> PeerListState`. This will add an asset to the project that represents a particular mutable state. The instance can be wired to scripts (like `WebRTCServer`) in the editor by dragging it to a field.

## Menus

Unity-FractalStream ships with a basic menu implementation that allows a user to interactively select a peer to connect to. This menu is powered by [Unity-MenuStack](https://github.com/bengreenier/unity-menustack) which provides a framework for quickly building out menu systems using Unity's built in [UI System](https://docs.unity3d.com/Manual/UISystem.html). Combined with two scripts (`PeerListDropdownAdapter` and `DropdownSelectorButton`) we're able to build a UI that interacts with a `PeerListState` instance.

### Prefabs

Unity-FractalStream ships with two prefabs that enable quick buildouts of WebRTC capable scenes. `StereoCapableCameraHead` represents a mono/stereo capable camera system with a `WebRTCServer` component that enables frame encoding and connectivity. Use this component instead of a traditional main `Camera` to power your scene rendering. `MenuRoot` represents the menu system that enables interactive selection of peers. Use this component if your scene should allow the user to connect to peers, rather than waiting for a peer to connect to it.

### Scripts

There are a handful of Unity scripts that enable our sample experience. We'll briefly cover the roles and responsibilities of each, below.

#### Dependencies

+ `SimpleJSON` enables us to parse JSON data effectively
+ All scripts under `UnityPackages\MenuStack` are part of the [Unity-MenuStack](https://github.com/bengreenier/unity-menustack) framework

#### Editor

+ `ConfigureCopyBuildTask` attempts to copy [config files](../webrtc-config.md) from the `Plugins` directory to the output directory. This can save development time, as you won't need to generate configuration files manually in the output directory, but it is optional.

#### Core

+ `StreamingUnityServerPlugin` provides a wrapper around the native plugin that powers the experience. An instance of this wrapper is created by `WebRTCServer`, and exposed publicly. Use this instance to listen to native events, connect to peers manually, encode frame data, and disconnect from peers.
+ `WebRTCServer` is the main WebRTC component that enables the experience. It configures the native plugin, handles client input data, and generally drives the experience.
+ `WebRTCServerDebug` is a development component that enables detailed logging data to be enabled via additional command line flags (for details, see [the inline comments](https://github.com/CatalystCode/3dtoolkit/blob/master/Samples/Server/Unity-FractalStream/Assets/Scripts/WebRTCServerDebug.cs#L27))

#### Experience

> Note: if you're building a new experience, these can be ignored.

+ `Fractals` drives fractal generation, which is necessary for our sample experience of a spinning fractal shape.
+ `FractalRoot` represents the root of a fractal shape, which is necessary for generation.

## Building

> Note: Unity-FractalStream requires Unity 2017.4.4f1 LTS or higher (we test at 2017.4.4f1 LTS)

> Note: Unity-FractalStream requires that [the 3D Streaming Toolkit solution is built](https://3dstreamingtoolkit.github.io/docs-3dstk/#how-to-build-the-toolkit) first, to create the needed native plugins

+ Open [the project directory](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Server/Unity-FractalStream) in Unity
+ Navigate to `File -> Build Settings`
+ Ensure `Platform` is `PC, Mac & Linux Standalone`
+ Ensure `Architecture` matches the architecture that the 3DStreamingToolkit solution was last built with
+ Ensure `Scenes/StereoCapableFractalCubes` is listed under `Scenes In Build`
+ Build
+ Run with `-force-d3d11-no-singlethreaded` via the command line

## How it works

Once the config settings are changed, the server will require a client to connect to. In this toolkit, we have a few [Sample Clients](https://github.com/3DStreamingToolkit/3DStreamingToolkit/tree/master/Samples/Client). 

When both the client and server are connected to the same signaling server, they will appear in the peer list. 

You can initiate the connection using the client or server, simply select the peer from the list and press join. That will start the video streaming and you can use the controls for that specific client to interact with the scene. 


