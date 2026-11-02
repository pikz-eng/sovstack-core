 sovstack-core

> Automated, reproducible, and sovereign edge infrastructure baseline designed for air-gapped environments.

`sovstack-core` provides a hardened, Infrastructure-as-Code (IaC) foundation for deploying and operating single-node appliances in environments where strict data isolation, zero outbound egress, and local governance are paramount (e.g., healthcare clinics, legal practices, and regulated SMEs).

This repository represents **Phase 1** of the sovStack ecosystem, establishing the declarative OS hardening, container orchestration, secret management, and network containment layer.

---

 Architecture Overview
```mermaid
flowchart TD
    subgraph ManagementStation [" 🛠️ Administrator Station (Out-of-Band) "]
        Admin["SysAdmin / Operator"]
        IaC["IaC Tools: Ansible Playbooks & Terraform"]
        Admin --> IaC
    end

    subgraph Boundary [" 🌐 Network Isolation & Access Boundary "]
        WG["WireGuard VPN Interface (wg0)<br>Encrypted Transport / Port 51820 UDP"]
        NFT["nftables Firewall Engine<br>• Ingress: Only wg0 & LAN<br>• Egress: Strict DROP (Zero Internet Air-Gap)"]
    end

    subgraph HostOS [" 🛡️ Host Node: AlmaLinux 9 Minimal "]
        Kernel["Linux Kernel Hardened (sysctl tuning)"]
        SELinux["SELinux Enforcing Mode (k3s-selinux policies)"]
        LocalStorage["Host NVMe/SSD Storage<br>(/var/lib/rancher/k3s/storage)"]
        
        Kernel --- SELinux
    end

    subgraph K3sCluster [" ☸️ Single-Node K3s Container Runtime (containerd) "]
        Traefik["Traefik Reverse Proxy<br>(Internal TLS / Ingress Controller)"]
        
        subgraph SecNamespace [" Namespace: vault-system "]
            Vault["HashiCorp Vault<br>(StatefulSet / In-Memory tmpfs)"]
            VaultRaft[("Vault Integrated Storage<br>(Raft Engine on Local Disk)")]
            VaultPKI["Vault Internal PKI CA<br>(Automated Internal TLS Issuance)"]
            
            Vault --- VaultRaft
            Vault --- VaultPKI
        end

        subgraph WorkloadsNamespace [" Namespace: core-workloads "]
            CoreApp["Baseline Edge Workloads"]
            Injector["Vault Sidecar Agent<br>(Injects secrets via memory tmpfs)"]
            CSI["K3s Local-Path Provisioner (CSI)"]
            
            CoreApp --- Injector
            CoreApp --> CSI
        end

        Traefik -->|Internal Routing| CoreApp
        Traefik -->|Management API| Vault
        Injector -.->|Dynamic Secrets & MTLS| Vault
        CSI -->|Mount Volume| LocalStorage
    end

    %% Network flows
    IaC ==>|1. Encrypted Peer Handshake| WG
    WG -->|2. Internal Forwarding| NFT
    NFT -->|3. Filtered Ingress| Traefik

    %% Styling for crisp visual hierarchy
    style ManagementStation fill:#f8f9fa,stroke:#6c757d,stroke-width:2px
    style Boundary fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    style HostOS fill:#e2e3e5,stroke:#495057,stroke-width:2px
    style K3sCluster fill:#e8f4f8,stroke:#0d6efd,stroke-width:2px
    style SecNamespace fill:#f3e8fd,stroke:#6f42c1,stroke-width:2px
    style WorkloadsNamespace fill:#ffffff,stroke:#198754,stroke-width:2px
    style NFT fill:#ffe5e5,stroke:#dc3545,stroke-width:2px
```
