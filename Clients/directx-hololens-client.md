# DirectX HoloLens client sample

## Hologram stability
To achieve stable holograms, HoloLens performs a sophisticated hardware-assisted holographic stabilization technique. This is largely automatic and has to do with motion and change of the point of view (CameraPose) as the scene animates and the user moves their head. A single plane, called the stabilization plane, is chosen to maximize this stabilization. While all holograms in the scene receive some stabilization, holograms in the stabilization plane receive the maximum hardware stabilization. More info about it [here](https://docs.microsoft.com/en-us/windows/mixed-reality/hologram-stability).

## Swap chain and Present
With Windows Mixed Reality, the system controls the swap chain. The system then manages presenting frames to each holographic camera to ensure a high quality user experience. It also provides a viewport update each frame, for each camera, to optimize aspects of the system such as image stabilization or Mixed Reality Capture. So, a holographic app using DirectX doesn't call `Present` on a DXGI swap chain. Instead, you use the [HolographicFrame](https://docs.microsoft.com/en-us/uwp/api/Windows.Graphics.Holographic.HolographicFrame) class to present all swapchains for a frame once you're done drawing it.

From **DeviceResources::Present**
```
HolographicFramePresentResult presentResult = frame->PresentUsingCurrentPrediction();
HolographicFramePrediction^ prediction = frame->CurrentPrediction;
```

The image stabilization technique actually kicks when calling [HolographicFrame::PresentUsingCurrentPrediction](https://docs.microsoft.com/en-us/uwp/api/windows.graphics.holographic.holographicframe.presentusingcurrentprediction) method.

## Remote rendering with image stabilization
A normal rendering loop would look like:
1. Create a new holographic frame.
2. From its current prediction, get view and projection matrices for the left/right cameras (these values will change with every frame).
3. Send the above matrices to vertex shader for mvp matrix calculation.
4. Render scene to the camera back buffer.
5. Present frame (the frame being presented was created in step 1).

For remote rendering scenario, the rendering loop involves both HoloLens and the rendering machine at the same time:
1. **[HoloLens]** Create a new holographic frame.
2. **[HoloLens]** From its current prediction, get view and projection matrices for the left/right cameras (these values will change with every frame).
3. **[HoloLens]** Send the view and projection matrices to the rendering machine.
4. **[Rendering machine]** Receive the view and projection matrices from the HoloLens.
5. **[Rendering machine]** Send the above matrices to vertex shader for mvp matrix calculation.
6. **[Rendering machine]** Render scene to texture (this is a side-by-side texture for stereo rendering).
7. **[Rendering machine]** Send the above texture to the HoloLens.
8. **[HoloLens]** Receive the texture data from the rendering machine.
9. **[HoloLens]** Use [orthographic projection](https://en.wikipedia.org/wiki/Orthographic_projection) in vertex shader since the received texture is a flat image.
10. **[HoloLens]** Render the received texture to the camera back buffer.
11. **[HoloLens]** Present frame (the frame being presented was created in step 1).

In order to reduce latency and increase performance, the whole process needs to perform smoothly. HoloLens and the rendering machine shouldn't block each other in the rendering loop.

Each holographic frame has a [PerceptionTimestamp](https://docs.microsoft.com/en-us/uwp/api/windows.perception.perceptiontimestamp), which affects how the frame is presented with prediction. Using this value, we can create more than one holographic frame and store them in a list. By doing this, the latest info of view and projection matrices are always being sent to the rendering machine and there is no need to wait for one frame to finish rendering.

On the rendering machine, the scene is rendered every time there is a new input from HoloLens and then the texture data is sent to the HoloLens. On the HoloLens side, after receiving the texture data including the perception timestamp, we can find the corresponding holographic frame in the list and render.

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

With the Hololens client, you can only initiate the connection using the server, simply select the peer from the list and press join. That will start the video streaming and you can move around the room to explore the scene. 
