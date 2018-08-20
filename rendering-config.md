# Rendering Service Configuration

[DirectX Spinning Cube Server](./Servers/directx-spinning-cube-server.md), [DirectX Multithreaded Server](./Servers/directx-multithreaded-server.md) and [OpenGL Spinning Cube Server](./Servers/opengl-spinning-cube-server.md) can be configured to run as a rendering service (headless).

### Modify serverConfig.json

To run the app as system service, `systemService` flag must be set to `true`.

### Powershell scripts

Open powershell in *Administrator* mode, run:

* `serviceRegister.ps1` script to register the app as system service.
* `serviceDelete.ps1` script to remove the registered service.
* `serviceStart.ps1` script to start the registered service.
* `serviceStop.ps1` script to stop the running service.