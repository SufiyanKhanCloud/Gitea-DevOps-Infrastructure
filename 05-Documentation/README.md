# Technical Documentation & Architecture
# Gitea-DevOps-Infrastructure
 

 A production-grade, self-hosted Git infrastructure designed for secure banking repository management.
 ### Infrastructure Overview
 
 
 * **Server:** Intel Core i3-4005U | 10GB RAM
 * **OS:** Windows Server with WSL integration
 * **Stack:** Gitea (Version 1.25.4), MSSQL, IIS Reverse Proxy
 
 
 ### Accomplishments
 
 
 * Successfully migrated 5+ banking repositories from GitHub to private Gitea organizations.
 * Solved the "Large File Payload" issue by synchronizing Gitea and IIS request limits to **512MB**.
 * Developed an automated backup-to-FTP pipeline.

# Gitea Infrastructure: Multi-Tenant Organization & Migration Strategy

##  Overview
To support enterprise-grade collaboration and data isolation, I migrated the internal Gitea infrastructure from a personal-user model to a **Multi-Tenant Organization** structure. This ensures strict separation between different client sectors (Fintech, Healthcare, etc.).

##  The Organizational Structure
I implemented a structure based on **Organizations** to allow for granular access control and prevent unauthorized cross-project visibility.

| Organization | Purpose | Logic |
| :--- | :--- | :--- |
| **Org-Fintech** | Banking & Financial Systems | Strict isolation for sensitive financial data. |
| **Org-Healthcare** | Medical & Patient Systems | Compliance-focused isolation for healthcare tools. |
| **Org-Support** | Internal Operations | Management of internal ticketing and support tools. |

##  Migration & Security Process

### 1. Ownership Transfer
Used Gitea’s **Transfer Ownership** feature to migrate existing repositories into their respective Organizations. This preserved all commit history and branching structures while updating the security hierarchy.

### 2. Role-Based Access Control (RBAC)
Implemented a "Least Privilege" security model:
* **Team Isolation:** Created specialized "Developer" teams within each Organization.
* **Granular Permissions:** Assigned **Write** access for daily operations while restricting **Administrative** rights to a primary/secondary admin for system stability.
* **Visibility:** Configured settings so users only see projects explicitly assigned to their team.

### 3. Infrastructure Optimization
* **Large File Support:** Optimized the reverse proxy (IIS) and Gitea `app.ini` limits to support large database migrations (up to 800MB).
* **Dedicated Data Repos:** Established a practice of separating source code from database schema snapshots to maintain repository performance.

##  Verification
- [x] Verified user-level isolation through cross-account testing.
- [x] Confirmed high-capacity file uploads through the optimized reverse proxy.
- [x] Established a 7-day rolling backup retention policy to optimize local storage.
