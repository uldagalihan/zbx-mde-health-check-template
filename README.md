# Microsoft Defender for Endpoint - Zabbix Health Check Template

## Zabbix 7.4 YAML Template: Microsoft Defender - Active Devices

This repository contains a **Zabbix 7.4 YAML template** designed for monitoring Microsoft Defender for Endpoint (MDE) devices. The template automatically gathers device health statuses, antivirus modes, and signature states directly from the Microsoft API, transforming them into trackable Zabbix metrics. 

## Features

* Uses a Zabbix **SCRIPT** master item to authenticate against the Microsoft Defender for Endpoint API via OAuth2 client credentials.
* Reconciles and merges disparate datasets by device/machine ID from multiple API endpoints.
* Normalizes Active/Blocked modes and signature states into human-readable labels.
* Executes Low-level discovery (LLD) using the machine's DNS name.
* Builds a clean, consolidated JSON output payload with global summary metrics and granular per-device status arrays.

## Requirements

* **Zabbix 7.4** or newer
* Network access to the Microsoft Defender for Endpoint API from the Zabbix Server or Zabbix Proxy

## Required API Permissions / Prerequisites

To authenticate with the Microsoft API, you must register an application in Microsoft Entra ID (Azure AD). Ensure the registered application possesses the following **Application permission**:
* `Machine.Read.All` (Microsoft Defender for Endpoint API)

## Macro Configuration

Configure the following macros on the host or template level. 

| Macro | Description | Default Value |
|-------|-------------|---------------|
| `{$MDE_TENANT_ID}` | Your Azure AD Tenant ID | *None* |
| `{$MDE_CLIENT_ID}` | App Registration Client ID | *None* |
| `{$MDE_CLIENT_SECRET}` | App Registration Client Secret | *None* |
| `{$MDE_RESOURCE}` | Resource URL for the MDE API | `https://api.securitycenter.microsoft.com` |
| `{$MDE_TOP_AVINFO}` | Top limit for the `/api/deviceavinfo` endpoint | `10000` |
| `{$MDE_TOP_MACHINES}` | Top limit for the `/api/machines` endpoint | `10000` |

## Import Instructions

1. Log into your Zabbix frontend.
2. Navigate to **Data collection** -> **Templates**.
3. Click the **Import** button in the top right corner.
4. Select the YAML template file from this repository.
5. Click **Import**.
6. Assign the **Microsoft Defender - Active Devices** template to a designated host representing your MDE environment.
7. Fill in the required host macros.

## How It Works

The master item runs a custom Zabbix JavaScript execution that performs securely authenticated requests against the MDE API. 

1. Retrieves data from `/api/deviceavinfo` and `/api/machines`.
2. Merges this data by the unique machine/device ID.
3. Automatically maps discovered endpoints by their `computerDnsName`.
4. Filters the device list to include only machines with specific `healthStatus` values:
   * `Active`
   * `NoSensorData`
   * `NoSensorDataImpairedCommunication`
5. The parsed information is compiled into a single JSON object divided into two segments: `summary` and `devices`.

**Data Mappings Implemented:**
* **AV Mode**: `0` => `Active`, `4` => `EDRBlocked`, `anything else` => `Other`
* **Signature State**: `"true"` => `True`, `"false"` => `False`, `anything else` => `Unknown`

## Discovered Per-Device Items

For every discovered device via Low-Level Discovery (LLD), Zabbix populates the following dependent items:
* `healthStatus`: Current health condition of the machine
* `signatureState`: Indicates whether the AV signatures are up to date 
* `avModeLabel`: Running mode of the antivirus
* `lastSeenTime`: Most recent communication timestamp with MDE
* `avSignatureUpdateTime`: Most recent signature definition update 

## Summary Metrics Exposed

Independent of per-device monitoring, global summary metrics are continually generated via dependent items:
* **Total Devices**: Total count of active/tracked devices.
* **AV Mode Breakdown**: Counts and percentages of machines in `Active`, `EDRBlocked`, and `Other` modes.
* **Health Status Breakdown**: Counts and percentages of machines in the tracked health states.
* **Signature State Breakdown**: Counts and percentages of machines grouped by signature state (`True`, `False`, `Unknown`).

## Notes and Limitations

* **Pagination**: API pagination is not implemented in this version. The template strictly relies on the `$top` limits natively passed through the OData query (`{$MDE_TOP_AVINFO}` and `{$MDE_TOP_MACHINES}`).
* **Missing DNS Names**: Discovered devices lacking a defined `computerDnsName` property are intentionally skipped by the master script. 
* **Health State Filtering**: Devices occupying health states outside of `Active`, `NoSensorData`, or `NoSensorDataImpairedCommunication` are excluded from the merged device lists.
* **Secret Macros**: The `{$MDE_TENANT_ID}`, `{$MDE_CLIENT_ID}`, and `{$MDE_CLIENT_SECRET}` macros **must** be stored as Zabbix "Secret text" macros preventing exposure in plaintext.
* **Zabbix 7.4 Requirement**: The data processing relies on capabilities standardized and tested against Zabbix 7.4.
* **Vanilla Template**: To maintain portability, this generic iteration does not contain pre-configured triggers, dashboard widgets, graphs, or webhook integrations.

## Example Repository Structure

```text
zbx-mde-health-check-template/
├── README.md
└── template_mde_active_devices.yaml
```

## Contributing

Contributions, issues, and feature requests are welcome! If you have optimized the internal JavaScript parsing, or implemented a robust pagination structure within Zabbix memory bounds, please feel free to submit a Pull Request.

## License

[Insert License Here] - Please review the LICENSE file for parameters regarding usage and distribution.
