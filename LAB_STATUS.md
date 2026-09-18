# Lab Status

## Current Phase

**Phase 4 — Domain Client Deployment and Domain Join — COMPLETE**

The lab now has a functioning Active Directory domain and its first domain-joined Windows 11 workstation.

`WIN-CL01` has been deployed, configured on the internal Active Directory network, joined to `winlab.test`, and validated as an Active Directory member workstation.

The next phase is:

**Domain Client Finalization and Group Policy**

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
| WIN-CL01 | Deployed / domain joined | Windows 11 domain workstation |

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

### WIN-CL01

```text
OS:              Windows 11 Pro
Hostname:        WIN-CL01
IPv4:            10.20.20.20/24
DNS server:      10.20.20.10
Domain:          winlab.test
Domain role:     MemberWorkstation
```

`WIN-CL01` is currently:

- Connected to the VirtualBox Internal Network `win-lab`.
- Configured with a static IPv4 address.
- Using `WIN-DC01` as its DNS server.
- Successfully joined to the `winlab.test` Active Directory domain.
- Successfully validated using a domain login.
- Able to use the local `localadmin` account with explicit local-account syntax such as `.\localadmin`.
- Not yet configured with a default gateway, so Internet connectivity is currently unavailable from the client.

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

`WIN-CL01` uses the internal network:

```text
Ethernet
  VirtualBox: Internal Network (win-lab)
  IPv4:       10.20.20.20/24 (static)
  Gateway:    none
  DNS:        10.20.20.10
  Purpose:    Active Directory domain workstation
```

Internal lab subnet:

```text
10.20.20.0/24
```

Active Directory DNS server:

```text
10.20.20.10
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
- First Windows client successfully joined to the domain

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

### Domain Client Validation

`WIN-CL01` was configured with:

```text
IPv4: 10.20.20.20/24
DNS:  10.20.20.10
```

Connectivity to `WIN-DC01` was verified.

Active Directory DNS resolution was validated:

```powershell
Resolve-DnsName WIN-DC01.winlab.test
```

Active Directory LDAP service discovery was validated:

```powershell
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.winlab.test
```

Windows Domain Controller discovery was validated:

```powershell
nltest /dsgetdc:winlab.test
```

Domain credentials were successfully validated against `WIN-DC01`.

A PowerShell `Add-Computer` domain-join attempt returned `Access is denied`, and a subsequent attempt hung. DNS, DC discovery, connectivity, and credentials were verified as operational. The root cause of the PowerShell-specific failure was not determined.

The domain join was successfully completed through Windows System Properties:

```text
sysdm.cpl
→ Computer Name
→ Change
→ Domain: winlab.test
```

After reboot, domain authentication was verified:

```powershell
whoami
```

Result:

```text
winlab\administrator
```

Domain membership was verified:

```powershell
Get-ComputerInfo |
    Select-Object CsName,CsDomain,CsDomainRole
```

Result:

```text
CsName    CsDomain      CsDomainRole
------    --------      ------------
WIN-CL01  winlab.test   MemberWorkstation
```

### DNS

The multihomed Domain Controller DNS issue remains resolved.

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
├── 03-active-directory-administration.md
└── 04-domain-client.md
```

`01-windows-server-foundation.md` documents the Windows Server VM, Guest Additions, host integration, server identity, updates, and two-NIC networking foundation.

`02-active-directory-dns.md` documents AD DS installation, forest/domain creation, DNS configuration, multihomed DNS troubleshooting, and final validation.

`03-active-directory-administration.md` documents the custom OU hierarchy, domain users, security groups, PowerShell administration, object management, nested groups, and AGDLP foundations.

`04-domain-client.md` documents the Windows 11 client deployment, static networking, AD DNS and Domain Controller discovery, domain join, authentication verification, local/domain account distinction, and domain-join troubleshooting.

---

## Next Step

Begin **Domain Client Finalization and Group Policy**.

Immediate work:

- Verify the `WIN-CL01` computer object from `WIN-DC01`.
- Move the `WIN-CL01` computer object into `WINLAB\Workstations`.
- Test interactive authentication using a standard domain user such as Alice Morgan or Bob Carter.
- Provide controlled Internet connectivity for the internal client network.
- Begin practical Group Policy administration using the domain-joined workstation.

The domain-joined workstation now provides the client-side foundation required for centralized Windows administration and Group Policy testing.

---

## Checkpoint

**Current stable checkpoint: first Active Directory domain workstation deployed and joined.**

Validated infrastructure:

```text
WIN-DC01
10.20.20.10
AD DS + DNS
     |
     | winlab.test
     |
WIN-CL01
10.20.20.20
MemberWorkstation
```

VirtualBox snapshots on `WIN-CL01`:

```text
00-fresh-windows-install
01-domain-joined
```

`01-domain-joined` is the current rollback checkpoint before finalizing the client computer object placement, standard-user authentication, Internet access, and beginning Group Policy administration.
