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
