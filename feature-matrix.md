Discover what features are landed across the different components and targets of the toolkit.

### Client feature matrix

<table>
  <tr>
    <th>Platform</th>
    <th>Low-latency AV Streaming</th>
    <th>Low-latency Data Streaming</th>
    <th>HTTP Signalling</th>
    <th>HTTPS Signalling</th>
    <th>Signalling Heartbeat</th>
    <th>OAuth24D Authentication</th>
    <th>Transmitted TURN credentials</th>
  </tr>
  <tr>
    <td>Hololens DirectX</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>Hololens Unity</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌</td>
    <td>❌</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>Win32 DirectX</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
  </tr>
  <tr>
    <td>Chrome</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌ (unneeded, supports direct oauth2)</td>
    <td>✔️</td>
  </tr>
<tr>
    <td>Firefox (untested)</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌ (unneeded, supports direct oauth2)</td>
    <td>✔️</td>
  </tr>
  <tr>
    <td>Edge (untested)</td>
    <td>✔️</td>
    <td>❌</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌ (unneeded, supports direct oauth2)</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>Android Native</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>iOS Native</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
</table>

### Server feature matrix

<table>
  <tr>
    <th>Platform</th>
    <th>Hardware encoding</th>
    <th>Low-latency AV Streaming</th>
    <th>Low-latency Data Streaming</th>
    <th>HTTP Signalling</th>
    <th>HTTPS Signalling</th>
    <th>Signalling Heartbeat</th>
    <th>OAuth2 Authentication</th>
    <th>Transmitting TURN credentials</th>
  </tr>
  <tr>
    <td>Win32 Directx</td>
    <td>✔️(requires nvidia keplar or higher)</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️(client_credentials grant)</td>
    <td>✔️</td>
  </tr>
  <tr>
    <td>Win32 Unity</td>
    <td>✔️(requires nvidia keplar or higher)</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>✔️</td>
    <td>❌</td>
    <td>❌</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
</table> 
