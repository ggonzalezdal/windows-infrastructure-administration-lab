# Lab Status

## Current Phase

**Phase 3 — Active Directory Administration — COMPLETE**

The lab now has a functioning Active Directory domain together with a custom organizational structure, domain users, security groups, and foundational PowerShell administration.

The next phase is:

**Domain Client Deployment and Domain Join**

---

## Current State

### Lab Platform

```text
Host:        MSI Creator M16 A12UD
Host OS:     Windows 11
Hypervisor:  VirtualBox 7.2.x
```

### Virtual Machines

| VM | State | Role |
| --- | --- | --- |
| WIN-DC01 | Deployed / operational | Domain Controller, AD DS, DNS |
| WIN-SRV01 | Planned | Member server / infrastructure services |
| WIN-CL01 | Planned | Domain workstation |

### WIN-DC01

```text
OS:              Windows Server 2025 Standard Evaluation
Hostname:        WIN-DC01
Domain:          winlab.test
NetBIOS domain:  WINLAB
FQDN:            WIN-DC01.winlab.test
```

`WIN-DC01` is currently:

- The first and only Domain Controller.
- A Global Catalog.
- The authoritative DNS server for the lab domain.
- The holder of all FSMO roles.

### Networking

`WIN-DC01` has two network interfaces:

```text
Ethernet
  VirtualBox: NAT
  IPv4:       10.0.2.15/24 (DHCP)
  Gateway:    10.0.2.2
  Purpose:    Internet connectivity
  AD DNS registration: disabled

Ethernet 2
  VirtualBox: Internal Network (win-lab)
  IPv4:       10.20.20.10/24 (static)
  Gateway:    none
  Purpose:    Active Directory / DNS / domain network
```

Internal lab subnet:

```text
10.20.20.0/24
```

Domain clients will use:

```text
DNS server: 10.20.20.10
```

### Active Directory

The following infrastructure is operational:

- Forest: `winlab.test`
- Domain: `winlab.test`
- NetBIOS domain: `WINLAB`
- Domain/forest functional level: Windows Server 2025
- AD DS installed and operational
- DNS installed and AD-integrated
- AD SRV records verified

Custom Organizational Unit structure:

```text
winlab.test
└── WINLAB
    ├── Groups
    ├── Servers
    ├── Users
    └── Workstations
```

Current lab identities:

```text
WINLAB\Users
├── Alice Morgan (alice.morgan)
└── Bob Carter   (bob.carter)
```

Current security group design:

```text
Alice Morgan ─┐
              ├── GG-IT ──> DL-IT-Modify ──> Future resource permission
Bob Carter ───┘
```

- `GG-IT` — Global Security group representing IT users.
- `DL-IT-Modify` — Domain Local Security group intended for Modify access to IT resources.
- Alice Morgan and Bob Carter are members of `GG-IT`.
- `GG-IT` is nested inside `DL-IT-Modify`.
- AGDLP (`Accounts → Global → Domain Local → Permissions`) has been introduced and implemented through the group-nesting stage.

Active Directory administration practiced through ADUC and PowerShell includes:

- Inspecting Organizational Units and Distinguished Names.
- Creating domain users.
- Querying and filtering users with `Get-ADUser`.
- Enabling and disabling accounts.
- Resetting passwords.
- Requiring password changes at next logon.
- Modifying user attributes with `Set-ADUser`.
- Creating Global and Domain Local security groups.
- Managing group membership.
- Moving Active Directory objects between OUs.

### DNS

The multihomed Domain Controller DNS issue has been resolved.

The NAT interface is excluded from DNS client registration:

```powershell
Set-DnsClient `
    -InterfaceAlias "Ethernet" `
    -RegisterThisConnectionsAddress $false
```

The DNS Server listens only on the internal AD interface:

```text
10.20.20.10
```

Final authoritative host record:

```text
WIN-DC01.winlab.test -> 10.20.20.10
```

The configuration was verified successfully after a full server reboot.

---

## Completed Documentation

```text
docs/
├── 01-windows-server-foundation.md
├── 02-active-directory-dns.md
└── 03-active-directory-administration.md
```

`01-windows-server-foundation.md` documents the Windows Server VM, Guest Additions, host integration, server identity, updates, and two-NIC networking foundation.

`02-active-directory-dns.md` documents AD DS installation, forest/domain creation, DNS configuration, multihomed DNS troubleshooting, and final validation.

`03-active-directory-administration.md` documents the custom OU hierarchy, domain users, security groups, PowerShell administration, object management, nested groups, and AGDLP foundations.

---

## Next Step

Begin **Domain Client Deployment and Domain Join**.

Planned work:

- Create the Windows 11 `WIN-CL01` VirtualBox VM.
- Connect `WIN-CL01` to the internal `win-lab` network.
- Configure client networking.
- Configure `10.20.20.10` as the client's DNS server.
- Verify connectivity to `WIN-DC01`.
- Verify Active Directory DNS/DC discovery.
- Join `WIN-CL01` to the `winlab.test` domain.
- Reboot and verify domain membership.
- Locate the new `WIN-CL01` computer object in Active Directory.
- Move the computer object into `WINLAB\Workstations`.
- Log into `WIN-CL01` using a domain account.
- Validate centralized domain authentication.

Once a domain workstation is operational, the lab can move into practical Group Policy administration.

---

## Checkpoint

**Current stable checkpoint: Active Directory administration foundation complete.**

The Domain Controller, DNS infrastructure, custom OU hierarchy, domain identities, security groups, and AGDLP group nesting are operational.

This is the recommended rollback point before deploying and joining the first Windows client to the domain.
