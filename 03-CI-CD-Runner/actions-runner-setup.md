# **03 - CI/CD Pipeline: Gitea Actions (Host Mode)**

## **The Resource Constraint & Architecture Pivot**

In modern DevOps, the default industry advice is often to "containerize everything" using Docker-in-Docker (DinD) architectures for CI/CD runners. However, applying this default to a hardware-constrained environment (10GB RAM) creates severe memory pressure and deployment bottlenecks.

Instead of over-provisioning hardware, I engineered the `act_runner` pipeline to execute in **Host Mode**.

## **Host-Mode Execution Logic**

By bypassing virtualization and running jobs directly on the Windows Server host via PowerShell, we achieved three critical optimizations:

1. **Zero Virtualization Overhead:** Eliminated the CPU/RAM tax required to spin up and tear down Linux VMs or Docker containers for every minor code push.
2. **Instant Execution:** Workflows trigger instantly using the host's native environment variables and binaries (Git, Node.js).
3. **Hardware Efficiency:** Maintained baseline server performance, ensuring the IIS Proxy and MSSQL database were never starved of memory during heavy CI/CD build steps.

## **Service Persistence (NSSM)**

To ensure the runner remains continuously available without requiring an active Remote Desktop (RDP) session, the binary was wrapped as a persistent Windows background service using **NSSM (Non-Sucking Service Manager)**.

### **Core Configuration Snippet (`config.yaml`)**

*This snippet demonstrates the specific label routing used to force workflows into Host Mode, bypassing the default Docker daemon requirements.*

```yaml
runner:
  # Enterprise Optimization: Direct host execution mapping
  labels:
    - "windows-latest:host"
    - "native-ps:host"
  
  # Fetch interval for checking new CI/CD jobs
  fetch_interval: 2s

  # Logging configuration for audit trails
  log:
    level: info

```

### **Service Installation Command**

*The command used to register the background agent to the OS.*

```powershell
nssm install gitea-runner "D:\Gitea-Runner\act_runner.exe" daemon
nssm set gitea-runner AppDirectory "D:\Gitea-Runner"
nssm start gitea-runner

```

## **Security Note: PATH Management**

Because the runner operates as the `SYSTEM` account, strict environment variable (`PATH`) hygiene was enforced. Only explicitly trusted enterprise binaries (e.g., MSBuild, Node, Git) are accessible to the runner, preventing arbitrary code execution outside of the project scope.

## **Next Steps & Integration**

With the database stable, the gateway secured, and the CI/CD pipeline optimized, the final step is ensuring zero-downtime disaster recovery.
