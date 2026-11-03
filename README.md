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


