# CC2025_32811_Dragotoniu_Corina_Part_2
# ETL Pipeline - Device Data Processing Solution

## Table of Contents
1. [Conceptual Architecture](#1-conceptual-architecture)
2. [UML Deployment Diagram](#2-uml-deployment-diagram)
3. [Build and Execution Steps](#3-build-and-execution-steps)

---

## 1. Conceptual Architecture

### Overview
This ETL pipeline processes device data from raw CSV files, transforms the data per device, and outputs structured JSON files with both current state and historical tracking capabilities.

### Architecture Diagram

```
┌─────────────────┐      ┌──────────────────────┐      ┌─────────────────────┐
│  Raw Blob       │      │  Process per Device  │      │  Processed Blob     │
│  Storage        │      │  (ETL Pipeline)      │      │  Storage            │
│                 │      │                      │      │                     │
│ • CSV files     │ ───> │ • Join data          │ ───> │ • /latest/          │
│ • Multi-device  │      │ • Aggregate          │      │ • /by-timestamp/    │
│ • Raw data      │      │ • Calculate metrics  │      │                     │
└─────────────────┘      └──────────────────────┘      └─────────────────────┘
         │                                                       │
         │ (trigger checks                                       │
         │  for new files)                                       v
         │                                              ┌─────────────────┐
         └──────────────────────────────────────────>   │ Logic App       │
                                                        │ (Email          │
                                                        │ Notification)   │
                                                        └─────────────────┘
```

### Components Description

#### 1. Raw Blob Storage (Input)
- **Purpose**: Store incoming raw CSV files
- **Content**: Multiple records from multiple devices
- **Format**: CSV with `device_id` column
- **Location**: Azure Blob Storage container

#### 2. ETL Pipeline (Transformation)
- **Service**: Azure Data Factory / Azure Synapse
- **Trigger**: Monitors raw blob storage for new files
- **Operations**:
  - Source data ingestion
  - Join operations across data streams
  - Aggregate data by device_id
  - Calculate derived columns (4 new metrics)
  - Split output per device

#### 3. Processed Blob Storage (Output)
- **Purpose**: Store processed JSON files
- **Structure**:
  - `/latest/` - Current device states (overwritten each run)
  - `/by-timestamp/` - Historical records (archived by timestamp)
- **Format**: JSON per device

#### 4. Logic App (Notification)
- **Purpose**: Send email notifications about pipeline status
- **Trigger**: Pipeline completion or failure
- **Action**: Send email to Microsoft Teams channel
- **Content**: Success/failure status with details

---

## 2. UML Deployment Diagram

```
╔════════════════════════════════════════════════════════════════════════╗
║                         Azure Cloud Environment                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────────┐   ║
║  │                    Azure Storage Account                        │   ║
║  │                                                                 │   ║
║  │  ┌────────────────────┐         ┌────────────────────┐          │   ║
║  │  │  Raw Blob          │         │  Processed Blob    │          │   ║
║  │  │  Container         │         │  Container         │          │   ║
║  │  │                    │         │                    │          │   ║
║  │  │  • device_data.csv │         │  • /latest/        │          │   ║
║  │  │  • raw files       │         │  • /by-timestamp/  │          │   ║
║  │  └────────────────────┘         └────────────────────┘          │   ║
║  └─────────────────────────────────────────────────────────────────┘   ║
║                │                              │                        ║
║                │ reads                        │ writes                 ║
║                v                              ^                        ║
║  ┌────────────────────────────────────────────────────┐                ║
║  │          Azure Data Factory                        │                ║
║  │                                                    │                ║
║  │  ┌───────────────────────────────────────────────┐ │                ║
║  │  │  ETL Pipeline: process_per_device             │ │                ║
║  │  │                                               │ │                ║
║  │  │  Components:                                  │ │                ║
║  │  │  • Blob Trigger (monitors raw container)      │ │                ║
║  │  │  • Data Flow:                                 │ │                ║
║  │  │    - source1                                  │ │                ║
║  │  │    - join1                                    │ │                ║
║  │  │    - aggregate1                               │ │                ║
║  │  │    - derivedColumn1                           │ │                ║
║  │  │  • Sinks:                                     │ │                ║
║  │  │    - historical (by-timestamp)                │ │                ║
║  │  │    - latest                                   │ │                ║
║  │  └───────────────────────────────────────────────┘ │                ║
║  └────────────────────────────────────────────────────┘                ║
║                                │                                       ║
║                                │ triggers on completion                ║
║                                v                                       ║
║  ┌──────────────────────────────────────────────────────┐              ║
║  │                   Azure Logic App                    │              ║
║  │                                                      │              ║
║  │  • Trigger: Pipeline completion/failure              │              ║
║  │  • Action: Send email notification                   │              ║
║  │  • Connector: Office 365 Outlook / Teams             │              ║
║  └──────────────────────────────────────────────────────┘              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Deployment Components

| Component | Type | Purpose |
|-----------|------|---------|
| Raw Blob Container | Azure Blob Storage | Input CSV files storage |
| Processed Blob Container | Azure Blob Storage | Output JSON files storage |
| ETL Pipeline | Azure Data Factory | Data transformation |
| Data Flow | ADF Data Flow | Per-device processing logic |
| Logic App | Azure Logic App | Email notification service |
---

## 3. Build and Execution Steps

### Prerequisites

- Azure subscription
- Azure Storage Account
- Azure Data Factory
- Azure Logic App
- Azure CLI installed (optional)
- Access permissions to create and manage Azure resources

### Setup Instructions

#### Step 1: Create Azure Storage Account

#### Step 2: Configure Azure Data Factory

1. **Create Data Factory**

2. **Create Linked Services**
   - Create linked service to Raw Blob Storage
   - Create linked service to Processed Blob Storage

3. **Create Pipeline: `process_per_device`**
   - Add blob trigger to monitor raw-data container
   - Configure trigger to activate on new file arrival

4. **Create Data Flow**
   - **Source**: `source1` → Read from raw-data container
   - **Join**: `join1` → Join data streams if multiple sources
   - **Aggregate**: `aggregate1` → Group by device_id
   - **Derived Column**: `derivedColumn1` → Calculate 4 new metrics
   - **Sinks**:
     - `historical` → Write to `/by-timestamp/{timestamp}/device-{id}.json`
     - `latest` → Write to `/latest/device-{id}.json`

#### Step 3: Configure Logic App

1. **Create Logic App**

2. **Configure Workflow**
   - **Trigger**: When a HTTP request is received (webhook from ADF)
   - **Action**: Send an email (Office 365 Outlook)
   - **Recipients**: Teams channel email address
   - **Email Body**: Include pipeline status, timestamp, and file details

3. **Connect to ADF Pipeline**
   - Add Web Activity in ADF pipeline (at end)
   - Configure webhook URL to Logic App trigger URL
   - Pass pipeline run status and details

#### Step 4: Prepare Sample Input Data

Create a sample CSV file `device_data.csv`:

```csv
device_id,timestamp,energy_kwh,location,status
DEV001,2024-01-15T10:00:00Z,12.5,Building-A,active
DEV001,2024-01-15T11:00:00Z,14.2,Building-A,active
DEV002,2024-01-15T10:00:00Z,8.7,Building-B,active
DEV002,2024-01-15T11:00:00Z,9.1,Building-B,active
DEV003,2024-01-15T10:00:00Z,15.3,Building-C,active
```

### Execution Steps

#### 1. Upload Input File

#### 2. Pipeline Triggers Automatically

- ETL pipeline detects new file in raw-data container
- Pipeline executes automatically
- Data flow processes records per device

#### 3. Monitor Pipeline Execution

Monitor via Azure Portal:
- Navigate to Data Factory
- Go to Monitor → Pipeline runs
- View execution details and logs

#### 4. Verify Output

Expected output structure:
```
processed-data/
├── latest/
│   ├── device-DEV001.json
│   ├── device-DEV002.json
│   └── device-DEV003.json
└── by-timestamp/
    └── 2024-01-15T14-30-00Z/
        ├── device-DEV001.json
        ├── device-DEV002.json
        └── device-DEV003.json
```

#### 5. Check Email Notification

- Open Email
- Verify email notification received
- Check status: Success or Failure

### Sample Output JSON

**File**: `processed-data/latest/device-DEV001.json`

```json
{
  "device_id": "DEV001",
  "generation_timestamp": "2024-01-15T14:30:00Z",
  "time_window": {
    "start": "2024-01-15T10:00:00Z",
    "end": "2024-01-15T11:00:00Z"
  },
  "records": [
    {
      "timestamp": "2024-01-15T10:00:00Z",
      "energy_kwh": 12.5,
      "location": "Building-A",
      "status": "active"
    },
    {
      "timestamp": "2024-01-15T11:00:00Z",
      "energy_kwh": 14.2,
      "location": "Building-A",
      "status": "active"
    }
  ],
  "summary": {
    "total_energy_kwh": 26.7,
    "average_energy_kwh": 13.35,
    "record_count": 2,
    "location": "Building-A"
  }
}
```

### Testing the Pipeline

1. **Test with valid data**

2. **Test with invalid data** (missing device_id)

3. **Verify notifications**
  - Check email for success message (valid data)
  - Check email for failure message (invalid data)

### Troubleshooting

#### Pipeline Not Triggering
- Verify blob trigger is enabled
- Check trigger configuration points to correct container
- Ensure storage account connection string is correct

#### No Output Files
- Check data flow execution logs
- Verify sink configurations
- Check permissions on processed-data container

#### No Email Notifications
- Verify Logic App trigger URL is correct in ADF
- Check Logic App run history for errors
- Verify Teams channel email address

#### Data Validation Errors
- Ensure CSV has required columns (device_id, timestamp)
- Check data types match expected schema
- Review data flow transformation logic

### Maintenance

- **Monitor costs**: Review Azure costs regularly
- **Clean old data**: Archive or delete old by-timestamp folders
- **Update logic**: Modify data flow as requirements change
- **Security**: Rotate storage account keys periodically

### Additional Resources

- [Azure Data Factory Documentation](https://docs.microsoft.com/azure/data-factory/)
- [Azure Logic Apps Documentation](https://docs.microsoft.com/azure/logic-apps/)
- [Azure Blob Storage Documentation](https://docs.microsoft.com/azure/storage/blobs/)

---

## Functional Requirements Summary

### Input Requirements
- Raw files in CSV format
- Multiple records from multiple devices
- device_id column for identification

### Processing Requirements
- ETL service processes data
- Trigger monitors for new data
- Per-device data transformation
- Join, aggregate, and derived column operations

### Output Requirements
- One JSON file per device per run
- `/latest/` folder with current state (overwritten)
- `/by-timestamp/` folder with historical records
- JSON includes: device_id, timestamp, time_window, records, summary

### Notification Requirements
- Logic App triggers on pipeline completion
- Email sent
- Success/failure status indicated
