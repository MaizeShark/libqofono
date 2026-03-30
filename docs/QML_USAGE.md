# QML Usage Guide for libqofono

This document describes how to use `libqofono` in a QML/QtQuick application.

## 1. Import the Module

To use `libqofono` in your QML application, import the `QOfono` module:

```qml
import QOfono 0.2
```

*(Note: Older code may use `import MeeGo.QOfono 0.2`, but this is deprecated and you should use `QOfono`).*

## 2. Core Components

The library provides several QML types that map to oFono D-Bus interfaces.

### `OfonoManager`

The `OfonoManager` is the entry point for discovering and managing modems.

*   **Properties**:
    *   `available` (bool): Whether the oFono daemon is available on the system.
    *   `modems` (QStringList): A list of modem paths currently managed by oFono.
    *   `defaultModem` (QString): The path of the default modem.
*   **Signals**:
    *   `modemAdded(modem)`: Emitted when a new modem is detected.
    *   `modemRemoved(modem)`: Emitted when a modem is removed.

**Example**:
```qml
OfonoManager {
    id: manager
    onAvailableChanged: console.log("oFono availability: " + available)
    onModemAdded: console.log("Modem added: " + modem)
}
```

---

### `OfonoModem`

Represents a single modem device. Use it to check the modem's state or to power it on/off.

*   **Properties**:
    *   `modemPath` (string): The D-Bus path of the modem. Set this to target a specific modem.
    *   `powered` (bool): Whether the modem is powered on.
    *   `online` (bool): Whether the modem is online (not in flight mode).
    *   `valid` (bool): Whether the modem object is valid and connected to D-Bus.
    *   `name`, `manufacturer`, `model`, `revision`, `serial`, `type`: Information about the modem hardware.
*   **Signals**:
    *   `poweredChanged`, `onlineChanged`: Emitted when these states change.

**Example**:
```qml
OfonoModem {
    id: modem
    modemPath: manager.defaultModem
    powered: true
    online: true
}
```

---

### `OfonoSimManager`

Manages the SIM card. Use it for PIN entry and to get SIM information.

*   **Properties**:
    *   `present` (bool): Whether a SIM card is present.
    *   `pinRequired` (int): The type of PIN required (e.g., `OfonoSimManager.NoPin`, `OfonoSimManager.SimPin`).
    *   `subscriberIdentity` (string): The IMSI of the SIM card.
    *   `cardIdentifier` (string): The ICCID of the SIM card.
*   **Methods**:
    *   `enterPin(pinType, pin)`: Submit a PIN code.
    *   `changePin(pinType, oldpin, newpin)`: Change a PIN.
    *   `lockPin(pinType, pin)`, `unlockPin(pinType, pin)`: Enable/disable PIN protection.

**Example**:
```qml
OfonoSimManager {
    id: simManager
    modemPath: modem.modemPath
    onPinRequiredChanged: {
        if (pinRequired === OfonoSimManager.SimPin) {
            // Show PIN entry UI
        }
    }
}
```

---

### `OfonoNetworkRegistration`

Handles cellular network registration and operator scanning.

*   **Properties**:
    *   `status` (string): The registration status (e.g., "registered", "roaming", "searching", "unregistered", "denied").
    *   `mode` (string): The registration mode ("auto" or "manual").
    *   `name` (string): The name of the registered operator.
    *   `technology` (string): The current radio technology (e.g., "gsm", "umts", "lte").
    *   `strength` (int): Signal strength (0-100).
    *   `scanning` (bool): Whether a network scan is currently in progress.
    *   `mcc` (string), `mnc` (string): Mobile Country Code and Mobile Network Code.
    *   `networkOperators` (QStringList): List of paths to available network operators.
*   **Methods**:
    *   `scan()`: Start a scan for available network operators. Emits `scanFinished` when done.
*   **Signals**:
    *   `scanFinished()`: Emitted when a manual scan is complete.
    *   `statusChanged(status)`: Emitted when registration status changes.

**Example**:
```qml
OfonoNetworkRegistration {
    id: netReg
    modemPath: modem.modemPath
    onStatusChanged: console.log("Network status: " + status)
    onScanFinished: console.log("Scan complete, operators found: " + networkOperators.length)
}
```

---

