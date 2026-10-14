# sp_WhoIsActive (Customized)

This repository contains a customized version of the famous `sp_WhoIsActive` stored procedure originally created by **Adam Machanic**. 

While the original script is an indispensable tool for SQL Server DBAs, this customized version introduces additional features focused on system metrics, SQL Agent job visibility, and client connection details, all while preserving the original script's performance and integrity.

## 🚀 Customizations & New Features

### 1. System Metrics (`@IncludeSystemMetrics`)
A new parameter `@IncludeSystemMetrics BIT = 0` has been added. 
When set to `1`, the procedure returns an additional result set with high-level system health metrics, including:
- **CPU & Memory:** Total, Used, Free, Usage Percent, CPU Alerts.
- **Connections & Sessions:** Total connections, User connections, Sleeping with transactions, Total sleeping.
- **Active Queries:** Number of active rows, active queries.
- **Top N:** Top Application, Top User, Top Host (by connection count).
- **Performance Counters:** Page Life Expectancy, Buffer Cache Hit Ratio.
- **Automatic Alerts:** `OK`, `WARNING`, `CRITICAL`, `EMERGENCY` based on CPU and Connection thresholds.

### 2. SQL Agent Job Name Enhancement
By default, the `program_name` column for SQL Agent jobs displays a hex string (e.g., `SQLAgent - TSQL JobStep (Job 0x... : Step 1)`). 
This customization automatically resolves the hex ID and step number to the actual, readable **Job Name** and **Step Name** (e.g., `MyBackupJob (Step 1)`). 
*Note: This feature requires read access to the `msdb` database.*

### 3. Client IP Address (`[client_ip]`)
A new output column `[client_ip]` has been added to display the client's network address.
- **Source:** Extracted from `sys.dm_exec_connections.client_net_address`.
- **Empty String instead of `<local machine>`:** To keep the output clean, if the connection is using Shared Memory (local machine), the script returns an empty string (`''`) instead of the default `<local machine>`.
- **Performance Optimized:** The `JOIN` to `sys.dm_exec_connections` is **only** included in the dynamic SQL if `[client_ip]` is explicitly requested in the `@output_column_list`. This ensures that if you don't need the IP, the procedure runs as fast as the original.
- **Sorting:** You can now use `[client_ip]` in the `@sort_order` parameter.

## 🛠️ Installation

1. Download the `sp_whoisactive.sql` script from this repository.
2. Open the script in SQL Server Management Studio (SSMS) or Azure Data Studio.
3. Connect to your SQL Server instance (preferably in the `master` database).
4. Execute the script to create or alter the `sp_WhoIsActive` stored procedure.

**Permissions Required:**
- `VIEW SERVER STATE` (to query DMVs).
- Read access to `msdb` (for the Job Name enhancement feature).

## 📖 Usage Examples

### Basic Usage with Client IP
To see the client IP address alongside the standard output, simply include `[client_ip]` in the `@output_column_list`:

```sql
EXEC dbo.sp_WhoIsActive 
    @output_column_list = '[dd hh:mm:ss.mss],[session_id],[login_name],[host_name],[client_ip],[program_name],[database_name],[status],[wait_info],[sql_text]',
    @get_plans = 1,
    @get_outer_command = 1,
    @find_block_leaders = 1;
