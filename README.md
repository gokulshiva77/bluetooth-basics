# 📡 Bluetooth Stack and Communication Overview

This document provides a deep dive into the Bluetooth protocol stack, roles, architecture, HCI flow, and how it maps to the OSI model. It is intended as a reference for developers working with Bluetooth profiles like HFP, A2DP, AVRCP, and more.

---

## 🧱 Bluetooth Architecture Overview

Bluetooth is divided into three main planes:

- **Controller**: Hardware + firmware (Radio, Baseband, Link Manager)
- **Host**: Software stack (L2CAP, SDP, RFCOMM, etc.)
- **Host Controller Interface (HCI)**: Interface between host and controller

Application
↑
Bluetooth Profiles (e.g., A2DP, HFP)
↑
Host Protocols (L2CAP, SDP, RFCOMM, AVDTP, etc.)
↑
HCI (Host Controller Interface)
↑
Controller (Link Manager, Baseband, Radio)

---

## 🧩 Roles and Responsibilities

| **Component**        | **Responsibility**                                                         |
|----------------------|-----------------------------------------------------------------------------|
| Application          | User-level features (e.g., audio player, dialer, file browser)             |
| Bluetooth Profiles   | Define service-specific behavior (e.g., HFP, A2DP, AVRCP)                  |
| Host Protocol Stack  | Provides protocol support (L2CAP, SDP, RFCOMM, AVDTP, AVCTP)               |
| HCI                  | Interface for control/data between Host and Controller                     |
| Controller           | Handles RF, Baseband, Link Manager, SCO/ACL connections                    |
| Device Roles         | Defines Master/Slave, Source/Sink, AG/HF based on context                  |

---

## 🔄 HCI Command Flow: Pair and Connect with Profiles

### 1. Device Discovery
```plaintext
HCI_Inquiry
HCI_Inquiry_Cancel
HCI_Remote_Name_Request
```
### 2. Pairing / Authentication (SSP or Legacy)
```plaintext
HCI_Create_Connection
HCI_IO_Capability_Request_Event
HCI_User_Confirmation_Request_Event
HCI_Link_Key_Notification_Event
HCI_Simple_Pairing_Complete_Event
```
#### Legacy Pairing
```plaintext
HCI_PIN_Code_Request_Event
HCI_PIN_Code_Request_Reply
```

#### 3. Set Encryption
```plaintext
HCI_Set_Connection_Encryption
```

### 4. SDP Discovery (Service Lookup)
Performed over L2CAP (PSM 0x0001)
Look for UUIDs:
HFP AG: 0x111F, HF: 0x111E
A2DP: 0x110A (Source), 0x110B (Sink)
AVRCP: 0x110C (Target), 0x110E (Controller)

### 5. RFCOMM Setup (e.g., HFP, AVRCP Control)
```plaintext
L2CAP_Create_Channel (PSM = 0x0003)
RFCOMM_Setup_Command
RFCOMM_Connection_Complete
```

### 6. AVDTP Setup (A2DP)
```plaintext
L2CAP_Create_Channel (PSM = 0x0019)
AVDTP_DISCOVER
AVDTP_GET_CAPABILITIES
AVDTP_SET_CONFIGURATION
AVDTP_OPEN
AVDTP_START
```

### 7. SCO/eSCO for Voice (HFP Audio)
```plaintext
HCI_Add_SCO_Connection
```
### 🧵 RFCOMM vs L2CAP
| **OSI Layer**    | **OSI Description**                 | **Bluetooth Equivalent**                     | **Bluetooth Description**                           |
| ---------------- | ----------------------------------- | -------------------------------------------- | --------------------------------------------------- |
| 7 - Application  | End-user software                   | Bluetooth Application (Media Player, Dialer) | Provides actual user-facing features                |
| 6 - Presentation | Data translation, encryption        | Codec/Encryption handled in app or profiles  | SBC codec, optional encryption (SSP)                |
| 5 - Session      | Session setup and management        | Bluetooth Profiles (HFP, A2DP, AVRCP, etc.)  | Defines signaling and roles                         |
| 4 - Transport    | Reliable transport and flow control | RFCOMM, AVDTP, AVCTP, L2CAP                  | Streaming, control, or serial data                  |
| 3 - Network      | Addressing, routing (IP)            | BNEP (used in PAN profile)                   | Used for IP over Bluetooth in PAN                   |
| 2 - Data Link    | Framing, MAC, error detection       | L2CAP, HCI, Link Manager                     | Logical links, multiplexing, pairing                |
| 1 - Physical     | Bit-level transmission              | Bluetooth Radio & Baseband                   | 2.4 GHz transmission, modulation, packet formatting |


### 🧠 OSI Model Refresher
| Layer | Name         | Description                                      |
| ----- | ------------ | ------------------------------------------------ |
| 7     | Application  | Interfaces with user apps like browsers, media   |
| 6     | Presentation | Formats, encrypts, compresses data               |
| 5     | Session      | Manages sessions between systems                 |
| 4     | Transport    | End-to-end communication (e.g., TCP)             |
| 3     | Network      | Routing, addressing (e.g., IP)                   |
| 2     | Data Link    | Node-to-node transmission, MAC                   |
| 1     | Physical     | Transmission over physical media (bits, signals) |


### 🔄 OSI Model vs Bluetooth Stack
| **OSI Layer**    | **OSI Description**                 | **Bluetooth Equivalent**                     | **Bluetooth Description**                           |
| ---------------- | ----------------------------------- | -------------------------------------------- | --------------------------------------------------- |
| 7 - Application  | End-user software                   | Bluetooth Application (Media Player, Dialer) | Provides actual user-facing features                |
| 6 - Presentation | Data translation, encryption        | Codec/Encryption handled in app or profiles  | SBC codec, optional encryption (SSP)                |
| 5 - Session      | Session setup and management        | Bluetooth Profiles (HFP, A2DP, AVRCP, etc.)  | Defines signaling and roles                         |
| 4 - Transport    | Reliable transport and flow control | RFCOMM, AVDTP, AVCTP, L2CAP                  | Streaming, control, or serial data                  |
| 3 - Network      | Addressing, routing (IP)            | BNEP (used in PAN profile)                   | Used for IP over Bluetooth in PAN                   |
| 2 - Data Link    | Framing, MAC, error detection       | L2CAP, HCI, Link Manager                     | Logical links, multiplexing, pairing                |
| 1 - Physical     | Bit-level transmission              | Bluetooth Radio & Baseband                   | 2.4 GHz transmission, modulation, packet formatting |


### 📘 Common Bluetooth UUIDs
| **Profile**         | **UUID** |
| ------------------- | -------- |
| HFP (Hands-Free)    | `0x111E` |
| HFP (Audio Gateway) | `0x111F` |
| A2DP Source         | `0x110A` |
| A2DP Sink           | `0x110B` |
| AVRCP Controller    | `0x110E` |
| AVRCP Target        | `0x110C` |


### 📌 Bluetooth Connection Types
| **Type** | **Use**                                 |
| -------- | --------------------------------------- |
| ACL      | Asynchronous Connection-Less (Data)     |
| SCO      | Synchronous Connection-Oriented (Voice) |
| eSCO     | Enhanced SCO (Voice + retransmission)   |


### 📖 References
* [Bluetooth Core Specification](https://www.bluetooth.com/specifications/specs/)
* Bluetooth Profiles: A2DP, HFP, AVRCP, SDP
* OSI Model (ISO/IEC 7498-1)

🛠️ Author
Prepared by Gokul