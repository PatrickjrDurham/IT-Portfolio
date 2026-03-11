#  Lab — SCCM (MECM 2403) Installation

> **TSSR Training — Cybersecurity / Sysadmin Portfolio**  
> Author: Patrick Durham | Date: 02/12/2026 | SCCM Version: 2403

---

##  Lab Overview

This lab documents the full installation of **Microsoft Endpoint Configuration Manager (MECM/SCCM) version 2403** in a local virtualized environment.

The goal is to simulate a realistic enterprise deployment to centralize workstation management, updates, and software deployments — **entirely on-premises, no cloud involved**.

---

##  Architecture

```
┌─────────────────────────────────────────────────────┐
│                Local virtual network                │
│                                                     │
│  [pfSense 2.7.2]  ──►  [DC Windows Server 2025]    │
│     (Firewall)           (AD DS, DNS, Schema ext.)  │
│                                  │                  │
│                    [SCCM Server Windows Server 2025] │
│                    (SQL 2022, MECM, IIS, WSUS, ADK) │
│                                  │                  │
│                         [Windows client workstation] │
└─────────────────────────────────────────────────────┘
```

### VM Specs (my lab specs — not necessarily optimal, just what I used)

| VM | OS | vCPU | RAM | Storage | Role |
|---|---|---|---|---|---|
| DC | Windows Server 2025 | 2 | 4 GB | 40 GB | AD DS, DNS, Schema Extension |
| SCCM | Windows Server 2025 | 4 | 12 GB | 100 GB | SQL 2022, MECM, IIS, WSUS, ADK |
| Firewall | pfSense 2.7.2 | 1 | 256 MB | 20 GB | Filtering & routing |
| Client | Windows 10/11 | 2 | 4 GB | 60 GB | Test workstation |

---

##  Prerequisites & Downloads

| Component | Link |
|---|---|
| MECM 2403 | [Microsoft Eval Center](https://www.microsoft.com/en-us/evalcenter/download-microsoft-endpoint-configuration-manager) |
| Windows ADK + ADK PE | [Microsoft Docs](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install) |
| SQL Server 2022 | [Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) |
| SSMS | [Microsoft SSMS](https://learn.microsoft.com/en-us/ssms/install/install) |
| ODBC Driver 18 | [Microsoft ODBC](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server) |

>  **Critical ODBC version note**: Use version **18.5.1.1** only. Version 18.6.1.1 crashes the MECM installation on Windows Server 2025 with no clear error message — costs a lot of time to debug.

---

##  Installation Steps

### 1. Infrastructure Preparation
- Deploy pfSense, Windows Server (DC + SCCM), client workstation
- Configure static IPs according to the lab addressing plan

### 2. AD Configuration — Accounts & OU
Create the `LAB` OU and the following dedicated service accounts:
- `SCCM-ADMIN` (copy of the built-in Administrator account)
- `SCCM Network Access Account`
- `SCCM Client Push Install Account`
- `SCCM Domain Join Account`
- `SCCM SQL Reporting Account`
- `SQLSvrAgent`
- `Tech`

Create the `SCCM Admins Group` group + add `SCCM-ADMIN` to the SCCM server's local Administrators group.

### 3. System Management Container (AD)
>  **Critical step** — a mistake here can take down the domain controller.

Via **ADSI Edit**: create the `System Management` container under `CN=System`, then delegate full permissions to the SCCM server's computer object.

### 4. Active Directory Schema Extension
The schema extension lets SCCM integrate natively with AD for automatic client discovery.

```
# From the DC, as administrator
cd \\cd.retail.LN\SMSSETUP\BIN\X64
extadsch.exe
```

Verification: the file `ExtADSch.txt` should appear at the root of `C:\`.

### 5. Firewall Rules (SCCM Server)
Inbound rule on the **Domain** profile only:
- **TCP 1433** — SQL Server
- **TCP 4022** — SQL Server Service Broker

### 6. IIS Installation
IIS role with the following features:
- .NET Framework 3.5
- Application Development
- IIS 6 Management Compatibility
- IIS Management Scripts and Tools

### 7. SQL Server 2022 Installation
Key points:
- Default instance
- Windows Authentication
- Service account: `SQLSvrAgent` (Automatic startup)
- **Required collation: `SQL_Latin1_General_CP1_CI_AS`**  
  *(Cannot be changed after the database is created — SCCM will not work with any other collation)*
- Add to SQL sysadmin group: `Tech`, `Domain Admins`, current user

### 8. SSMS Installation
Run `vs_SSMS.exe` without any additional extensions. Verify connection using the `Tech` account (disable "Encrypt (Optional)" for local connection).

### 9. Windows ADK + ADK PE Installation
- Run `adksetup.exe` → select only the components required for MECM
- Run `adkwinpesetup.exe` → install Windows PE add-on

### 10. WSUS Installation
- WSUS role with **SQL Server Connectivity** (disable WID Connectivity)
- Content path: `C:\WSUS`
- Database instance: FQDN of the SCCM server
- Verify in SSMS that the WSUS database was created successfully

### 11. MECM Installation
```
MCM_Configmgr_2403.exe → Extract → splash.exe → Install
```
- Default options (standalone primary site)
- Evaluation version
- Site code: e.g. `PLU` / Site name: e.g. `SCCM Plume`
- Create a temp folder for the MECM prerequisite download
- Check for blocking errors before starting the installation

---

##  Personal Feedback

###  What I found that
- Once you know where to look, the Microsoft documentation is actually well written
- Watching SCCM automatically discover clients through AD discovery is very satisfying
- The level of control over deployments and updates is impressive for a fully on-prem tool
- SQL collation and AD schema extension are concepts you rarely encounter in training — genuinely useful to learn hands-on

###  What I found annoying / painful
- The sheer volume of prerequisites before you can even launch MECM — one missed step and the whole thing fails
- The ODBC 18.6.1.1 bug on Windows Server 2025 cost me significant time: the setup crashes with no meaningful error message
- The SCCM VM is resource-hungry (12 GB RAM minimum) — hard to run alongside other VMs on a local setup
- ADSI Edit is stressful: one wrong click on the DC and you risk taking down the whole AD

###  What I would improve or do differently
- Automate repetitive steps (AD account creation, firewall rules) with **PowerShell**
- Set up a **snapshot system** before each critical step (System Management container, schema extension) for clean rollback capability
- Test **SCCM + Intune co-management** to get a hybrid cloud/on-prem perspective
- Forward SCCM logs to **Wazuh** (see SIEM lab) to monitor deployments and detect anomalous behavior

---

##  Repository Structure

```
sccm-lab/
├── README.md                    ← this file
└── docs/
    └── Sccm_Install_Doc_EN.docx ← full documentation with screenshots
```

---

## 🔗 Useful Links
- [Official MECM Documentation](https://learn.microsoft.com/en-us/mem/configmgr/)
- [SCCM Prerequisites — Microsoft](https://learn.microsoft.com/en-us/mem/configmgr/core/plan-design/configs/site-and-site-system-prerequisites)
- [MITRE ATT&CK — T1072: Software Deployment Tools](https://attack.mitre.org/techniques/T1072/) *(SCCM can be abused in attack scenarios — important to understand from a defensive perspective)*

---

> 💡 *This lab is part of a technical portfolio built during TSSR training, as part of a cybersecurity career path. See also: [Teleport Bastion Lab](../teleport-bastion/) | [Wazuh SIEM Lab](../wazuh-siem/)*
