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

## 🌐 Networking & Routing Logic
* **Host Header Routing:** Configured IIS to inspect incoming Host Headers, allowing `git.domain.com` to be isolated from other web services on the same host.
* **ACME Challenge Pathing:** Crafted a specific URL Rewrite exclusion for `/.well-known/acme-challenge/` to ensure seamless certificate renewals without interfering with the Gitea application proxy.
* **Zero-Touch Maintenance:** A daily Windows Scheduled Task handles certificate health checks and performs "silent renewals" 30 days prior to expiry.
* **Storage Persistence:** All SSL tools, renewal logs, and configuration data are hosted on a persistent data drive (**D:**) to ensure infrastructure resilience against system-drive failures.

## Key Accomplishments
* **Enterprise Migration:** Successfully migrated 5+ high-sensitivity banking repositories from GitHub to private Gitea organizations.
* **Payload Optimization:** Synchronized IIS and Gitea request limits to support **512MB** uploads, verified with a 100 MiB test file.
* **Secure Access Control:** Provisioned fine-grained access for team members (e.g., `Ameen.dev`) using Private Organizations to prevent data leakage.
