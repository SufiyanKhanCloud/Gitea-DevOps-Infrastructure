# **04 - Disaster Recovery: Split-Phase Vaulting & Ransomware Protection**

## **The Architectural Bottleneck: File Locking**

In standard deployment environments, attempting to back up a live Git instance using traditional file-copy utilities (`robocopy` or `zip`) consistently results in database fracturing. Because the OS locks active queue and index files during developer commits, standard backup scripts either crash or capture corrupted states.

To guarantee 100% data integrity with **Zero Downtime**, I engineered a **Split-Phase Disaster Recovery Engine** that decouples the database snapshot from the filesystem archiving.

## **Phase 1: Relational Integrity (Live SQL Snapshot)**

Because the infrastructure was migrated to MS SQL Server 2022 (see Module 01), the backup engine no longer relies on file copying for the database.

* A PowerShell automation triggers `sqlcmd` to execute a native, live database snapshot (`.bak`).
* This completely bypasses OS file-locking, ensuring that teams, webhooks, and RBAC permissions are captured flawlessly while developers are actively pushing code.

## **Phase 2: Ransomware Air-Gapping & Vaulting**

Traditional enterprise backups often rely on SMB mapped network drives (e.g., a `Z:\` drive). However, in the event of a ransomware breach, malware traverses mapped drives and encrypts the backups.

To establish a **Zero-Trust Storage Architecture**, I deprecated all persistent mapped drives and implemented an **Isolated Programmatic Vaulting Strategy**:

1. **Intelligent Staging:** The script uses `robocopy /XF LOCK` to mirror the application state to a temporary staging directory, gracefully skipping transient lock files.
2. **Encapsulation:** The payload is compressed into a single, date-stamped archive via `Compress-Archive`.
3. **Air-Gapped Transmission:** Utilizing `System.Net.WebClient`, the script opens a temporary FTP pipeline to an off-site Linux vault, injects credentials directly from memory, transmits the payload, and instantly severs the connection.

*Result: The production server maintains no persistent OS-level access to the backup vault. If the host is compromised, the ransomware has no network path to the archives.*

### **Automation: Service-Level Task Scheduling**

The entire engine is orchestrated via Windows Task Scheduler, elevated from a "user script" to a persistent background service:

* **Security Context:** Configured to execute as `SYSTEM` ("Run whether user is logged on or not"), allowing the engine to survive server reboots without requiring an active RDP session.
* **Execution Policy:** Utilized `-ExecutionPolicy Bypass` in the task arguments to allow the automation to run while keeping the global OS execution policy strictly locked down.

## **Data Lifecycle & Retention Governance**

To optimize storage I/O and prevent disk bloat, the engine enforces a strict self-cleaning lifecycle:

* **Local Staging:** 3-Day rolling retention (Immediate fast-recovery).
* **FTP Vault:** 7-Day rolling retention (Programmatically pruned via FTP `ListDirectoryDetails` and `DeleteFile` methods).

