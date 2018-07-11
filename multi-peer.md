A common desired scenario is to connect multiple peers to a single instance of a server. [#152](https://github.com/3DStreamingToolkit/3DStreamingToolkit/pull/152) added support for just this! 🎉 

## Configuring multi-peer

> For configurable multi-peer to work properly, the signaling server implementation must support [webrtc-signal-http-capacity](https://github.com/bengreenier/webrtc-signal-http-capacity).

By default, Multi-peer is enabled in our samples with support for an unlimited number of clients. However, this is likely not desired for a production scenario. To limit the number of peers that can connect to a server we must modify our configuration value, namely `systemCapacity` from [`serverConfig.json`](./webrtc-config.md).

For example, to limit the server to accepting only `2` peer connections, we'd set the following value:

```
"systemCapacity": 2
```

By default, this value is set as follows:

```
"systemCapacity": -1
```

Which disables having any max capacity - In other words, by default an infinite number of peers is allowed.

## Caveats

Here are some of the caveats to keep in mind when leveraging multi-peer features:

* Nvidia [enforces](https://stackoverflow.com/questions/30490505/nvenc-fail-to-compress-h264-with-for-multiple-video-streams) a maximum of 2 gpu encoding sessions on desktop series graphics cards
* Capacity management is controlled via [signaling](https://github.com/bengreenier/webrtc-signal-http-capacity)