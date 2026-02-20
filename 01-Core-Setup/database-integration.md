### **01-Core-Setup**

**File:** `database-integration.md`
**Content:**

 ## Gitea & MSSQL Integration
 

 This module covers the connection between the Gitea service and a Microsoft SQL Server backend.
 * **Database:** MSSQL 2019/2022.
 * **Connection Logic:** Configured Gitea to utilize a dedicated service account with `db_owner` permissions on the Gitea database.
 * **Configuration Snippet (app.ini):**
 
 

 ```ini
 [database]
 DB_TYPE = mssql
 HOST = localhost:1433
 NAME = gitea
 USER = gitea_admin
 PASSWD = ************
 
 ```
 Backup work in progress
 
---
