# Game Session RPC

Once a controller and game host establish a direct connection (via [`registry.relay`](../registry/relay.md)), they communicate using RPC methods sent over the direct TCP link.

These methods manage the gameplay session: delivering control layouts, configuring sensors, handling virtual buttons, cookies, and trial/purchase flows.

## Session Flow

```mermaid
sequenceDiagram
    participant Host as Game Host
    participant Controller

    Host->>Controller: AckPacket
    Controller->>Host: GetPortalId
    Controller->>Host: setCapabilities(mask)
    Host-->>Controller: onPortalId(id)
    Controller->>Host: RequestXML(height, width, deviceId)
    opt some hosts set the mode while answering the request
        Host->>Controller: SetControlMode(GAME)
    end
    Host-->>Controller: XML delivered as BMByteChunk
    Controller->>Host: onControlSchemeParsed(deviceId)
    opt when the host wants input on a particular transport
        Host->>Controller: setReliabilityForTouch(touch, sensors)
    end

    note over Host,Controller: Gameplay Loop
    Controller->>Host: Touch / Acceleration / Gyro / DPad
    Host->>Controller: vibrate / getCookie / setCookie
```

## RPC Methods

### Host to Controller

| Method | Description |
|--------|-------------|
| [`SetControlMode`](set-control-mode.md) | Switch the controller to a specific input mode. |
| [`enableAccelerometer`](sensor-config.md) | Enable/disable accelerometer, optionally set sample interval. |
| [`setTouchInterval`](sensor-config.md) | Set the touch input sample interval. |
| [`enableTouch`](sensor-config.md) | Enable or disable touch input. |
| [`enableGyro`](sensor-config.md) | Enable or disable gyroscope input. |
| [`setGyroInterval`](sensor-config.md) | Set the gyroscope sample interval. |
| [`enableOrientation`](sensor-config.md) | Enable or disable orientation input. |
| [`setOrientationInterval`](sensor-config.md) | Set the orientation sample interval. |
| [`setReliabilityForTouch`](reliability-config.md) | Set touch/sensor reliability. |
| [`getCookie`](cookies.md) | Request a persistent value from the controller's storage. |
| [`setCookie`](cookies.md) | Write a persistent value to the controller's storage. |
| [`vibrate`](vibrate.md) | Trigger the controller's vibration motor. |
| [`promptTrialUpsell`](trial-purchase.md) | Prompt the controller to show the trial upsell UI. |
| [`updateWallet`](trial-purchase.md) | Tell the controller to refresh its in-app currency wallet. |
| [`WaitForNewHost`](trial-purchase.md) | Hand the controller a portal identifier and park it in `WAIT` mode. |

### Controller to Host

| Method | Description |
|--------|-------------|
| [`GetPortalId`](portal-id.md) | Request the host's portal identifier. |
| [`setCapabilities`](capabilities.md) | Report the controller's hardware capabilities as a bitmask. |
| [`RequestXML`](request-xml.md) | Request the control layout schema for the controller's screen dimensions. |
| [`onNavigationString`](navigation-keyboard.md) | Deliver a system navigation event (e.g., back button). |
| [`onKeyString`](navigation-keyboard.md) | Deliver a hardware keyboard key press. |
| [`onControlSchemeParsed`](request-xml.md) | Confirm the control scheme arrived and parsed. |
| [`gotCookie`](cookies.md) | Answer a `getCookie` with the stored name and value. |
| [`menuEvent`](../controls/context-menu.md) | Report that a context menu option was chosen. |
| `bmPause` | Report that the player opened or closed the controller's own menu. |
| the scheme's `functionHandler` names | Report a button press or release. One method per button, named by the [control scheme](../controls/display-objects.md). |
| [`startTrial`](trial-purchase.md) | Signal the start of a trial session. |
| [`endTrial`](trial-purchase.md) | Signal the end of a trial session. |

## Notes

- All session RPCs use the [`BMInvoke`](../objects/bm-invoke.md) object format on channel 3 (Message).
- Unlike registry traffic, session RPC occurs directly between peers. The registry server is not involved during gameplay.
- The control scheme XML is delivered via [`BMByteChunk`](../objects/bm-byte-chunk.md) chunked transfer. See [RequestXML & Delivery](request-xml.md).
