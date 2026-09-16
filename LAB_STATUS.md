# Lab Status

## Current Phase

**Phase 2 — Active Directory and DNS Foundation — COMPLETE**

The lab has progressed from the initial standalone Windows Server foundation to a functioning Active Directory domain.

The next phase is:

**Phase 3 — Active Directory Administration**

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
| WIN-CLIENT01 | Planned | Domain workstation |

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
└── 02-active-directory-dns.md
```

`01-windows-server-foundation.md` documents the Windows Server VM, Guest Additions, host integration, server identity, updates, and two-NIC networking foundation.

`02-active-directory-dns.md` documents AD DS installation, forest/domain creation, DNS configuration, multihomed DNS troubleshooting, and final validation.

---

## Next Step

Begin **Phase 3 — Active Directory Administration**.

Planned work:

- Inspect the default Active Directory structure.
- Create Organizational Units (OUs).
- Create domain users.
- Create security groups.
- Manage group membership.
- Practice AD administration with PowerShell.
- Establish a clean logical structure for future servers and workstations.

After the initial AD structure is ready, deploy `WIN-CLIENT01`, connect it to the `win-lab` network, configure `10.20.20.10` as its DNS server, and join it to the `winlab.test` domain.

---

## Checkpoint

**Current stable checkpoint: First Domain Controller + AD DNS complete and reboot-tested.**

This is the recommended rollback point before beginning Active Directory object administration and adding domain clients.
