## Building WebRTC Libraries from Source (You don't need to do this unless you want to change the underlying native or UWP WebRTC library)

> Note: Make sure you install our [prerequisites](https://github.com/3DStreamingToolkit/3DStreamingToolkit#prerequisites)  

In order to build the WebRTC library with the 3DStreamingToolkit extensions, you need to clone the merged native and UWP library: https://github.com/3DStreamingToolkit/webrtc-extensions-3dstk 

Follow the instructions in the README to build the native and UWP libraries. 

Once finished building, for local use, copy the contents of 
```
C:\<path to source>\dist
```
to the `WebRTCLibs` folder in this repo.
