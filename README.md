## Script: Get_Log_Files_Space_For_Shrink.sql

**Description**:
This script uses `DBCC SQLPERF(LOGSPACE)` to retrieve the size and usage percentage of **transaction log files** across all databases. It calculates the **used space**, **free space**, and **free percentage** for each log file and sorts by largest free log space.

---

### 🔧 What It Does:
- Captures log file size and usage using `DBCC SQLPERF(LOGSPACE)`
- Calculates:
  - Used log space (MB)
  - Free log space (MB)
  - Free percentage
- Helps identify:
  - Logs that are nearly full (possible growth risk)
  - Over-provisioned logs with too much unused space

---

### 💡 Query:
```sql
CREATE TABLE #LogStats (
    DatabaseName SYSNAME,
    LogSizeMB DECIMAL(10,2),
    LogUsedPercent DECIMAL(5,2),
    Status INT  
);

INSERT INTO #LogStats
EXEC('DBCC SQLPERF(LOGSPACE)');

SELECT 
    DatabaseName,
    LogSizeMB,
    CAST(LogSizeMB * LogUsedPercent / 100.0 AS DECIMAL(10,2)) AS UsedLogMB,
    CAST(LogSizeMB * (1 - LogUsedPercent / 100.0) AS DECIMAL(10,2)) AS FreeLogMB,
    CAST((1 - LogUsedPercent / 100.0) * 100.0 AS DECIMAL(5,2)) AS FreePercent
FROM 
    #LogStats
ORDER BY 
    FreeLogMB DESC;

DROP TABLE #LogStats;
```

---

### 📋 Output Columns:

| Column         | Description                                       |
|----------------|---------------------------------------------------|
| `DatabaseName` | Name of the database                              |
| `LogSizeMB`    | Total size of the transaction log file in MB      |
| `UsedLogMB`    | Approximate used portion of the log file in MB    |
| `FreeLogMB`    | Approximate free space available in log file      |
| `FreePercent`  | Percentage of log space that is still free        |

---

### ✅ Use Case:
- Monitor transaction log growth
- Detect log usage spikes
- Plan for log backups, shrink operations, or autogrowth configuration

---

### ⚠️ Notes:
- `DBCC SQLPERF(LOGSPACE)` is lightweight and safe to run frequently
- Logs with **low free space** may cause performance or auto-grow issues
- Always **backup the transaction log** regularly in Full/Bulk-Logged recovery models


