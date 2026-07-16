# Active Directory & Identity Management
> **Windows Server 2025 · Azure Free Account · Identity & Access Management**

![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202025-0078D4?style=flat-square&logo=windows)
![Cloud](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0089D6?style=flat-square&logo=microsoftazure)
![Cost](https://img.shields.io/badge/Cost-%240%20Free%20Tier-brightgreen?style=flat-square)
![Duration](https://img.shields.io/badge/Duration-3--5%20Hours-yellow?style=flat-square)
![Certs](https://img.shields.io/badge/Cert%20Alignment-Network%2B%20%7C%20Security%2B%20%7C%20AZ--104-blueviolet?style=flat-square)

---

## Overview

This lab builds a fully functional **Active Directory (AD)** environment from scratch — covering everything from Domain Controller promotion to Group Policy enforcement and real-world help desk tasks. The skills developed here apply directly to on-premises enterprise environments and cloud-based identity management through **Microsoft Entra ID (formerly Azure AD)**.

Active Directory is the most widely deployed identity platform in enterprise IT. It is also the **most targeted system in ransomware and lateral movement attacks.** Knowing how to build it means knowing how to defend it.

---
🎬 Watch Me Build This Lab!
https://www.loom.com/share/5f9f0cc1bea64d6fb0532b79d5ac0b78
---

## Architecture

```
                        ┌─────────────────────────────────────────────────┐
                        │               lab.local (AD Forest)              │
                        │                                                   │
                        │   ┌─────────────────────────────────────────┐   │
                        │   │         Domain Controller (DC01)         │   │
                        │   │         Windows Server 2025              │   │
                        │   │                                           │   │
                        │   │  ┌─────────┐  ┌─────┐  ┌─────────────┐ │   │
                        │   │  │  AD DS  │  │ DNS │  │    GPMC     │ │   │
                        │   │  └─────────┘  └─────┘  └─────────────┘ │   │
                        │   └─────────────────┬───────────────────────┘   │
                        │                     │ Authenticates             │
                        │         ┌───────────┼───────────┐               │
                        │         │           │           │               │
                        │   ┌─────▼──┐  ┌────▼───┐  ┌───▼────┐          │
                        │   │  Win   │  │  Win   │  │  Win   │          │
                        │   │  WS01  │  │  WS02  │  │  WS03  │          │
                        │   │(domain │  │(domain │  │(domain │          │
                        │   │joined) │  │joined) │  │joined) │          │
                        │   └────────┘  └────────┘  └────────┘          │
                        └─────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│                        OU Structure (lab.local)                          │
│                                                                          │
│   lab.local                                                              │
│   ├── OU=IT          → IT_Admins Group       → alice.chen               │
│   ├── OU=Finance     → Finance_Users Group   → bob.patel                │
│   ├── OU=HR          → HR_Users Group        → carol.jones              │
│   ├── OU=Sales       → Sales_Users Group     → david.smith             │
│   └── OU=Computers   → Domain-joined machines                           │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│                         GPO Enforcement Flow                             │
│                                                                          │
│  Group Policy Management Console (GPMC)                                  │
│         │                                                                │
│         ▼                                                                │
│  IT Security Policy (GPO)  ──linked to──►  OU=IT                        │
│         │                                      │                        │
│         │  Enforces:                           ▼                        │
│         ├─ Min password length: 12        alice.chen                    │
│         ├─ Password complexity: ON        IT Workstations               │
│         ├─ Screen lock: 900 seconds                                     │
│         └─ USB access: DENIED                                           │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│                     Azure Deployment Option                              │
│                                                                          │
│   Your Local Machine                   Azure (East US Region)           │
│   ┌──────────────┐    RDP (3389)    ┌──────────────────────┐           │
│   │   RDP Client │ ───────────────► │  VM: Standard_B2s    │           │
│   │  (clipboard  │                  │  Windows Server 2025  │           │
│   │   enabled)   │                  │  ┌────────────────┐  │           │
│   └──────────────┘                  │  │   AD DS + DNS  │  │           │
│                                     │  │   GPMC + RSAT  │  │           │
│                                     │  └────────────────┘  │           │
│                                     └──────────────────────┘           │
│                                       ⚠ Stop VM when not in use        │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Career Relevance

| Role | How This Lab Applies |
|---|---|
| **IT Support / Help Desk** | Password resets, account unlocks, group membership changes — the top 3 ticket types in any enterprise |
| **Sysadmin** | Designing OU structure, deploying GPOs, managing domain-joined machines at scale |
| **Cloud Engineer** | Entra ID uses the same concepts: users, groups, roles, conditional access. On-prem AD knowledge transfers directly |
| **Security Analyst** | AD is the primary target in ransomware attacks. Understanding how it works is the foundation of defending it |

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Azure Free Account** | [azure.microsoft.com/free](https://azure.microsoft.com/free) — includes $200 credit |
| **RDP Client** | Windows: built-in. macOS: [Microsoft Remote Desktop](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466) |
| **OR — Local Option** | 8GB RAM minimum, quad-core CPU, 60GB free disk, virtualisation enabled in BIOS |
| **VirtualBox** (local only) | [virtualbox.org](https://www.virtualbox.org) — free |
| **Windows Server 2025 ISO** (local only) | [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) — free 180-day eval |

---

## Lab Steps

### Step 1 — Provision the VM

**Option A — Azure (Recommended)**

> No local hardware requirements. Runs in Microsoft's datacentre, connect via RDP.

| Setting | Value |
|---|---|
| Region | East US |
| Image | Windows Server 2025 Datacenter — Gen2 |
| Size | Standard_B2s (2 vCPU / 4GB RAM) |
| Authentication | Password |
| Inbound ports | RDP (3389) |
| OS Disk | Standard SSD |

> ⚠️ **Cost tip:** Stop (do not delete) the VM after every session. A B2s VM costs ~$0.05/hr. Stopping it pauses compute billing and stretches your $200 credit significantly.

**Enable clipboard before connecting:**
1. Open the Remote Desktop client on your local machine
2. Enter the VM's public IP → click **Show Options**
3. Select the **Local Resources** tab
4. Ensure **Clipboard** is checked under *Local devices and resources*
5. Click **Connect**

> 💡 Prefer downloading the `.rdp` file from the Azure portal (**Connect → Download RDP File**) over the browser-based console. Browser clipboard support is severely limited.

---

**Option B — VirtualBox (Local)**

1. Install [VirtualBox](https://www.virtualbox.org)
2. Download the [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
3. Create a new VM: **4GB RAM, 60GB disk, Windows Server 2019/2022 type**
4. Mount the ISO and boot — follow the installation wizard
5. Select **Windows Server 2025 Datacenter with Desktop Experience**

---

### Step 2 — Install AD Domain Services

RDP into the VM. Server Manager opens automatically on login.

**Via GUI:** `Manage → Add Roles and Features → Server Roles → Active Directory Domain Services → Add Features → Install`

**Via PowerShell:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

Install **Group Policy Management Console** immediately after:
```powershell
Install-WindowsFeature -Name GPMC
```
> ⚠️ Install GPMC now. Without it, you will not see Group Policy Management in the Tools menu and will hit a wall at Step 5.

---

### Step 3 — Promote to Domain Controller

> Installing the AD DS role does not create a domain. Promotion creates your forest, your domain, and makes this server the authoritative DNS and identity provider.

**Via GUI:**
1. Click the **yellow warning flag** in Server Manager
2. Click **Promote this server to a domain controller**
3. Select **Add a new forest** → Root domain name: `lab.local`
4. Set a DSRM password (write it down — recovery use only)
5. Accept DNS and NetBIOS defaults → **Install**
6. Server restarts automatically

**Via PowerShell:**
```powershell
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

---

### Step 4 — Build the Organisational Structure

Open **Active Directory Users and Computers (ADUC)** from `Tools` in Server Manager.

**Create Organisational Units:**
```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

**Create Security Groups:**
```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

**Create Users and Assign Group Memberships:**

> ⚠️ Run the entire block at once — not line by line. The `$password` variable must be defined before the `New-ADUser` commands. Select all and press **F8** in PowerShell ISE, or paste the whole block into a PowerShell window and press Enter.

```powershell
# Step 1 — define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create all 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — add each user to their department group
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

### Step 5 — Configure Group Policy

Open **Group Policy Management** from the `Tools` menu in Server Manager.

1. Expand `Forest: lab.local → Domains → lab.local`
2. Right-click the **IT OU** → *Create a GPO in this domain and link it here*
3. Name it: `IT Security Policy`
4. Right-click the GPO → **Edit** and apply the settings below

| Policy Path | Setting | Value |
|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | `12` |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | `Enabled` |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | `900 seconds` |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | `Enabled` |

**Test the GPO:**
```powershell
# On the domain-joined workstation, run as Administrator:
gpupdate /force

# Verify which policies are applied:
gpresult /r
```

---

### Step 6 — Common Help Desk Tasks

**Reset a password:**
```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

**Unlock a locked account:**
```powershell
Unlock-ADAccount -Identity "carol.jones"
```

**Disable an account (employee offboarding):**
```powershell
Disable-ADAccount -Identity "david.smith"

# View all currently disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

**Audit inactive accounts (90+ days):**
```powershell
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate
```

**Check a user's group memberships:**
```powershell
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification Checklist

| Check | Command | Expected Result |
|---|---|---|
| Domain controller is running | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs created |
| Users exist and are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| PowerShell prompts for `Name:` when creating users | You ran `New-ADUser` before defining `$password`. Run the **entire script block at once** — the `$password` line must come first |
| Cannot copy/paste into VM | Open RDP client → **Show Options → Local Resources** → check **Clipboard** → reconnect. Or download the `.rdp` file from Azure portal instead of using the browser console |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting |
| Cannot RDP after domain join | Log in as `LAB\Administrator` — not just `Administrator` |
| GPO not applying | Run `gpupdate /force` then `gpresult /r` to see applied policies |
| User cannot log in after creation | Confirm account is **Enabled** and `ChangePasswordAtLogon` is set |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog, or run `Add-WindowsFeature RSAT-ADDS` |

---

## Skills Demonstrated

- ✅ Provisioning and configuring Windows Server 2025 in Azure
- ✅ Installing and configuring Active Directory Domain Services (AD DS)
- ✅ Promoting a server to a Domain Controller and creating an AD forest
- ✅ Designing and implementing a departmental OU structure
- ✅ Creating and managing users, security groups, and group memberships
- ✅ Enforcing security baselines via Group Policy Objects (GPOs)
- ✅ Performing real-world help desk tasks: password resets, account unlocks, offboarding
- ✅ Running AD audit and reporting queries with PowerShell
- ✅ Applying the principle of least privilege through role-based access control

---

## Certification Alignment

| Certification | Relevant Domains |
|---|---|
| **CompTIA Network+** | Network infrastructure, DNS, domain architecture |
| **CompTIA Security+** | Identity and access management, least privilege, GPO security controls |
| **Azure Administrator (AZ-104)** | Microsoft Entra ID, identity management, VM deployment |

---

*Part of an ongoing home lab series documenting enterprise IT and cloud security skills.*
