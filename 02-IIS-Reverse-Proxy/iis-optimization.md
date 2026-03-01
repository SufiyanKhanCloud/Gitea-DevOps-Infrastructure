# **02 - IIS Reverse Proxy & Security Architecture**

## **The "Front Door" Gateway Concept**

Directly exposing an internal application engine (operating on Port 3005) to the public web is a severe security anti-pattern. To establish a professional, secure perimeter, I engineered a **Layer 7 Reverse Proxy** using Microsoft Internet Information Services (IIS).

This architecture abstracts the internal Gitea service, routes external traffic exclusively over Port 443 (HTTPS), and provides centralized control over SSL decryption and payload filtering.

## **Developer Experience (DX): Large Payload Optimization**

By default, IIS aggressively restricts incoming requests to 30MB, and standard public Git providers (like GitHub/Bitbucket) hard-cap files at 100MB.

To support our enterprise engineering teams working with large binary assets and SQL database dumps, I bypassed these default bottlenecks and synchronized the IIS/Gitea pipeline to support massive **512MB (536,870,912 bytes)** payloads without HTTP timeout failures.

### **Core Configuration Snippet (`web.config`)**

*This configuration was injected at the IIS Site Root to securely override the default `maxAllowedContentLength` filtering.*

```xml
<security>
    <requestFiltering>
        <requestLimits maxAllowedContentLength="536870912" />
    </requestFiltering>
</security>

```

*Validation: The proxy handshake and streaming buffer were successfully stress-tested via a continuous 100 MiB synthetic payload upload.*

## **Zero-Touch SSL & Multi-Tenant Security**

Securing the gateway required implementing several advanced cryptographic protocols to ensure persistence and compatibility:

* **Server Name Indication (SNI):** Implemented SNI to allow this Git environment to coexist on the same IP address as other corporate subdomains (e.g., HRMS) without causing TLS certificate clashing.
* **ACME Protocol Automation:** Deployed `win-acme` for zero-touch Let's Encrypt (R13) certificate lifecycle management. I engineered specific URL Rewrite exclusions for the `/.well-known/acme-challenge/` directory to guarantee silent background renewals.
* **Protocol Hardening:** Enforced **HSTS (HTTP Strict Transport Security)** at the IIS level to mandate HTTPS-only connections, neutralizing protocol downgrade attacks and man-in-the-middle (MITM) vulnerabilities.

## **Next Steps & Integration**

With the network gateway secured and payload limits optimized, the infrastructure is prepared to handle high-velocity code deployments.
