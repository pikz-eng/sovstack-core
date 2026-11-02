 sovstack-core

> Automated, reproducible, and sovereign edge infrastructure baseline designed for air-gapped environments.

`sovstack-core` provides a hardened, Infrastructure-as-Code (IaC) foundation for deploying and operating single-node appliances in environments where strict data isolation, zero outbound egress, and local governance are paramount (e.g., healthcare clinics, legal practices, and regulated SMEs).

This repository represents **Phase 1** of the sovStack ecosystem, establishing the declarative OS hardening, container orchestration, secret management, and network containment layer.

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
