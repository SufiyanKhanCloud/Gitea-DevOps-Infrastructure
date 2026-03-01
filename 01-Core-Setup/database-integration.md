# **01 - Core Setup: Gitea & MSSQL Integration**

## **Architecture Context & The Database Bottleneck**

By default, Gitea utilizes an embedded SQLite database. While sufficient for lightweight deployment, it creates a fatal bottleneck in a production environment: **OS File Locking**. During active developer commits, the SQLite file remains locked, causing automated backup scripts to fail or fracture the database.

To guarantee zero-downtime backups and high-concurrency performance, the database layer was fully decoupled and migrated to a dedicated Enterprise Relational Database.

## **Database Infrastructure**

* **Database Engine:** MS SQL Server 2022 Express
* **Database Name:** `gitea`
* **Connection Logic:** Gitea connects via a dedicated service account (`gitea_user`) mapped strictly with `db_owner` permissions on the target database, enforcing the principle of least privilege.

## **Master Configuration Snippet (`app.ini`)**

The following block demonstrates the production parameters used to securely bind the application to the MSSQL instance.

*Path: `D:\Gitea-Data\custom\conf\app.ini*`

```ini
[database]
DB_TYPE = mssql
HOST = 127.0.0.1:1433
NAME = gitea
USER = gitea_user
PASSWD = [REDACTED - See Password Vault]
SCHEMA = 
SSL_MODE = disable
LOG_SQL = false

```
