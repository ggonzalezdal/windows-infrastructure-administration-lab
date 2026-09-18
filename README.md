# Windows Infrastructure Administration Lab

Hands-on lab for building, administering, automating, troubleshooting,
and recovering a realistic Windows domain environment.

## Objectives

-   Windows Server administration
-   Active Directory Domain Services (AD DS)
-   DNS and DHCP
-   Group Policy
-   Windows networking
-   File and storage services
-   PowerShell administration and automation
-   Security, monitoring, backup, and recovery
-   Remote administration
-   Realistic Windows infrastructure operations

## Lab Platform

-   Host: MSI Creator M16 A12UD
-   Host OS: Windows 11
-   Hypervisor: VirtualBox 7.2.x
-   Server platform: Windows Server 2025 Standard Evaluation

## Lab Architecture

  -----------------------------------------------------------------------
  VM                OS                Current State     Role
  ----------------- ----------------- ----------------- -----------------
  WIN-DC01          Windows Server    Deployed          AD DS, DNS,
                    2025 Standard                       Domain Controller
                    Evaluation                          

  WIN-SRV01         Windows Server    Planned           Member server /
                                                        infrastructure
                                                        services

  WIN-CL01          Windows 11 Pro    Deployed          AD domain member
                                                        workstation
  -----------------------------------------------------------------------

### Network Design

`WIN-DC01` is multihomed with two VirtualBox network adapters:

  -----------------------------------------------------------------------
  Interface         VirtualBox        Address           Purpose
                    Network                             
  ----------------- ----------------- ----------------- -----------------
  Ethernet          NAT               10.0.2.15/24      Internet
                                      (DHCP)            connectivity

  Ethernet 2        Internal Network  10.20.20.10/24    Active Directory
                    `win-lab`         (static)          / DNS / domain
                                                        traffic
  -----------------------------------------------------------------------

The internal Active Directory network is:

``` text
10.20.20.0/24
```

`WIN-DC01` provides AD-integrated DNS on:

``` text
10.20.20.10
```

`WIN-CL01` is connected to the internal `win-lab` network with:

``` text
IPv4: 10.20.20.20/24
DNS:  10.20.20.10
```

The NAT interface on `WIN-DC01` owns the default route and is excluded
from AD DNS registration.

`WIN-CL01` currently has no default gateway, so Internet access from the
client is not yet configured.

## Active Directory Environment

``` text
Forest:            winlab.test
Domain:            winlab.test
NetBIOS domain:    WINLAB
Domain Controller: WIN-DC01
DC FQDN:           WIN-DC01.winlab.test
DNS server:        10.20.20.10
Domain client:     WIN-CL01
Client IPv4:       10.20.20.20
```

`WIN-DC01` is currently the first and only Domain Controller, Global
Catalog, and holder of all FSMO roles in the lab.

## Completed Work

### Windows Server Foundation

-   Created the `WIN-DC01` VirtualBox VM.
-   Installed Windows Server 2025 Standard Evaluation.
-   Installed VirtualBox Guest Additions.
-   Configured bidirectional clipboard integration.
-   Configured the host/guest shared folder:
    -   Host: `C:\VM-Share\`
    -   Guest: `\\VBOXSVR\VM-Share\`
-   Renamed the server to `WIN-DC01`.
-   Configured Spanish keyboard input while retaining the English
    Windows display language.
-   Configured Num Lock to remain enabled across reboots.
-   Installed Windows updates before Domain Controller promotion.

### Networking Foundation

-   Configured the original VirtualBox NAT adapter for Internet access.
-   Added a second VirtualBox Internal Network adapter using `win-lab`.
-   Assigned `10.20.20.10/24` statically to the internal interface.
-   Kept the default gateway exclusively on the NAT interface.
-   Verified outbound IP and HTTPS connectivity.

### Active Directory and DNS

-   Installed the AD DS role and management tools.
-   Created the new `winlab.test` forest and domain.
-   Installed DNS during Domain Controller promotion.
-   Promoted `WIN-DC01` as the first Domain Controller.
-   Verified domain, forest, Global Catalog, and FSMO role
    configuration.
-   Verified Active Directory DNS SRV discovery records.
-   Troubleshot multihomed Domain Controller DNS registration.
-   Disabled DNS registration on the NAT NIC.
-   Restricted the DNS Server service to `10.20.20.10`.
-   Verified after a full reboot that `WIN-DC01.winlab.test` publishes
    only the internal IPv4 address.

### Active Directory Administration

-   Explored the default Active Directory domain structure and built-in
    containers.
-   Distinguished Active Directory containers from Organizational Units
    (OUs).
-   Created the custom `WINLAB` OU hierarchy:

``` text
winlab.test
└── WINLAB
    ├── Groups
    ├── Servers
    ├── Users
    └── Workstations
