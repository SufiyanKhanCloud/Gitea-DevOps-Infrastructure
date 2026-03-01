# Gitea DevOps Infrastructure (Windows-Native 2026)
A production-grade, self-hosted Git environment designed for secure enterprise repository management.

## Infrastructure Overview
* **Hardware:** Intel Core i3-4005U | 10GB RAM
* **OS:** Windows Server with WSL2 Integration
* **Version Control:** Gitea v1.25.4
* **Database:** Microsoft SQL Server (MSSQL)
* **Web Gateway:** IIS Reverse Proxy

## Project Modules
This repository is organized into five core modules. Click the links below for detailed configurations:

1. [**Core Setup**](./01-Core-Setup/database-integration.md): MSSQL integration and Gitea service configuration.
2. [**IIS Reverse Proxy**](./02-IIS-Reverse-Proxy/iis-optimization.md): Handling 512MB large file payloads and SSL passthrough.
3. [**CI/CD Runner**](./03-CI-CD-Runner/actions-runner-setup.md): Host-mode Gitea Actions for local Windows automation.
4. [**Automated Backups**](./04-Automated-Backups/backup-strategy.md): SQL dumping and automated FTP vaulting strategy.
5. [**Project Documentation**](./05-Documentation/README.md): Full server lifecycle and maintenance logs.

##  Networking & Routing Logic
* **Host Header Routing:** Configured IIS to inspect incoming Host Headers, allowing `git.domain.com` to be isolated from other web services on the same host.
* **ACME Challenge Pathing:** Crafted a specific URL Rewrite exclusion for `/.well-known/acme-challenge/` to ensure seamless certificate renewals without interfering with the Gitea application proxy.
* **Zero-Touch Maintenance:** A daily Windows Scheduled Task handles certificate health checks and performs "silent renewals" 30 days prior to expiry.
* **Storage Persistence:** All SSL tools, renewal logs, and configuration data are hosted on a persistent data drive (**D:**) to ensure infrastructure resilience against system-drive failures.

## Key Accomplishments
* **Enterprise Migration:** Successfully migrated 5+ high-sensitivity banking repositories from GitHub to private Gitea organizations.
* **Payload Optimization:** Synchronized IIS and Gitea request limits to support **512MB** uploads, verified with a 100 MiB test file.
* **Secure Access Control:** Provisioned fine-grained access for team members (e.g., `Ameen.dev`) using Private Organizations to prevent data leakage.
* **SSL Automation:** Achieved 100% automated certificate renewal with persistence on a secondary data drive, ensuring zero manual intervention until May 2026 and beyond.
* **Multi-Tenant SSL:** Successfully resolved multi-site certificate conflicts by implementing SNI, allowing multiple secure subdomains to coexist on Port 443 with unique certificates.

---

# Gitea Enterprise Infrastructure (Windows-Native)

A production-grade, self-hosted DevOps ecosystem designed for secure, high-availability enterprise repository management and CI/CD automation.

## 🏗️ Architecture Overview

* **OS Environment:** Windows Server (Native Host Execution via NSSM)
* **Hardware Profile:** Resource-Optimized (Intel Core i3 | 10GB RAM)
* **Version Control:** Gitea v1.25.4
* **Database Engine:** MS SQL Server 2022 Express (MSSQL)
* **Web Gateway:** IIS Layer 7 Reverse Proxy
* **CI/CD Pipeline:** Gitea Actions (Host Mode)

## 📂 Project Modules

This repository documents the complete infrastructure lifecycle. Click the modules below for detailed scripts, configurations, and runbooks:

1. **[Core Setup & MSSQL Migration](./01-Core-Setup/database-integration.md)**: Architecture for decoupling the database to eliminate file-locking bottlenecks under concurrent load.
2. **[IIS Reverse Proxy & Security](./02-IIS-Reverse-Proxy/iis-optimization.md)**: Layer 7 routing, SNI configuration, and handling 500MB+ large payload passthroughs.
3. **[CI/CD Runner (Host Mode)](https://www.google.com/search?q=./03-CI-CD-Runner/actions-runner-setup.md)**: Deploying native Windows automation to bypass Docker/VM overhead on constrained hardware.
4. **[Disaster Recovery Engine](https://www.google.com/search?q=./04-Automated-Backups/backup-strategy.md)**: Split-phase automated vaulting (SQL snapshot + Application staging) and FTP air-gapping.
5. **[Incident Response Runbook](https://www.google.com/search?q=./05-Documentation/README.md)**: Bare-metal restoration guides and system maintenance logs.

## 🌐 Networking & Security Logic

* **Multi-Tenant SNI:** Implemented Server Name Indication (SNI) on IIS to allow multiple secure subdomains (e.g., Git, HRMS) to coexist on Port 443 without certificate clashing.
* **Automated SSL Lifecycle:** Configured zero-touch Let's Encrypt (R13) certificates via `win-acme`. Crafted URL Rewrite exclusions for `/.well-known/acme-challenge/` to ensure silent background renewals.
* **Infrastructure Resilience:** All SSL tools, renewal logs, and Gitea binaries are hosted on a persistent, dedicated data drive (**D:**) to survive primary OS wiping or C: drive failures.

## 🚀 Key Engineering Accomplishments

* **Ransomware-Proof Vaulting:** Deprecated vulnerable mapped network drives (SMB) in favor of a programmatic, encrypted FTP pipeline. The server air-gaps its backups daily, ensuring 100% isolation from local ransomware infections.
* **Zero-Downtime Database Backups:** Migrated the backend from SQLite to MS SQL Server 2022. This eliminated OS file-locking crashes and allows the PowerShell engine to take live `sqlcmd` snapshots while developers are actively pushing code.
* **Developer Experience (DX) Optimization:** Benchmarked public cloud limits (GitHub/Bitbucket @ 100MB) and strategically synchronized IIS and Gitea request limits to support **500MB+ payloads**. This provides the engineering team 5x the flexibility for large binaries without forcing early Git LFS adoption.
* **Enterprise Access Control:** Successfully migrated high-sensitivity repositories into isolated, private Gitea Organizations with strict Role-Based Access Control (RBAC), preventing unauthorized data leakage across teams.
* **High-Performance CI/CD:** Engineered the CI/CD pipeline to run in "Host Mode." By executing workflows directly via Windows PowerShell rather than containerization, idle RAM overhead was reduced to near-zero, perfectly optimizing the 10GB hardware constraint.

---
