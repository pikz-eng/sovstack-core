# sovstack-core

## Introduction

**sovstack-core** gives you a simple way to set up a secure, air-gapped computer server for your local network. 

That means your data and applications stay 100% with you, completely isolated from the public internet and big tech cloud services.

- Your system runs on your own hardware using **AlmaLinux OS**.
- It automatically installs **K3s** to run container apps, **HashiCorp Vault** to keep your passwords and secrets safe, and an encrypted **VPN** to connect safely.
- This code provides automated scripts (using Terraform and Ansible) to set up everything from scratch in a few minutes, without needing to manually configure each piece.

---

## Target Use Cases & Why It Matters

Building and running modern software—especially **local AI models (LLMs), automation agents, and sensitive business applications**—usually requires a complex, error-prone setup of security certificates, firewalls, and orchestrators. 

`sovstack-core` removes this barrier by providing a secure, turnkey foundation tailored for:
* **AI & Edge Developers:** Deploy local AI workloads and services on-premise without risking data privacy or leaking data to public cloud providers.
* **SMEs and Medical Clinics:** Run private, sovereign local infrastructure that complies with strict privacy and data-control requirements.
* **System Administrators:** Spin up a fully automated, production-grade AlmaLinux + K3s + Vault environment with a single command.

---

## Architecture Overview

```mermaid
flowchart LR
    Admin[Admin Node<br>Terraform & Ansible] -- Encrypted VPN Tunnel --> EdgeGW[Edge VPN Gateway]

    subgraph "Air-Gapped Sovereign Environment"
        EdgeGW --> K3s[K3s Kubernetes Cluster]
        K3s --> Vault[(HashiCorp Vault<br>Secrets Management)]
        K3s --> Core[Infra Workloads & AI Ready]
    end

    style Admin fill:#f5f5f5,stroke:#333,stroke-width:2px
    style EdgeGW fill:#ffe6cc,stroke:#d79b00,stroke-width:2px
    style K3s fill:#d5e8d4,stroke:#82b366,stroke-width:2px
    style Vault fill:#e1d5e7,stroke:#9673a6,stroke-width:2px
---

```mermaid
flowchart TD
    %% ==========================================
    %% TOP ZONE: MANAGEMENT & CONTROL PLANE
    %% ==========================================
    subgraph ManagementPlane ["MANAGEMENT & PROVISIONING PLANE (Out-of-Band)"]
        AdminStation["SysAdmin / Operator Workstation"]
        TerraformEngine["Terraform Engine<br>(KVM/Proxmox Provisioner)"]
        AnsibleEngine["Ansible Automation Engine<br>(OS Hardening & Bootstrap)"]
        
        AdminStation --> TerraformEngine
        AdminStation --> AnsibleEngine
    end

    %% ==========================================
    %% DEMARCATION BOUNDARY: AIR-GAP PERIMETER
    %% ==========================================
    subgraph PerimeterBoundary ["------------------ AIR-GAP & ZERO-EGRESS SECURITY BOUNDARY ------------------"]
        WGGateway["WireGuard VPN Gateway (wg0)<br>Port 51820/UDP • Key-Auth Only"]
        NFTables["nftables Policy Engine<br>• Ingress: Strict wg0 + LAN<br>• Egress: DEFAULT DROP (0.0.0.0/0)"]
    end

    %% ==========================================
    %% BOTTOM ZONE: SOVEREIGN ON-PREMISES APPLIANCE
    %% ==========================================
    subgraph ApplianceNode ["SOVEREIGN APPLIANCE NODE (AlmaLinux 9 Minimal • SELinux Enforcing)"]
        
        subgraph OrchestrationLayer ["K3s Lightweight Container Engine (containerd)"]
            TraefikIngress["Traefik Ingress Router<br>(Internal mTLS Endpoint)"]

            subgraph SecurityOrchestrator ["Security & Secrets Management"]
                VaultServer[("HashiCorp Vault<br>(Raft Consensus Engine)")]
                VaultPKI["Vault Internal PKI<br>(Zero-Internet TLS CA)"]
                VaultServer --- VaultPKI
            end

            subgraph ComputeWorkloads ["Compute & System Workloads"]
                CoreWorkload["Sovereign Core Services"]
                SidecarAgent["Vault Injector Agent<br>(tmpfs in-memory delivery)"]
                CoreWorkload --- SidecarAgent
            end
            
            TraefikIngress -->|Authenticated Route| CoreWorkload
            SidecarAgent -.->|Fetch Ephemeral Secrets| VaultServer
        end

        subgraph StorageLayer ["Dedicated Host Storage Architecture"]
            LocalCSI["K3s Local-Path CSI Driver"]
            NVMeArray[("High-Performance Local NVMe / SSD<br>[/var/lib/rancher/k3s/storage]")]
            LocalCSI --> NVMeArray
        end

        ComputeWorkloads -->|Persistent Mount| LocalCSI
    end

    %% Inter-plane connections
    TerraformEngine ==>|1. Provision VM/Node| ApplianceNode
    AnsibleEngine ==>|2. Configure Hardening via VPN| WGGateway
    WGGateway -->|3. Decrypted Internal Traffic| NFTables
    NFTables -->|4. Authorized Internal Ingress| TraefikIngress

    %% ==========================================
    %% ENTERPRISE COLOR PALETTE (HPE / GreenLake style)
    %% ==========================================
    style ManagementPlane fill:#f4fbf9,stroke:#00a982,stroke-width:2px,stroke-dasharray: 5 5
    style AdminStation fill:#ffffff,stroke:#00a982,stroke-width:2px
    style TerraformEngine fill:#ffffff,stroke:#00a982,stroke-width:2px
    style AnsibleEngine fill:#ffffff,stroke:#00a982,stroke-width:2px

    style PerimeterBoundary fill:#fff9f0,stroke:#e67e22,stroke-width:2px
    style WGGateway fill:#ffffff,stroke:#e67e22,stroke-width:2px
    style NFTables fill:#ffeef0,stroke:#d9534f,stroke-width:2px

    style ApplianceNode fill:#f0f7fc,stroke:#0070ba,stroke-width:2px
    style OrchestrationLayer fill:#ffffff,stroke:#0070ba,stroke-width:2px
    style TraefikIngress fill:#e8f4f8,stroke:#0070ba,stroke-width:1px
    style SecurityOrchestrator fill:#f9f4fc,stroke:#6f42c1,stroke-width:2px
    style VaultServer fill:#ffffff,stroke:#6f42c1,stroke-width:2px
    style ComputeWorkloads fill:#f4fbf9,stroke:#28a745,stroke-width:2px
    style StorageLayer fill:#fdfefe,stroke:#495057,stroke-width:2px
```
