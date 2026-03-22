# ACSCloud - TR-069 ACS Cloud Platform

**ACSCloud** is a scalable and feature-rich TR-069 Auto Configuration Server (ACS) designed for remote management, batch configuration, and firmware upgrades of CPE/ONT/IoT devices.

Ideal for telecom operators, ISPs, smart communities, and industrial IoT deployments requiring large-scale device management.

---

## Key Features

### Device Management
- **Auto Registration** - Zero-config device onboarding with automatic authentication
- **Real-time Monitoring** - Monitor device online status, uptime, and last inform time
- **Batch Operations** - Bulk reboot, factory reset, and configuration deployment
- **Organization Management** - Multi-level organization structure for regional/carrier-based grouping

![Device List](./docs/screenshots/cpelist.png)

![Device View](./docs/screenshots/device_view.png)

![System Info](./docs/screenshots/systeminfo.png)

### Parameter Configuration
- **Visual Templates** - Configure WAN, LAN, WLAN, SIP, VoIP parameters via templates
- **Batch Deployment** - Configure multiple devices simultaneously with async execution
- **Configuration Snapshots** - Historical version tracking with rollback support
- **Custom RPC Paths** - Flexible parameter path configuration

![WAN Template](./docs/screenshots/wan_template.png)

![RPC Tree](./docs/screenshots/rpctree.png)

### Firmware & Updates
- **Firmware Upgrade** - Batch firmware updates with scheduled execution
- **Config File Distribution** - Vendor Configuration File batch deployment
- **Version Management** - Firmware version control and compatibility matching

### Diagnostics
- **Ping Test** - Remote device ping diagnostics
- **Connection Diagnostics** - Device connection status and credentials retrieval
- **Async Notifications** - Real-time notification support

![CPE Snapshot](./docs/screenshots/cpesnapshot.png)

### Open API
- **RESTful API** - Complete HTTP API for third-party system integration
- **Multi-tenant** - Independent tokens with permission isolation
- **Task Query** - Real-time task execution status tracking


---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Spring Boot + MyBatis |
| Database | MySQL 5.7 |
| Cache | Redis |
| Web Server | Nginx |
| Protocol | TR-069 (CWMP)|
| Security | HTTPS, Token Auth |

---

## Requirements

- **OS**: CentOS 7+ / Linux
- **JDK**: 1.8+
- **MySQL**: 5.7+ (set `lower_case_table_names=1`)
- **Redis**: 3.0+
- **Memory**: 4GB+ recommended
- **Disk**: 100GB+ based on device scale

## API Examples

### Configure Device Parameters
```bash
POST /api/task/setParameterValues
Content-Type: application/json
X-Token: {your-api-token}

[
  {
    "serialNumber": "DEVICE_SN_001",
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

### Reboot Device
```bash
POST /api/task/reboot
Content-Type: application/json

[
  {
    "serialNumber": "DEVICE_SN_001"
  }
]
```

### Firmware Upgrade
```bash
POST /api/task/firmwareUpgrade
Content-Type: application/json

{
  "name": "firmware_upgrade_v1",
  "scheduleTime": "",
  "firmware": {
    "downloadUrl": "http://your-server/firmware/v2.0.0.bin",
    "downloadFileName": "v2.0.0.bin",
    "downloadFileSize": "5242880"
  },
  "serialNumbers": ["DEVICE_SN_001", "DEVICE_SN_002"]
}
```

### Query Task Status
```bash
GET /api/task/query?requestToken={requestToken}
```

---

## Use Cases

- **ISP Home Gateway Management** - ONT, router batch configuration and O&M
- **Enterprise Network Management** - Unified management of APs, switches, gateways
- **Smart Community/Parks** - Large-scale IoT device orchestration
- **Industrial IoT (IIoT)** - Remote device monitoring and configuration
- **OSS/BSS Integration** - API integration with existing management systems

---

## Contact

- **GitHub Issues**: [https://github.com/luckyshine2026/acscloud/issues](https://github.com/luckyshine2026/acscloud/issues)
- **Business Inquiries**: Contact via GitHub
- **Emal Contact**: yanhongmei197710@gmail.com, shengchuan1@gmail.com

---

## License

This project is for authorized users only. For commercial licensing, please contact the author.

---

## Project Structure

```
acscloud/
├── README.md              # This file
├── init.sql              # Database initialization
├── docs/                 # Documentation
│   ├── screenshots/      # Screenshots
│   ├── api.md            # API documentation
│   └── troubleshooting.md # FAQ
```
