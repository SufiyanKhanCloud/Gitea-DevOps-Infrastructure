# IIS FTP Implementation Guide: Multi-Platform Access

This repository documents the implementation of a secure IIS FTP server designed to support native Windows File Explorer mapping and cross-platform clients like FileZilla.

## Project Objective

Configure an IIS FTP server to allow direct, folder-style access for remote team members while balancing user convenience with security best practices (NTFS permissions and firewall hardening).

---

## Phase 1: IIS FTP Configuration

To support native Windows File Explorer mapping, specific handshake protocols and passive data ports must be defined.

### 1. SSL Requirements

* **Setting:** Set **FTP SSL Settings** to **Allow SSL**.
* **Reason:** Windows File Explorer does not natively support explicit FTP over TLS for simple folder mapping. "Allow SSL" permits the standard connection required for this workflow.

### 2. Authentication & Authorization

* **Authentication:** Basic Authentication enabled; Anonymous Authentication disabled.
* **Authorization:** Access restricted to a dedicated service account (`FTP_User`) with **Read** and **Write** permissions.

### 3. Passive Port Configuration (Firewall Support)

1. Navigate to **FTP Firewall Support** at the Server level in IIS Manager.
2. Defined **Data Channel Port Range**: `5000-5100`.
3. Specified the **External IP Address** of the firewall.
4. **Note:** The `Microsoft FTP Service` must be restarted via `services.msc` to bind these port allocations.

---

## Phase 2: Security & Permissions

Layered security ensures that network-level access is backed by operating system-level restrictions.

### 1. NTFS Folder Permissions

* **Target Directory:** `D:\Project_Root`
* **Configuration:** Granted `FTP_User` specific permissions: **Modify**, **Read & Execute**, **List Folder Contents**, and **Write**.

### 2. Windows Defender Firewall

* **Inbound Rules:**
* **Control Channel:** TCP Port `21`
* **Data Channel:** TCP Ports `5000-5100`


* **Scope:** Configured to allow remote access for team members.
* **Security Note:** Exposure is mitigated by using a dedicated, non-privileged service account rather than administrative credentials.

---

## Phase 3: Client Connection Methods

### Method A: Windows File Explorer (Native)

Ideal for Windows users who prefer interacting with the remote server as a local directory.

1. Input the following URI into the File Explorer address bar:
`ftp://<USER>:<PASSWORD>@<SERVER_IP>`
2. **UX Tip:** Pin the resulting directory to **Quick Access** for persistent mapping.

### Method B: FileZilla (Cross-Platform)

Recommended for Linux users (Pop!_OS) or robust file management.

* **Protocol:** FTP - File Transfer Protocol
* **Encryption:** Only use plain FTP (insecure)
* **Logon Type:** Normal
* **Transfer Mode:** Use **Passive** for remote connections; use **Active** for internal loopback testing.

---

## Technical Lessons Learned

* **Root Folder Behavior:** Native Windows FTP mapping drops users directly into the physical path's root. If the directory is empty, the interface appears blank, which can be misidentified as a connection timeout.
* **Protocol Constraints:** The trade-off between encryption (TLS) and native OS compatibility (File Explorer) requires compensating controls at the network and folder permission levels.
* **Service Binding:** IIS FTP firewall changes do not take effect until a full service restart of the `Microsoft FTP Service`, regardless of IIS site status.

---