```

-   Created `Alice Morgan` through Active Directory Users and Computers
    (ADUC).
-   Created `Bob Carter` with PowerShell.
-   Practiced user account administration:
    -   Querying users with `Get-ADUser`
    -   Enabling and disabling accounts
    -   Resetting passwords
    -   Requiring password changes at next logon
    -   Modifying user attributes with `Set-ADUser`
    -   Filtering users by directory attributes
-   Created the `GG-IT` Global Security group.
-   Added Alice Morgan and Bob Carter to `GG-IT`.
-   Created the `DL-IT-Modify` Domain Local Security group.
-   Nested `GG-IT` inside `DL-IT-Modify`.
-   Introduced and implemented the `A → G → DL → P` (AGDLP) permissions
    model.
-   Practiced moving Active Directory objects between OUs with
    `Move-ADObject`.
-   Used PowerShell to inspect Organizational Units, Distinguished
    Names, users, groups, and group membership.

### Domain Client Deployment

-   Created and installed the `WIN-CL01` Windows 11 Pro VirtualBox VM.
-   Installed VirtualBox Guest Additions and configured guest
    integration.
-   Created the `00-fresh-windows-install` snapshot.
-   Renamed the workstation to `WIN-CL01`.
-   Connected `WIN-CL01` to the internal `win-lab` network.
-   Configured static IPv4 address `10.20.20.20/24`.
-   Configured `10.20.20.10` (`WIN-DC01`) as the client's DNS server.
-   Verified connectivity to `WIN-DC01`.
-   Verified DNS resolution for `WIN-DC01.winlab.test`.
-   Verified Active Directory LDAP SRV service discovery.
-   Verified Domain Controller discovery with
    `nltest /dsgetdc:winlab.test`.
-   Verified domain credentials against `WIN-DC01`.
-   Documented troubleshooting of a PowerShell `Add-Computer`
    failure/hang.
-   Successfully joined `WIN-CL01` to `winlab.test` through Windows
    System Properties.
-   Successfully logged into the workstation as `WINLAB\Administrator`.
-   Verified `WIN-CL01` as an Active Directory `MemberWorkstation`.
-   Verified local login syntax using `.\localadmin`.
-   Verified the `WIN-CL01` computer object in Active Directory.
-   Moved `WIN-CL01` into the `WINLAB\Workstations` OU.
-   Successfully logged into `WIN-CL01` as `winlab\alice.morgan`.
-   Successfully logged into `WIN-CL01` as `winlab\bob.carter`.
-   Verified `WINLAB` as the user domain and `WIN-DC01` as the logon
    server.
-   Observed enforcement of the domain password policy during Alice
    Morgan's first login.
-   Created the `01-domain-joined` VirtualBox snapshot.

## DNS Multihoming Fix

The Domain Controller originally registered both its internal address
(`10.20.20.10`) and VirtualBox NAT address (`10.0.2.15`) in the AD DNS
zone.

DNS client registration was disabled on the NAT adapter:

``` powershell
Set-DnsClient `
    -InterfaceAlias "Ethernet" `
    -RegisterThisConnectionsAddress $false
```

The DNS Server was then restricted to the internal AD interface:

``` powershell
$dns = Get-DnsServerSetting -All
$dns.ListeningIPAddress = @("10.20.20.10")
Set-DnsServerSetting -InputObject $dns
```

Final reboot-tested DNS state:

``` text
WIN-DC01.winlab.test -> 10.20.20.10
```

## Project Structure

-   `docs/` --- Detailed phase documentation and troubleshooting notes
-   `scripts/` --- PowerShell and automation scripts
-   `README.md` --- Project overview and architecture
-   `LAB_STATUS.md` --- Current lab state and next steps
-   `CHANGELOG.md` --- Chronological project history

## Documentation

-   `docs/01-windows-server-foundation.md` --- Windows Server VM
    preparation, Guest Additions, server identity, updates, and two-NIC
    network foundation
-   `docs/02-active-directory-dns.md` --- First Domain Controller, AD
    DS, DNS, multihomed DNS troubleshooting, and final validation
-   `docs/03-active-directory-administration.md` --- OU design, domain
    users, security groups, PowerShell administration, nested groups,
    and AGDLP foundations
-   `docs/04-domain-client.md` --- Windows 11 client deployment, static
    networking, AD DNS/DC discovery, domain join, computer-object
    placement, standard domain-user authentication, and troubleshooting

## Current Status

**Domain client deployment, Active Directory placement, and standard
domain-user validation complete.**

Current validated state:

-   `WIN-DC01` operational on Windows Server 2025.
-   `winlab.test` Active Directory forest/domain operational.
-   AD-integrated DNS operational.
-   Internal AD network operational at `10.20.20.0/24`.
-   Domain Controller/DNS address: `10.20.20.10`.
-   Internet connectivity retained on `WIN-DC01` through the VirtualBox
    NAT adapter.
-   NAT interface excluded from AD DNS registration.
-   DNS configuration verified after reboot.
-   Custom `WINLAB` OU hierarchy deployed.
-   Domain user accounts created and administered.
-   Global and Domain Local security groups created.
-   Nested group membership validated.
-   AGDLP permissions model introduced.
-   Active Directory administration performed through both ADUC and
    PowerShell.
-   `WIN-CL01` deployed with Windows 11 Pro.
-   `WIN-CL01` configured as `10.20.20.20/24`.
-   Client DNS configured to `WIN-DC01` at `10.20.20.10`.
-   Active Directory DNS and Domain Controller discovery validated from
    the client.
-   `WIN-CL01` successfully joined to `winlab.test`.
-   `WIN-CL01` validated as an Active Directory `MemberWorkstation`.
-   `WIN-CL01` computer object placed in `WINLAB\Workstations`.
-   Domain authentication successfully tested with `alice.morgan` and
    `bob.carter`.
-   `WIN-DC01` verified as the domain logon server for both users.
-   `01-domain-joined` snapshot created.

## Next Phase

**Group Policy Administration**

Immediate next steps:

-   Open and explore Group Policy Management on `WIN-DC01`.
-   Understand GPOs, links, scope, inheritance, and Computer
    Configuration vs. User Configuration.
-   Create the first lab Group Policy Object.
-   Link a workstation-targeted GPO to `WINLAB\Workstations`.
-   Apply and verify the policy from `WIN-CL01`.
-   Practice `gpupdate` and `gpresult`.
-   Provide controlled Internet connectivity for the internal client
    network in a later networking step.
