updated version:  [surplife_protocol_description.md](https://gist.github.com/marcelrv/36f9d2c66a273289211cc5fa73b00018#file-surplife_protocol_description-md)
 
 # Commands to control the light


For the LED Curtain, the initialization sequence consists of setting the current device time and performing a handshake/keep-alive check.

Here is the deciphered protocol description.

# 🔌 LED Curtain Initialization Sequence

## 1. Time Synchronization (`0x10`)
**Direction:** App $\to$ Device  
**Sequence Nr:** 1  
**Command:** `0x0A` (Application Layer Header) + Payload

The app immediately sends the current local time to the device. This is used for scheduled timers and alarms stored on the device.

**Raw Payload:** `10 14 19 0b 1c 13 20 26 05 00 0f d1`

| Byte Index | Hex Value | Decimal | Meaning | Decoded Value |
| :--- | :--- | :--- | :--- | :--- |
| **0** | `0x10` | 16 | **Opcode** | **Set Time** |
| **1** | `0x14` | 20 | Year (Century) | 20xx |
| **2** | `0x19` | 25 | Year (Year) | xx25 |
| **3** | `0x0B` | 11 | Month | November |
| **4** | `0x1C` | 28 | Day | 28th |
| **5** | `0x13` | 19 | Hour | 19 (7 PM) |
| **6** | `0x20` | 32 | Minute | 32 min |
| **7** | `0x26` | 38 | Second | 38 sec |
| **8** | `0x05` | 5 | Day of Week | Friday (Mon=1 ... Fri=5) |
| **9** | `0x00` | 0 | Reserved | - |
| **10-11**| `0x0F D1`| - | Checksum/End | - |

**📅 Decoded Timestamp:** `2025-11-28 19:32:38` (Friday)

---

## 2. Handshake / Keep-Alive (`0xEA`)
**Direction:** App $\to$ Device  
**Sequence Nr:** 2

This packet verifies the connection is active and typically requests a status update from the device.

**Raw Payload:** `ea 81 8a 8b 59`

| Byte Index | Hex Value | Meaning |
| :--- | :--- | :--- |
| **0** | `0xEA` | **Opcode:** Keep Alive / Ping |
| **1-4** | `0x81 8A 8B 59` | **Magic Bytes / Challenge:** This specific sequence is a standard "Hello" signature used in Surplife/MagicHome BLE devices. |

---

## 3. Device Status Report (Response)
**Direction:** Device $\to$ App (Notification)  
**UUID:** `0000ff02...`

The device responds to the handshake by reporting its current state.

**Raw Data:** `ea 81 01 00 c2 03 23 62 01 64 f0 ...`

| Byte Index | Hex Value | Deciphered Meaning |
| :--- | :--- | :--- |
| **0** | `0xEA` | **Response Opcode** (Matches the request `0xEA`) |
| **1** | `0x81` | Device Type Category? |
| **2** | `0x01` | **Power State:** `0x01` = **ON** |
| **3** | `0x00` | **Mode:** `0x00` (Likely RGB/Color Mode) |
| **4** | `0xC2` | Device Model ID (Curtain Controller) |
| **5** | `0x03` | Firmware Major Version |
| **6-7** | `0x23 62` | Firmware Minor Version / Build |
| **8** | `0x01` | Active Effect ID? |
| **9** | `0x64` | **Brightness / Speed:** `100` (100%) |
| **10** | `0xF0` | Data Delimiter |

### Summary of Initialization
1.  **Set Time:** The app tells the curtain it is **Friday, Nov 28, 2025, at 19:32:38**.
2.  **Handshake:** The app sends the standard "Hello" bytes.
3.  **Status:** The curtain replies "I am **ON**, brightness is **100%**, and I am a Curtain Device (`0xC2`)."


## Switch On
```
[BLE UpperTransportLayer.createUpper =>] isAct: false isProtect: false sequenceNr: 3 cmdId: 0a payload: 71 24 len:5
[BLE Write  =>] UUID: 0000ff01-0000-1000-8000-00805f9b34fb data: 0x 0a 71 24 - header : 00 03 80 00 00 02 03  - full: 00 03 80 00 00 02 03 0a 71 24
[BLE Notify <=] UUID: 0000ff02-0000-1000-8000-00805f9b34fb data: 0x  ea 81 01 00 c2 03 23 66 01 32 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00 - header : 04 32 80 00 00 19 1a 15 - full: 04 32 80 00 00 19 1a 15 ea 81 01 00 c2 03 23 66 01 32 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00
[BLE Notify <=] UUID: 0000ff02-0000-1000-8000-00805f9b34fb data: 0x  ea 81 01 00 c2 03 24 66 01 32 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00 - header : 04 33 80 00 00 19 1a 16 - full: 04 33 80 00 00 19 1a 16 ea 81 01 00 c2 03 24 66 01 32 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00

 ```

## Colourfull mode

| Byte Index | Field Name      | Description                                                                 |
|------------|-----------------|-----------------------------------------------------------------------------|
| 0          | ?               | 0xe0                                                                        |
| 1          | Mode            | 0x0b for colorfull mode                                                       |
| 2        | Hue    | color 360 degree mapped to 0-255                                                      |
| 3        | brightness    | Represents intensity from 0x00 (Off) to 0xFF (100%).        |                                               


payload: e2 0b 29 bf

example
```
[Remote::Gadget ]-> [BLE UpperTransportLayer.createUpper =>] isAct: false isProtect: false sequenceNr: 4 cmdId: 0a payload: e2 0b 29 bf len:11
[BLE Write  =>] UUID: 0000ff01-0000-1000-8000-00805f9b34fb data: 0x 0a e2 0b 29 bf - header : 00 04 80 00 00 04 05  - full: 00 04 80 00 00 04 05 0a e2 0b 29 bf
[BLE UpperTransportLayer.createUpper =>] isAct: false isProtect: false sequenceNr: 5 cmdId: 0a payload: e2 0b 65 9f len:11
```


## Effect mode

| Byte Index | Field Name      | Description                                                                 |
|------------|-----------------|-----------------------------------------------------------------------------|
| 0          | ?               | 0xe0                                                                        |
| 1          | Mode            | 0x02 for effect mode                                                       |
| 2          | ?               |  0x00                                                        |
| 3          | effect id       |  0x00 to 0x11                                                        |
| 4          | speed          |  0x00 to 0x64 (0-100%)                                                        |
| 5          | brightness     |  0x00 to 0x64 (0-100%)                                                        |


payload: e0 02 00 0f 1f 50 

```
[BLE UpperTransportLayer.createUpper =>] isAct: false isProtect: false sequenceNr: 21 cmdId: 0a payload: e0 02 00 02 50 50 len:17
[BLE Write  =>] UUID: 0000ff01-0000-1000-8000-00805f9b34fb data: 0x 0a e0 02 00 02 50 50 - header : 00 15 80 00 00 06 07  - full: 00 15 80 00 00 06 07 0a e0 02 00 02 50 50
[BLE Notify <=] UUID: 0000ff02-0000-1000-8000-00805f9b34fb data: 0x  ea 81 01 00 c2 03 23 67 02 50 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00 - header : 04 3d 80 00 00 19 1a 16 - full: 04 3d 80 00 00 19 1a 16 ea 81 01 00 c2 03 23 67 02 50 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00
```

## Music mode

| Byte Index | Field Name      | Description                                                                 |
|------------|-----------------|-----------------------------------------------------------------------------|
| 0          | ?               | 0xe1                                                                        |
| 1          | Mode            | 0x05 for music                                                       |
| 2          | active               |  0x00 off 0x01 on                                                |
| 3          | brightness          |  0x00 to 0x64 (0-100%)                                                        |
| 4          | effect id       |  0x00 to 0x11                                                        |
| 5          | ?                |   |
| 6          | ?                |                                                          |
| 7          | Sensetivity     |  0x00 to 0x64 (0-100%)                                                        |


payload: `e1 05 00 64 03 00 00 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 a1 00 00 00 06 a1 00 64 64 a1 96 64 64 a1 78 64 64 a1 5a 64 64 a1 3c 64 64 a1 1e 64 64`

``` 
[BLE UpperTransportLayer.createUpper =>] isAct: false isProtect: false sequenceNr: 57 cmdId: 0a payload: e1 05 00 64 03 00 00 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 a1 00 00 00 06 a1 00 64 64 a1 96 64 64 a1 78 64 64 a1 5a 64 64 a1 3c 64 64 a1 1e 64 64 len:161
[BLE Write  =>] UUID: 0000ff01-0000-1000-8000-00805f9b34fb data: 0x 0a e1 05 00 64 03 00 00 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 a1 00 00 00 06 a1 00 64 64 a1 96 64 64 a1 78 64 64 a1 5a 64 64 a1 3c 64 64 a1 1e 64 64 - header : 00 39 80 00 00 36 37  - full: 00 39 80 00 00 36 37 0a e1 05 00 64 03 00 00 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 a1 00 00 00 06 a1 00 64 64 a1 96 64 64 a1 78 64 64 a1 5a 64 64 a1 3c 64 64 a1 1e 64 64
[BLE Notify <=] UUID: 0000ff02-0000-1000-8000-00805f9b34fb data: 0x  ea 81 01 00 c2 03 23 62 03 64 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00 - header : 04 5f 80 00 00 19 1a 16 - full: 04 5f 80 00 00 19 1a 16 ea 81 01 00 c2 03 23 62 03 64 f0 00 00 00 00 00 01 01 90 00 00 01 02 03 00
```

## phone music mode


| Byte Index | Field Name      | Description                                                                 |
|------------|-----------------|-----------------------------------------------------------------------------|
| 0          | ?               | 0xe2                                                                        |
| 1          | Mode            | 0x0a for music                                                       |
| 2 -18          | volume               |  ?                                                 |
Note, this is preceeded by the ox05 mudic mode setting.
payload with values send several times per second
payload: e2 0a 01 01 01 04 05 06 05 04 03 02 01 01 00 00 00 00 00 00 00 00 len:65




 
# 🛠️ Color Calibration Protocol

The calibration process is used to map the app's logical color commands (Red, Green, Blue) to the specific physical output pins on the LED controller. This fixes issues where selecting "Red" might turn the lights "Green" or "Blue" due to different LED strip wiring standards (e.g., RGB vs. GRB vs. RBG).

**Parent Opcode:** `0xE0` (Direct Control)

## 1. Sequence Overview
The calibration process follows a strict three-step sequence:
1.  **Raw Channel Test (`0x15`):** The app lights up physical pins one by one (bypassing current software mapping) and asks the user what color they see.
2.  **Set Mapping (`0x13`):** Based on user input, the app calculates the correct pin order and sends it to the device.
3.  **Save & Exit (`0x0E`):** The app tells the device to save this configuration to flash memory.

---

## 2. Step 1: Raw Channel Preview
**Sub-Command:** `0x15`

This command drives specific hardware channels directly with a 0-255 intensity value. It ignores any previously saved color calibration.

**Format:** `0A E0 15 00 00 [Ch1] [Ch2] [Ch3] [Ch4] [Ch5]`

| Byte | Value | Description |
| :--- | :--- | :--- |
| **0** | `0x0A` | Protocol Header |
| **1** | `0xE0` | Opcode |
| **2** | **`0x15`** | **Mode: Raw Channel Test** |
| **3-4**| `0x00` | Reserved |
| **5** | `0x00`-`0xFF` | **Physical Pin 1 Intensity** (Default: Red Pin) |
| **6** | `0x00`-`0xFF` | **Physical Pin 2 Intensity** (Default: Green Pin) |
| **7** | `0x00`-`0xFF` | **Physical Pin 3 Intensity** (Default: Blue Pin) |
| **8** | `0x00`-`0xFF` | Physical Pin 4 (Warm White) |
| **9** | `0x00`-`0xFF` | Physical Pin 5 (Cool White) |

**Example from Logs:**
*   `e0 15 00 00 ff 00 00...` -> Turns on **Pin 1** to Max.
*   `e0 15 00 00 00 ff 00...` -> Turns on **Pin 2** to Max.

---

## 3. Step 2: Set Channel Mapping
**Sub-Command:** `0x13`

This command defines which physical pin corresponds to which logical color.

**Format:** `0A E0 13 [R_Map] [G_Map] [B_Map] [W_Map] [CW_Map]`

| Byte | Value | Description |
| :--- | :--- | :--- |
| **0** | `0x0A` | Protocol Header |
| **1** | `0xE0` | Opcode |
| **2** | **`0x13`** | **Mode: Set Channel Mapping** |
| **3** | `Index` | **Logical Red** maps to Physical Pin Index `X` |
| **4** | `Index` | **Logical Green** maps to Physical Pin Index `Y` |
| **5** | `Index` | **Logical Blue** maps to Physical Pin Index `Z` |
| **6** | `Index` | Logical Warm White Map |
| **7** | `Index` | Logical Cool White Map |

**Mapping Logic:**
*   `00` = Pin 1
*   `01` = Pin 2
*   `02` = Pin 3

**Example:**
If the strip is **RGB** (Standard): Payload is `00 01 02` (Red=Pin1, Green=Pin2, Blue=Pin3).
If the strip is **GRB** (Common): Payload is `01 00 02` (Red=Pin2, Green=Pin1, Blue=Pin3).

**Log Data:**
> `e0 13 00 01 02 03 04 05`
> *   Red = Pin 0
> *   Green = Pin 1
> *   Blue = Pin 2
> *   (This sets the device to standard RGB order).

---

## 4. Step 3: Save Configuration
**Sub-Command:** `0x0E`

Finalizes the procedure. The device writes the mapping from Step 2 into non-volatile memory and returns to normal operation mode.

**Format:** `0A E0 0E 01`

| Byte | Value | Description |
| :--- | :--- | :--- |
| **0** | `0x0A` | Header |
| **1** | `0xE0` | Opcode |
| **2** | **`0x0E`** | **Mode: Calibration Config** |
| **3** | `0x01` | **Action:** Save / Confirm |