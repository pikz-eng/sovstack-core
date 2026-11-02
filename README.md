 sovstack-core

> Automated, reproducible, and sovereign edge infrastructure baseline designed for air-gapped environments.

`sovstack-core` provides a hardened, Infrastructure-as-Code (IaC) foundation for deploying and operating single-node appliances in environments where strict data isolation, zero outbound egress, and local governance are paramount (e.g., healthcare clinics, legal practices, and regulated SMEs).

This repository represents **Phase 1** of the sovStack ecosystem, establishing the declarative OS hardening, container orchestration, secret management, and network containment layer.

---

```mermaid
flowchart TD
    subgraph ManagementPlane [" 🛠️ OPERATOR / SYSADMIN (Private Network) "]
        Admin["Administrator Workstation"]
        Terraform["Terraform (Proxmox/KVM Provider)"]
        Ansible["Ansible Engine"]
        
        Admin --> Terraform
        Terraform -->|Export Host Details| Inventory["ansible/inventory.ini"]
        Inventory -.-> Ansible
    end

    subgraph TransportLayer [" 🔐 ACCESS & ISOLATION LAYER "]
        WG_Tunnel["Encrypted WireGuard VPN (wg0)<br>Port: 51820/UDP | Auth: ED25519 Keys"]
        LAN_Net["Local Enterprise Network (LAN)<br>Subnet: 192.168.10.0/24 (No Internet Gateway)"]
    end

    subgraph OnPremNode [" 🖥️ ON-PREMISE NODE: sovstack-core (AlmaLinux 9) "]
        
        subgraph NFTablesEngine [" 🛡️ nftables Firewall Enforcement "]
            
            subgraph IngressRules [" INGRESS (Allowed Rules) "]
                In_WG["Port 51820/UDP: WireGuard Ingress (Admin)"]
                In_SSH["Port 22/TCP: SSH Console (STRICT wg0 interface only)"]
                In_K8s["Port 6443/TCP: K3s API (STRICT wg0 & 127.0.0.1 only)"]
                In_Web["Ports 80, 443/TCP: Internal Web/Traefik (LAN only)"]
            end

            subgraph EgressRules [" EGRESS (Air-Gap Policy) "]
                Out_Drop["⛔ DEFAULT DROP: 0.0.0.0/0 (All Internet/WAN blocked)"]
                Out_Local["ALLOW: Established/Related & Internal LAN traffic"]
            end
        end

        subgraph VMResources [" 📦 Compute & Storage Resources "]
            NodeOS["OS: AlmaLinux 9 Minimal (SELinux Enforcing)"]
            NodeSpecs["Compute: 4 vCPU | 8GB-16GB RAM"]
            NodeDisk["Storage: Local NVMe/SSD (/var/lib/rancher/k3s)"]
            K3sEngine["K3s Single-Node (containerd)"]
            
            NodeOS --- NodeSpecs
            NodeSpecs --- NodeDisk
            NodeDisk --- K3sEngine
        end
    end

    %% Connections
    Ansible ==>|Run Hardening Playbooks| WG_Tunnel
    WG_Tunnel ==> In_WG
    In_WG --> In_SSH
    In_WG --> In_K8s
    LAN_Net ==> In_Web
    
    IngressRules --> K3sEngine

    %% Styling
    style ManagementPlane fill:#f8f9fa,stroke:#6c757d,stroke-width:2px
    style TransportLayer fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    style OnPremNode fill:#e8f4f8,stroke:#0d6efd,stroke-width:2px
    style NFTablesEngine fill:#fdfdfe,stroke:#495057,stroke-width:1px
    style IngressRules fill:#e8f8f5,stroke:#28a745,stroke-width:2px
    style EgressRules fill:#fdedec,stroke:#dc3545,stroke-width:2px
    style VMResources fill:#ffffff,stroke:#6f42c1,stroke-width:2px
```
