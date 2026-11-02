# sovstack-core

> Automated, reproducible, and sovereign edge infrastructure baseline designed for air-gapped environments.

`sovstack-core` provides a hardened, Infrastructure-as-Code (IaC) foundation for deploying and operating single-node appliances in environments where strict data isolation, zero outbound egress, and local governance are paramount (e.g., healthcare clinics, legal practices, and regulated SMEs).

This repository represents **Phase 1** of the sovStack ecosystem, establishing the declarative OS hardening, container orchestration, secret management, and network containment layer.

---

## Architecture Overview

```mermaid
flowchart TD
    subgraph Management ["Management and Control"]
        Admin["Operator Workstation"]
        IaC["IaC Engine: Terraform and Ansible"]
        Admin --> IaC
    end

    subgraph Perimeter ["Zero-Trust Perimeter"]
        VPN["WireGuard Encrypted Tunnel"]
        Firewall["nftables (Zero Outbound Egress)"]
    end

    subgraph Appliance ["sovstack-core Node (Edge / Air-Gap)"]
        subgraph K3sCluster ["K3s Single-Node Cluster"]
            Ingress["Traefik Reverse Proxy (Internal TLS)"]
            Vault[("HashiCorp Vault (Sealed PKI)")]
            Workloads["Isolated Core Workloads"]
            Storage["Persistent Local CSI Storage"]

            Ingress --> Vault
            Ingress --> Workloads
            Workloads -.-> Vault
            Workloads --> Storage
        end
    end

    IaC -->|Provision and Bootstrap| VPN
    VPN -->|Authenticated Tunnel| Firewall
    Firewall --> Ingress

    style Management fill:#f8f9fa,stroke:#6c757d,stroke-width:2px
    style Perimeter fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    style Appliance fill:#e8f4f8,stroke:#17a2b8,stroke-width:2px
    style K3sCluster fill:#ffffff,stroke:#28a745,stroke-width:2px
    style Vault fill:#f3e8fd,stroke:#6f42c1,stroke-width:2px
```
