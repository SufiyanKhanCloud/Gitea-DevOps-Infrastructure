# **05 - Enterprise Multi-Tenant Architecture & RBAC**

## **Architectural Overview: Tenant Isolation**

To support enterprise-grade collaboration and ensure strict data compliance, the infrastructure was migrated from a "flat" user-repository model into a strict **Multi-Tenant Organization** hierarchy.

This architecture guarantees cryptographically and logically isolated data silos, ensuring that distinct client sectors (e.g., Financial Services vs. Healthcare) maintain zero cross-project visibility.

## **The Logical Segregation Strategy**

I implemented dedicated Organizational units to enforce granular access control and prevent unauthorized data leakage.

| Organization Engine | Sector Focus | Architectural Logic & Compliance |
| --- | --- | --- |
| **`Org-Fintech`** | Banking & Financial Systems | Strict logical isolation for high-sensitivity financial IP and transaction schemas. |
| **`Org-Healthcare`** | Medical & Patient Systems | Compliance-focused isolation to protect health-tech integrations. |
| **`Org-Support`** | Internal Operations | Centralized management for DevOps runbooks, scripts, and internal tooling. |

## **Security Implementation: Role-Based Access Control (RBAC)**

Transitioning the repositories required implementing a "Zero-Trust" internal access model utilizing the **Principle of Least Privilege (PoLP)**:

1. **Ownership Migration:** Utilized native Gitea transfer protocols to migrate legacy repositories into the new Organizational silos without fracturing active commit histories or branch protections.
2. **Team-Level Isolation:** Engineered dedicated "Developer" and "DevOps" teams within each Organization. Users are dynamically assigned visibility only to the specific projects provisioned to their team.
3. **Granular Permissions:** Deprecated universal "Admin" rights. Daily operational accounts are restricted to **Write** access, while global infrastructure modifications are restricted to a dedicated, vaulted administrative service account.

## **Infrastructure Optimization for Data Hygiene**

To support the massive data loads common in Fintech/Healthcare environments:

* **Large File Passthrough:** Synchronized IIS reverse proxy configurations with Gitea's `app.ini` to allow up to **512MB** payload migrations (e.g., massive database schema snapshots).
* **Repository Decoupling:** Established engineering governance requiring developers to separate source code repositories from database artifact repositories, preventing Git index bloat and maintaining lightning-fast clone speeds.

---

### **File 2: `06-SSL-Orchestration.md**`

*(This replaces your SSL & Routing documentation)*

# **06 - Subdomain Routing & Automated SSL Orchestration**

## **1. Layer 7 Network Architecture**

To host the Gitea environment on a professional enterprise subdomain without exposing non-standard application ports (Port 3005) to the public web, a **Layer 7 Reverse Proxy** was engineered via Microsoft IIS.

**Traffic Flow Topology:**
`External User (HTTPS :443)` ➔ `IIS Reverse Proxy` ➔ `URL Rewrite Engine` ➔ `Gitea Native Service (HTTP :3005)`

## **2. IIS Reverse Proxy Configuration**

The gateway acts as the sole entry point, handling SSL decryption before passing raw traffic to the internal Go application.

### **Inbound URL Rewrite Logic**

An IIS Inbound Rule bridges the external request to the internal loopback:

* **Match:** `(.*)` (All incoming requests).
* **Action:** Rewrite to `http://127.0.0.1:3005/{R:1}`.
* **ACME Challenge Exclusion:** A critical exception rule was engineered for `/.well-known/acme-challenge/*`. This ensures that incoming SSL validation requests from the Certificate Authority are intercepted by the local ACME client and *not* forwarded into the Gitea application proxy, preventing renewal failures.

## **3. Cryptographic Security & SNI Implementation**

Securing a multi-tenant Windows Server requires advanced traffic shaping to prevent domain conflicts.

### **Server Name Indication (SNI)**

Because this server hosts multiple secure applications (e.g., HRMS, Git, Documentation) on a single IPv4 address, **SNI** was enforced on all Port 443 bindings.

* **The Engineering Fix:** SNI allows the IIS Gateway to inspect the incoming `Host Header` and present the correct SSL certificate during the TLS handshake *before* the HTTP request is fully processed. This entirely eliminates "Certificate Mismatch" errors across co-hosted domains.

### **Automated SSL Lifecycle (Let's Encrypt)**

* **Orchestrator:** `win-acme` (v2.x) Let's Encrypt Client.
* **Automation:** A background Windows Scheduled Task executes a daily health check, performing silent "zero-touch" background renewals 30 days prior to certificate expiration.
* **Persistence:** All cryptographic tools, private keys, and renewal registries are hosted on a dedicated data volume (`D:\`) to ensure the SSL infrastructure survives a primary OS (Drive `C:\`) failure or wipe.

## **4. Application-Level Proxy Synchronization**

The internal Gitea configuration was synchronized to seamlessly generate secure external assets (like clone URLs and avatar images) despite operating internally over HTTP.

*Path: `custom/conf/app.ini*`

```ini
[server]
PROTOCOL = http
DOMAIN = git.example.com
ROOT_URL = https://git.example.com/
LOCAL_ROOT_URL = http://127.0.0.1:3005/

```

## **5. Protocol Hardening**

* **HSTS Enforced:** **HTTP Strict Transport Security (HSTS)** was enabled at the IIS Gateway level. This forces all modern web browsers to interact with the repository strictly over encrypted HTTPS pipelines, completely neutralizing protocol downgrade attacks and MITM (Man-in-the-Middle) vulnerabilities.