### `OfonoVoiceCallManager` & `OfonoVoiceCall`

Used for making and receiving voice calls.

*   **`OfonoVoiceCallManager` Properties**:
    *   `emergencyNumbers` (QStringList): List of emergency numbers.
*   **`OfonoVoiceCallManager` Methods**:
    *   `dial(number, calleridHide)`: Dial a number. `calleridHide` can be "default", "enabled", or "disabled".
    *   `hangupAll()`: Terminate all active calls.
    *   `swapCalls()`: Swaps active and held calls.
*   **`OfonoVoiceCallManager` Signals**:
    *   `callAdded(callPath)`: Emitted when a new call (incoming or outgoing) is created.
    *   `dialComplete(status)`: Emitted when a dial attempt finishes.

*   **`OfonoVoiceCall` Properties**:
    *   `voiceCallPath` (string): The D-Bus path of the call.
    *   `state` (string): Current state ("active", "held", "dialing", "alerting", "incoming", "waiting", "disconnected").
    *   `lineIdentification` (string): The remote party's phone number.
    *   `name` (string): The remote party's name (if available from the SIM).
    *   `startTime` (string): The time the call started.
*   **`OfonoVoiceCall` Methods**:
    *   `answer()`: Answer an incoming call.
    *   `hangup()`: End the call.

**Example (Making a call)**:
```qml
OfonoVoiceCallManager {
    id: vcm
    modemPath: modem.modemPath
    onCallAdded: {
        activeCall.voiceCallPath = callPath
    }
}

OfonoVoiceCall {
    id: activeCall
    onStateChanged: console.log("Call state: " + state)
}

// To dial:
// vcm.dial("123456789", "")
```

---

### `OfonoConnMan` & `OfonoContextConnection`

Used for managing cellular data connections (APNs).

*   **`OfonoConnMan` Properties**:
    *   `attached` (bool): Whether the modem is attached to the packet radio service.
    *   `contexts` (QStringList): List of available connection contexts (APNs).
*   **`OfonoContextConnection` Properties**:
    *   `contextPath` (string): The path of the connection context.
    *   `active` (bool): Whether the connection is active.
    *   `type` (string): The connection type (e.g., "internet", "mms").
    *   `accessPointName` (string): The APN.

**Example**:
```qml
OfonoConnMan {
    id: connMan
    modemPath: modem.modemPath
}

OfonoContextConnection {
    id: dataContext
    contextPath: connMan.contexts.length > 0 ? connMan.contexts[0] : ""
    onActiveChanged: console.log("Data connection active: " + active)
}
```

## 3. Practical Example: Displaying Modem Status

```qml
import QtQuick 2.0
import QOfono 0.2

Column {
    OfonoManager { id: manager }

    OfonoModem {
        id: modem
        modemPath: manager.defaultModem
    }

    Text { text: "Modem: " + (modem.valid ? modem.name : "None") }
    Text { text: "Powered: " + modem.powered }
    Text { text: "Online: " + modem.online }

    Button {
        text: "Toggle Flight Mode"
        onClicked: modem.online = !modem.online
    }
}
```

## 4. Summary Table of QML Types

| QML Type | Purpose | Key Interface Mapping |
| :--- | :--- | :--- |
| `OfonoManager` | Global oFono control | `org.ofono.Manager` |
| `OfonoModem` | Individual modem state | `org.ofono.Modem` |
| `OfonoSimManager` | SIM card management | `org.ofono.SimManager` |
| `OfonoNetworkRegistration` | Network status | `org.ofono.NetworkRegistration` |
| `OfonoVoiceCallManager` | Call handling | `org.ofono.VoiceCallManager` |
| `OfonoVoiceCall` | Individual call state | `org.ofono.VoiceCall` |
| `OfonoConnMan` | Data connection mgmt | `org.ofono.ConnectionManager` |
| `OfonoContextConnection` | Data connection state | `org.ofono.ConnectionContext` |
| `OfonoMessageManager` | SMS management | `org.ofono.MessageManager` |
| `OfonoRadioSettings` | Radio technology selection | `org.ofono.RadioSettings` |
| `OfonoNetworkOperatorListModel` | List of available operators | (Model wrapper for operators) |
| `OfonoSimListModel` | List of SIMs | (Model wrapper for SIMs) |
