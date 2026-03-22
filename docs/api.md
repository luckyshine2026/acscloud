# ACSCloud API Documentation

## Overview

- **Base URL**: `http://your-domain:8888/api/`
- **Authentication**: Token-based
- **Request Format**: JSON
- **Response Format**: JSON

## Authentication

Obtain your Open API Token from the user center after logging into the TR-069 management console.

Include the token in request header:
```
X-Token: {your-api-token}
```

## Standard Response Format

```json
{
  "resCode": "8200",
  "resMessage": "Success!",
  "data": {}
}
```

| resCode | Description |
|---------|-------------|
| 8200 | Success |
| Other | Error, see resMessage |

---

## Device Management

### Get Device Basic Info

```
GET /api/device/get_cpe_basic_info?serialNumber={serialNumber}
```

**Response Example**:
```json
{
  "resCode": "8200",
  "data": {
    "runTimeLength": "91 days 10 hours 26 minutes",
    "lastInformTime": "2022-10-28 18:32:54",
    "onlineStatus": true
  },
  "resMessage": "Success!"
}
```

### Get All Devices

```
GET /api/device/get_all_cpes
```

**Response Example**:
```json
{
  "resCode": "8200",
  "data": [
    {
      "serialNumber": "801047E070000699",
      "cpeId": 9350
    },
    {
      "serialNumber": "2150081736BTG3006839",
      "cpeId": 9351
    }
  ],
  "resMessage": "Success!"
}
```

### Get Device Snapshot

```
GET /api/device/get_cpe_snapshot?serialNumber={serialNumber}
```

**Filter by parameter name**:
```
GET /api/device/get_cpe_snapshot?serialNumber={serialNumber}&param_name=SIP.URI
```

---

## Task Operations

### Set Parameter Values

```
POST /api/task/setParameterValues
```

**Single Device - Single Parameter**:
```json
[
  {
    "serialNumber": "simTestSerial",
    "params": [
      {
        "name": "InternetGatewayDevice.ManagementServer.PeriodicInformInterval",
        "type": "string",
        "value": "60"
      }
    ]
  }
]
```

**Single Device - Multiple Parameters**:
```json
[
  {
    "serialNumber": "simTestSerial",
    "params": [
      {
        "name": "InternetGatewayDevice.Services.VoiceService.1.VoiceProfile.1.Line.1.SIP.AuthUserName",
        "type": "string",
        "value": "abc124"
      },
      {
        "name": "InternetGatewayDevice.Services.VoiceService.1.VoiceProfile.1.Line.1.SIP.AuthPassword",
        "type": "string",
        "value": "test23424"
      }
    ]
  }
]
```

**Multiple Devices - Batch Configuration**:
```json
[
  {
    "serialNumber": "simTestSerial",
    "params": [
      {
        "name": "InternetGatewayDevice.ManagementServer.PeriodicInformInterval",
        "type": "string",
        "value": "100"
      }
    ]
  },
  {
    "serialNumber": "simTestSerial2",
    "params": [
      {
        "name": "InternetGatewayDevice.ManagementServer.PeriodicInformInterval",
        "type": "string",
        "value": "120"
      }
    ]
  }
]
```

**Response**:
```json
{
  "resCode": "8200",
  "data": {
    "failedRequestCpes": [],
    "successRequestCpes": [
      {
        "serialNumber": "simTestSerial",
        "msg": "ACS received configuration request"
      }
    ],
    "requestToken": "53F381ED2C754648B5B65D306B4C8AEE",
    "totalRequestCpes": "1"
  },
  "resMessage": "Success!"
}
```

---

### Reboot Device

```
POST /api/task/reboot
```

**Request**:
```json
[
  {
    "serialNumber": "107BEFFD4AF8"
  }
]
```

**Response**:
```json
{
  "resCode": "8200",
  "data": {
    "failedRequestCpes": [],
    "successRequestCpes": [
      {
        "serialNumber": "107BEFFD4AF8",
        "msg": "ACS received reboot request"
      }
    ],
    "requestToken": "A57060172C5B4E4297E174113C8511EA",
    "totalRequestCpes": "1"
  },
  "resMessage": "Success!"
}
```

---

### Factory Reset

```
POST /api/task/factoryReset
```

**Request**:
```json
[
  {
    "serialNumber": "107BEFFD4AF8"
  }
]
```

---

### Firmware Upgrade

```
POST /api/task/firmwareUpgrade
```

**Request**:
```json
{
  "name": "firmware_upgrade_v1",
  "scheduleTime": "",
  "firmware": {
    "downloadUrl": "http://your-server/firmware/v2.0.0.bin",
    "downloadFileName": "v2.0.0.bin",
    "downloadFileSize": "5242880",
    "downloadPassword": "",
    "downloadUserName": ""
  },
  "serialNumbers": ["107BEFFD4AF8"]
}
```

| Field | Description | Required |
|-------|-------------|----------|
| name | Task name | Yes |
| scheduleTime | Schedule time (yyyy-MM-dd HH:mm:ss), empty for immediate | No |
| firmware.downloadUrl | Firmware download URL | Yes |
| firmware.downloadFileName | Download file name | Yes |
| firmware.downloadFileSize | File size in bytes | Yes |
| serialNumbers | Device serial number array | Yes |

---

### Query Task Status

```
GET /api/task/query?requestToken={requestToken}
```

**Response**:
```json
{
  "resCode": "8200",
  "data": [
    {
      "serialNumber": "107BEFFD4AF8",
      "params": [
        {
          "name": "InternetGatewayDevice.ManagementServer.PeriodicInformInterval",
          "value": "",
          "type": "string",
          "status": "new",
          "msg": "",
          "executeTime": ""
        }
      ]
    }
  ],
  "resMessage": "Success!"
}
```

**Status Values**:
| Status | Description |
|--------|-------------|
| new | New task |
| pending | Pending |
| processing | Processing |
| success | Success |
| failed | Failed |

---

## Common Parameter Paths

### Management Server
```
InternetGatewayDevice.ManagementServer.PeriodicInformInterval  # Inform interval (seconds)
InternetGatewayDevice.ManagementServer.URL                     # ACS URL
```

### WAN Configuration
```
InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.Enable
InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.Username
InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.Password
InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.X_HW_VLAN
InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.X_HW_SERVICELIST
```

### WLAN Configuration
```
InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.Enable
InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.SSID
InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.WPAAuthenticationMode
InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.PreSharedKey.1.PreSharedKey
```

### VoIP/SIP Configuration
```
InternetGatewayDevice.Services.VoiceService.1.VoiceProfile.1.Line.1.SIP.AuthUserName
InternetGatewayDevice.Services.VoiceService.1.VoiceProfile.1.Line.1.SIP.AuthPassword
```

---

## Error Codes

| resCode | Description |
|---------|-------------|
| 8200 | Success |
| 8201 | Invalid parameters |
| 8202 | Device offline |
| 8203 | Device not found |
| 8204 | Task creation failed |
| Other | System error |
