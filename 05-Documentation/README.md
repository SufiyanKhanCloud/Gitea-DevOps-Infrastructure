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

### **Gitea-Infrastructure-SSL.md**

# 🚀 Gitea Infrastructure: Subdomain Routing & SSL Orchestration

This repository documents the production-grade deployment of a Gitea instance on a Windows Server environment using a **Reverse Proxy** and **Automated SSL Lifecycle Management**.

## 🏗️ 1. Network Architecture

To host Gitea on a professional subdomain without exposing non-standard ports, a **Layer 7 Reverse Proxy** was implemented.

### **Traffic Flow**

`User (HTTPS)` → `IIS (Port 443)` → `URL Rewrite` → `Gitea (Port 3005)`

### **Subdomain Logic**

* **DNS Configuration:** An `A Record` was created for the `git` subdomain, pointing to the server's public IP.
* **Host Header Routing:** IIS was configured to use **Host Headers**. This allows the server to distinguish between multiple subdomains on a single IP address, routing traffic to the correct internal service based on the incoming URL.

---

## 🔄 2. IIS Reverse Proxy Configuration

Gitea runs as a background service on a local loopback port. IIS acts as the "Gateway."

### **URL Rewrite Rules**

An inbound rule was established to bridge external traffic to the internal Go application:

* **Match:** All incoming requests `(.*)`.
* **Action:** Rewrite to `http://127.0.0.1:3005/{R:1}`.
* **ACME Exclusion:** A specific condition was added to exclude `/.well-known/acme-challenge/*` from the rewrite. This ensures the SSL Certificate Authority can verify domain ownership without being redirected into the Gitea application.

---

## 🔒 3. SSL/TLS & SNI Implementation

Securing a multi-site environment required advanced **Server Name Indication (SNI)** settings.

### **Automated SSL (Let's Encrypt)**

* **Tool:** `win-acme` (v2.x)
* **Lifecycle:** Automated via Windows Scheduled Tasks (60-day renewal cycle).
* **Storage:** The tool and certificates are persisted on a secondary data drive (`D:\`) to ensure portability and survival of system drive wipes.

### **Multi-Certificate Conflict Resolution (SNI)**

Since the server hosts multiple secure sites, **SNI** was enabled on all Port 443 bindings.

* **The Solution:** By enabling SNI, the server can present the correct SSL certificate during the TLS handshake *before* the HTTP request is fully processed. This allows the co-existence of Let's Encrypt certificates and other third-party certificates (ZeroSSL, etc.) on the same IP.

---

## ⚙️ 4. Application-Level Sync

The Gitea configuration was synchronized to recognize its proxied environment.

**File:** `app.ini`

```ini
[server]
PROTOCOL = http
DOMAIN = git.example.com
ROOT_URL = https://git.example.com/
LOCAL_ROOT_URL = http://127.0.0.1:3005/

```

* **Protocol:** Set to `http` for internal communication.
* **Root URL:** Set to `https` to ensure the UI generates secure links for Git clones and assets.

---

## 🛠️ 5. Maintenance & Security Hygiene

* **Certificate Store:** Expired and conflicting certificates were manually purged from the **Personal** and **Web Hosting** stores (`certlm.msc`) to prevent "Certificate Mismatch" errors.
* **Hardening:** **HSTS (HTTP Strict Transport Security)** was enabled in IIS to force all browser connections to stay on the encrypted path.
* **Renewal:** A daily cron-style task checks for certificate health and performs silent renewals 30 days before expiration.

---

### **Summary of Achievement**

* ✅ Secured Gitea via **HTTPS/TLS 1.2+**.
* ✅ Implemented **SNI** for multi-tenant certificate hosting.
* ✅ Automated the 90-day renewal lifecycle.
* ✅ Decoupled app configuration from infrastructure via **Reverse Proxy**.

---

**Next Step:** You can now create a new repository on GitHub named `gitea-infrastructure-docs` and paste this content into the `README.md`.

**Would you like me to help you create a "Health Check" script that pings your Gitea API to make sure the site is up?**
