# 🛡️ Project — Automated Bastion Deployment with Teleport & Ansible

> **TSSR Training — Cybersecurity Portfolio**  
> Status: 🚧 In Progress  
> Stack: Ansible · Teleport · step-ca · nmap · Debian 12

---

## Project Goal

The goal of this project is to build a **fully automated privileged access management (PAM) bastion** using open-source tools.

A single Ansible playbook run should be able to:
- Scan the network and detect all machines (Linux, Windows, web servers)
- Deploy Teleport on a Debian 12 server
- Set up an internal Certificate Authority (CA) using **step-ca**
- Issue and distribute signed certificates to every machine
- Register all resources into Teleport
- Lock down direct SSH/RDP access — **Teleport only**

The project is designed as a learning experience, building skills progressively across networking, PKI, automation, and access security.

---

## Why This Stack?

| Tool | Role | Why |
|---|---|---|
| **Ansible** | Automation engine | Industry standard for infrastructure automation |
| **Teleport** | Bastion / PAM | Modern open-source alternative to legacy bastions |
| **step-ca** | Internal CA | Lightweight PKI with REST API — integrates natively with Teleport |
| **nmap** | Network scanning | Detect and fingerprint machines on the network |
| **Debian 12** | Server OS | Stable, lightweight, standard in enterprise environments |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Local Network                            │
│                                                             │
│  [Ansible Controller — your PC]                             │
│          │                                                  │
│          ▼                                                  │
│  [Teleport + step-ca — Debian 12]                           │
│     (Bastion server / Internal CA)                          │
│          │                                                  │
│    ┌─────┼──────────────┐                                   │
│    ▼     ▼              ▼                                   │
│ [Linux] [Windows]  [Web Server]                             │
│ (SSH)   (RDP)      (App)                                    │
│                                                             │
│  → All access goes through Teleport only                    │
│  → Direct SSH/RDP blocked on all machines                   │
│  → All certificates signed by internal step-ca             │
└─────────────────────────────────────────────────────────────┘
```

---

## Project Roadmap

### Phase 1 — Proof of Concept: Network Scan
**Goal:** Understand what's on the network before touching anything.

- [ ] Write an Ansible playbook that runs `nmap` on the target subnet
- [ ] Parse nmap results to identify machine types:
  - Port 22 open → Linux node (SSH)
  - Port 3389 open → Windows node (RDP)
  - Port 80/443 open → Web application
- [ ] Generate a dynamic Ansible inventory from scan results
- [ ] Validate the inventory manually before proceeding

**What I expect to learn:** nmap scripting, Ansible dynamic inventory, Jinja2 templating

---

### Phase 2 — Teleport Deployment
**Goal:** Install and configure Teleport on the Debian 12 server.

- [ ] Write a role `teleport-server` that:
  - Adds the official Teleport APT repository
  - Installs Teleport
  - Generates `teleport.yaml` from a Jinja2 template
  - Enables and starts the Teleport systemd service
- [ ] Create the first admin user via `tctl`
- [ ] Verify the web dashboard is accessible

**What I expect to learn:** Ansible roles, Jinja2 templates, systemd service management

---

### Phase 3 — Internal CA with step-ca
**Goal:** Set up a trusted internal Certificate Authority so all machines trust Teleport.

- [ ] Write a role `internal-ca` that:
  - Installs `step-ca` on the Teleport server (or a dedicated VM)
  - Initializes the CA and generates the root certificate
  - Exposes the CA via its REST API
- [ ] Configure Teleport to use the step-ca signed certificate
- [ ] Export and distribute the CA root certificate to all machines via Ansible
  - Linux: `update-ca-certificates`
  - Windows: certificate store via PowerShell

**What I expect to learn:** PKI concepts, CSR/certificate signing flow, step-ca REST API, Ansible certificate management

---

### Phase 4 — Resource Enrollment
**Goal:** Register all detected machines into Teleport automatically.

- [ ] Write a role `teleport-node` for Linux machines:
  - Copy CA root certificate
  - Generate a join token via `tctl`
  - Install Teleport agent
  - Configure SSH access via key file only (no password)
- [ ] Write a role `teleport-windows` for Windows machines:
  - Distribute CA certificate via PowerShell
  - Configure `windows_desktop_service` in Teleport
  - Disable NLA, configure RDP port
- [ ] Write a role `teleport-app` for web servers:
  - Add `app_service` block to the node's `teleport.yaml`
  - Register the application in the cluster

**What I expect to learn:** Teleport agent enrollment, SSH key-based auth, Windows automation via Ansible WinRM

---

### Phase 5 — Hardening
**Goal:** Ensure no machine is reachable without going through Teleport.

- [ ] Block direct SSH (port 22) on all Linux machines except from the Teleport server
- [ ] Block direct RDP (port 3389) on all Windows machines except from the Teleport server
- [ ] Verify all access via Teleport dashboard and `tsh` CLI
- [ ] Test session recording and audit log

**What I expect to learn:** iptables/ufw firewall rules via Ansible, security validation

---

## Planned Repository Structure

```
teleport-bastion-ansible/
├── README.md                  ← this file
├── ansible.cfg
├── inventory.yml              # static inventory (Teleport server)
├── group_vars/
│   └── all.yml                # global variables (subnet, cluster name, etc.)
├── playbooks/
│   ├── 1-scan.yml             # network scan → dynamic inventory
│   ├── 2-install-teleport.yml # Teleport deployment
│   ├── 3-install-ca.yml       # step-ca setup + cert distribution
│   ├── 4-enroll.yml           # resource registration
│   └── 5-harden.yml           # lock down direct access
└── roles/
    ├── network-scan/          # nmap scan + inventory generation
    ├── teleport-server/       # Teleport install + config
    ├── internal-ca/           # step-ca setup + cert signing
    ├── teleport-node/         # Linux agent enrollment
    ├── teleport-windows/      # Windows RDP enrollment
    └── teleport-app/          # Web app registration
```

---

## Known Challenges

- **Dynamic inventory from nmap** — nmap XML output needs to be parsed and converted into a valid Ansible inventory. Not straightforward, will require research.
- **WinRM for Windows automation** — Ansible communicates with Windows via WinRM, not SSH. Setup can be tricky, especially with certificate trust.
- **step-ca REST API in Ansible** — using the `uri` Ansible module to submit CSRs and retrieve signed certificates programmatically.
- **Secret management** — join tokens, CA passwords and private keys need to be handled securely using **Ansible Vault**.

---

## Personal Notes

This project is intentionally built step by step, starting with a working proof of concept for each phase before moving to the next. The goal is not just to make it work, but to understand every component deeply enough to explain and document it.

Each phase will be documented with:
- What was done
- What worked and what didn't
- What I would improve

---

> 💡 *This project is part of a technical portfolio built during TSSR training, oriented toward cybersecurity. See also: [SCCM Lab](../sccm-lab/)
