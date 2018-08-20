# Running functional tests tutorial

This tutorial provides instructions on how to run the functional end-to-end tests on a rendering machine to validate the following:
* Compatible hardware and Nvidia drivers to run Nvencode
* Video capturer capability
* Low-latency encode/decode capability
* Correct connection to the signaling server
* Correct connection to the TURN server (if configured) 
* Correct end-to-end video streaming 
* End-to-end latency validation 

### 1. Use a local or VM rendering machine
The target machine must run Windows 10 or Windows Server 2016 and must have a compatible Nvidia GPU that [can run Nvencode](https://developer.nvidia.com/video-encode-decode-gpu-support-matrix). 

### 2. Prerequisites
* Download and install the [x64 Visual C++ Redistributable for Visual Studio 2015](https://download.microsoft.com/download/9/3/F/93FCF1E7-E6A4-478B-96E7-D4B285925B00/vc_redist.x64.exe)
* Download the [functional tests binary](https://github.com/CatalystCode/3DStreamingToolkit/releases/download/v2.0/3DStreamingToolkit-FunctionalTests-v2.0.zip) and unzip the content in any folder on the machine.
* Setup up a local or remote signaling server. Follow the instructions [here](https://github.com/CatalystCode/3DStreamingToolkit/wiki/Signaling-Service#signaling-server) 
* (optional) For VPN/Corp networks, you will need a TURN server. For setup, follow the instructions [here](https://www.microsoft.com/developerblog/2018/01/29/orchestrating-turn-servers-cloud-deployment). The Docker container with simple TURN server is enough for the end-to-end tests. 

### 3. Modify the `webrtcConfig.json`
In order to test the end-to-end scenario, you need to modify `webrtcConfig.json` and specify the signaling and TURN server created above. In case you do not want to test a VPN/Corp network connection, set the `iceConfiguration` to `none`. The final configuration should look like:
```
{
  "iceConfiguration": "none",
  "turnServer": {
    "uri": "turn:turnserveruri:5349",
    "username": "username",
    "password": "password"
  },
  "serverUri": "https://SIGNALING_URI_CREATED_IN_STEP_2",
  "port": 443,
  "heartbeat": 5000
}
```
If you created a TURN server in step 2, keep the `iceConfiguration` to `relay` and add the `uri`, `username` and `password` in the configuration file:
```
{
  "iceConfiguration": "relay",
  "turnServer": {
    "uri": "turn:TURN_URI_CREATED_IN_STEP_2:3478",
    "username": "TURN_USERNAME_CREATED_IN_STEP_2",
    "password": "TURN_PASSWORD_CREATED_IN_STEP_2"
  },
  "serverUri": "https://SIGNALING_URI_CREATED_IN_STEP_2",
  "port": 443,
  "heartbeat": 5000
}
```


### 3. Running the tests
* Open `cmd` and go to the unzipped folder: e.g. `C:/NativeServerTests/`
* Run the command `NativeServer.Tests.exe --gtest_also_run_disabled_tests`
* If everything is configured correctly, all tests will pass. 

## Tests failures
In case you have one more tests failing, here is a guideline of what could be wrong:
* **[FAILED]EncoderTests.DISABLED_HasCompatibleGPUAndDriver** - Your machine does not have a compatible Nvidia GPU or the Nvidia drivers are outdated. For Desktop, get latest [here](http://www.nvidia.com/Download/index.aspx). For Windows Server 2016 VMs, get latest [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup).

* **[FAILED]EncoderTests.DISABLED_HardwareEncodingIsEnabled** - Same as above

* **[FAILED]EndToEndTests.DISABLED_ServerConnectToSignalingServer** - The server connection to the signaling server was not successful. Make sure you have the correct `serverUri` in the `webrtcConfig.json` or the correct [signaling deployment](https://github.com/CatalystCode/3DStreamingToolkit/wiki/Signaling-Service#signaling-server) 

* **[FAILED] EndToEndTests.DISABLED_ClientConnectToSignalingServer** - Same as above.

* **[FAILED] EndToEndTests.DISABLED_ServerClientConnectToSignalingServer** - Same as above

* **[FAILED] EndToEndTests.DISABLED_SingleClientToServer** - This is the end-to-end streaming test and it can have multiple errors:
   * [ERROR] WAIT_FOR_SERVER - The server connection to the signaling server was not successful.
   * [ERROR] INIT_PEER_CONNECTION - The TURN configuration is invalid. Make sure you have the correct turn `uri`, `username` and `password` in the `webrtcConfig.json`, or, switch the `iceConfiguration` to `none`
   * [ERROR] STREAM_VIDEO - The video stream was not received. Check your Nvidia and GPU drivers. 
   * [ERROR] OPEN_DATA_CHANNEL - The data channel could not open, are you blocking all UDP ports?

## One rendering server connects to multiple clients test
In **DISABLED_MultiClientsToServer** test, you can set the following parameters:
* _MaxClients_: The number of clients connecting to the rendering server at the same time. Note that if you increase this number, the performance will drop.
* _MinFPS_: If there is any client has lower fps than this value, the test will fail.
* _MaxLatency_: If there is any client has higher latency than this value, the test will fail.