> Note: if you don't see the answer here, **please search [our issues](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues) for your question**, before [opening a new one](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/new).

The page contains some commonly asked questions, failures, and quick solutions.

## Build Issues

### "uuid" attribute not found
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/75).

Make sure to set your build to `x86` and `Release`. This is specific to the MediaEngineUWP native component.

### Unity client build failure
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/64).

**Problem:** The unity client fails to build, giving the following error:

```
Assets\Scripts\ControlScript.cs(336,50): error CS1061: 'Media' does not contain a definition for 'CreateMediaStreamSource' and no extension method 'CreateMediaStreamSource' accepting a first argument of type 'Media' could be found (are you missing a using directive or an assembly reference?)
```

This seems to occur if you have an old version of the `Org.WebRtc` library.

To fix:

> Note: this simply removes the library and runs our installation script to reinstall it.

```powershell
cd Libraries
rd /Q /S WebRTCUWP
git checkout -- WebRTCUWP
powershell -File .\WebRTCUWP\InstallLibraries.ps1
```

You will need to rebuild the `Plugins\UnityClient\WebRTCWraper` project in `x86/Release` mode after running the above steps.

You may need to close and reopen unity afterward.

### Can't build StreamingDirectXHololensClient
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/69).

**Problem**: You're getting the following error:
```
3>C:\Program Files (x86)\MSBuild\Microsoft\.NetNative\x86\ilc\IlcInternals.targets(318,5): error : Error: Windows Runtime metadata is invalid in Windows SDK, follow the steps in http://go.microsoft.com/fwlink/?LinkId=733341 to repair your installation.
```

To resolve this problem, you need to repair the Windows SDK and tools for Windows 10, version 1511.
Steps to repair:

1. Start -> Control Panel
2. Under Programs choose “uninstall a program”.
3. This will present a dialog with all the applications installed on your Windows operating system. In the upper right hand corner type “kit”. This will filter the list of installed programs. One of the filtered applications will be “Windows Software Development Kit – Windows 10.0.14393.795”
4. Single click on Windows Software Development Kit – Windows 10.0.14393.795, and choose “change”.
5. This will present the Windows Software Development Kit setup dialog and there is an option to “repair”. Select repair and click Next.
6. This will repair the installation of your Windows SDK.

If you've reached this issue and you **don't have the SDK at all** you can get it here - you need the `10.0.14393.795` version.

## App Issues

### StreamingDirectxClient.exe showing white screen on server connect
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/78).

This can be addressed by removing the nvpipe.dll from the build folder. It will force WebRTC to use software encoder. 

### Why do server samples running on MacOS crash when trying to connect
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/80).

The macbook audio driver is not supported by the version of webrtc our toolkit currently targets. The easiest workaround is to disable the audio devices.

### Unity server sample fails to connect
Based on [this issue](https://github.com/3DStreamingToolkit/3DStreamingToolkit/issues/82).

It turns out that Unity 64-bit doesn't like `x86 dll` so it fails to load `StreamingUnityServerPlugin.dll`. The solution is rebuilding the plugin (`StreamingUnityServerPlugin.sln`) as **x64** and change the build settings in Unity editor to **x86_64**.