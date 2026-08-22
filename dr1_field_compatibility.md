# DR1 CAN — field command compatibility

Captured from the debug **Command console** against a real unit on 2026-08-17. Extracted from the
on-screen results before an overnight Doze restart cleared them, so it is a partial pass (commands
`0x01`–`0x26`); the groups below `SILENT_UNLOCK` still need a re-run. The on-device
`CommandCompatStore` now persists these automatically, so future passes survive leaving the screen.

## Unit under test

| Field | Value |
|---|---|
| Model | **DR1 CAN** (READ_MODE → unit type DR1 CAN) |
| Firmware | v7 |
| Mode during test | User |
| Loaded CAN file | **`ACCORD_19`** (CAN_FILENAME) |
| VIN | **`1HGCV3F42LA013794`** (READ_VIN) → WMI `1HG` = Honda |
| Vehicle | **Honda Accord, 2019** (CAN file + VIN agree) |
| Serial | `DR D029A54D` |
| Battery | ~12.3 V |

## Rejection codes observed

The `ACK_ERROR` code is the real signal for what a model supports:

| Code | Meaning | Interpretation |
|---|---|---|
| `01` | "does not recognize or permit this command" | **Unsupported** on this model |
| `02` / `04` | invalid parameter / invalid setting | **Recognized**; needs a valid payload |
| `03` | invalid mode | **Recognized**; needs a different unit mode (auto-mode-change resolves this) |

## Results

### ✅ Accepted (confirmed working)

| Code | Command | Notes |
|---|---|---|
| 0x01 | READ_ALL_SETTINGS | full settings; Alarm color: Green |
| 0x05 | CHANGE_MODE | ACK_OK |
| 0x1E | READ_FIRMWARE_VERSION | firmware v7, selector 0x04 |
| 0x1F | READ_VIOLATIONS | 0 stored triggers |
| 0x20 | CLEAR_VIOLATIONS | ACK_OK |
| 0x21 | READ_STATE | ignition off, disarmed, all closed, batt 12.268 V, GPS |
| 0x22 | READ_MODE | DR1 CAN, v7, User |
| 0x23 | READ_VIN | `1HGCV3F42LA013794` |
| 0x27 | READ_SERIAL | `DR D029A54D` |
| 0x28 | READ_VIOLATION_INHIBIT | door/hood/rear/tilt inhibited |
| 0x2C | READ_VEHICLE_BATTERY_VOLTAGE | 12.340 V |
| 0x2F | DEBUG_MESSAGE | ACK_OK |
| 0x32 | GET_TILT_SENSITIVITY | 0x28 |
| 0x40 | READ_RESET_CAUSE | 3C 02 |
| 0x41 | READ_IO_PROTECTION | 00 |
| 0x0C | UNLOCK_DOORS | ACK_OK |
| 0x12 | CAN_FILENAME | `ACCORD_19` |
| 0x13 | CAN_READ_FLAG | 0x0A |
| 0x14 | CAN_FILE_VERSION | v1.0 |
| 0x26 | SILENT_UNLOCK | ACK_OK |

### READ_SPECIFIC_SETTING field survey

Captured from the persistent debug compatibility store on 2026-08-17. A successful response is
two bytes: the requested setting id echoed first, followed by its current value. The bytes are not
encrypted.

| Setting | Raw response | DR1 interpretation |
|---|---|---|
| `0x00` | `00 00` | Auto-lock with ignition: **Off** |
| `0x01` | `01 00` | Auto-unlock with ignition: **Off** |
| `0x02` | `02 01` | Passive arm locks doors: **Yes** |
| `0x03` | `03 01` | User arming: **Active** |
| `0x04` | `04 02` | Lock/unlock pulse: **Double unlock** |
| `0x05` | `05 01` | Short-pulse duration: **400 ms** |
| `0x06` | `06 01` | Arm/disarm tones: **On** |
| `0x07` | `07 00` | Parking-light OEM trigger: **Off** |
| `0x08` | `08 00` | **Unmapped on DR1/CAN**; current value `0` |
| `0x09` | `09 00` | **Unmapped on DR1/CAN**; current value `0` |
| `0x0A` | `0A 01` | Confirmation chirp duration: **14 ms** |
| `0x0B` | `0B 02` | Factory-alarm output duration: **3 seconds** |
| `0x0E` | `0E 01` | Alarm output: **Horn** |
| `0x14` | `14 00` | Dealer arming: **Passive after doors** |
| `0x15` | `15 00` | Auto-lock on Bluetooth disconnect: **Off** |

No persistent result was present for `0x0C`, `0x0D`, or `0x0F`–`0x13`; these must be tested or
re-tested before recording a value.

### ❌ Unsupported on DR1 (rejected code 01)

READ_GPIO (0x09), VEHICLE_EVENT (0x0D), REQUEST_AUTH (0x0E), PROVIDE_AUTH (0x0F), SECURE_ARM (0x44),
SECURE_DISARM (0x45), READ_MODE_HISTORY (0x46), SNOW_MODE (0x47), READ_REGISTER (0x48),
ELITE_MODE (0x49), ALARM_TRIGGER (0x4B), GET_MOVEMENT_SENSITIVITY (0x4D).

Note: GET_TILT_SENSITIVITY works but GET_MOVEMENT_SENSITIVITY does not.

### ⚠️ Recognized but needs correct input

| Code | Command | Reject |
|---|---|---|
| 0x02 | READ_SPECIFIC_SETTING | code 02 — needs a setting id |
| 0x04 | WRITE_SPECIFIC_SETTING | code 04 — needs id + value |
| 0x30 | BUMP_TEST | code 03 — needs Installer mode |
| 0x18 | VEHICLE_CONTROL (payload 02) | code 03 — needs a different mode |

### ⏱️ No response (GATT accepted, no unit reply)

LOCK_DOORS (0x0B), SILENT_LOCK (0x25). Pattern: the *unlock* variants ACK, the *lock* variants go
silent.

### Not tested (destructive / skipped)

WRITE_ALL_SETTINGS (0x03), CONFIGURE_GPIO (0x0A), RESET_TO_DEFAULT (0x17), WRITE_VIN (0x24),
WRITE_VIOLATION_INHIBIT (0x29), DISCONNECT_BT (0x2D), LOW_POWER_OR_REPAIR_MODE (0x2E),
SET_TILT_SENSITIVITY (0x31), RESET_UNIT (0x42), SET_MOVEMENT_SENSITIVITY (0x4C),
SET_SECURE_KEY (0x4E), and the CAN file-transfer commands (0x06–0x08, 0x16).

### Not captured (overnight restart cut the pass short — re-run these)

CHECK_CAN_ERRORS (0x2A), LEARN_DOORS (0x2B), DOOR_CHANGE (0x33), BUS_STATE (0x34),
CAN_FILE_DELETE (0x4A), CAN_LEARN_MODE (0x15), and the entire **RF remotes**, **GM-LAN**,
**PIC2 / 2nd MCU**, **Bootloader**, and **Protocol / response** groups.
