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

\*\*
-----------------------------------------------------------------------\*\*

VM OS Current State Role

\*\* ----------------- ----------------- -----------------
-----------------\*\*

`WIN-DC01` Windows Server Deployed AD DS, DNS,

                    2025 Standard                       Domain Controller

                    Evaluation                          

`WIN-SRV01` Windows Server Planned Member server /

                                                        infrastructure

                                                        services

`WIN-CL01` Windows 11 Pro Deployed AD domain member

                                                        workstation

------------------------------------------------------------------------

### Network Design

`WIN-DC01` is multihomed with two VirtualBox network adapters:

\*\*
------------------------------------------------------------------------\*\*

Interface VirtualBox Address Purpose

                    Network                              

\*\* ----------------- ----------------- ------------------
-----------------\*\*

Ethernet NAT `10.0.2.15/24` Internet

                                      (DHCP)             connectivity

Ethernet 2 Internal Network `10.20.20.10/24` Active Directory

                    `win-lab`         (static)           / DNS / domain

                                                         traffic

------------------------------------------------------------------------

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

`WIN-SRV01` is connected to the internal `win-lab` network with:

``` text
IPv4: 10.20.20.30/24
DNS:  10.20.20.10
Gateway: none
```

`WIN-SRV01` is joined to `winlab.test` as a `MemberServer` and its
computer object is located in `OU=Servers,OU=WINLAB,DC=winlab,DC=test`.

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
Member server:     WIN-SRV01
Server IPv4:       10.20.20.30
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

    -   Host: `C:\VM-Share\\`

    -   Guest: `\\\VBOXSVR\VM-Share\\`

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

-   Successfully logged into `WIN-CL01` as `WINLAB\alice.morgan`.

-   Successfully logged into `WIN-CL01` as `WINLAB\bob.carter`.

-   Verified `WINLAB` as the user domain and `WIN-DC01` as the logon

    server.

-   Observed enforcement of the domain password policy during Alice

    Morgan's first login.

-   Created the `01-domain-joined` VirtualBox snapshot.

### Group Policy Administration

-   Opened and explored Group Policy Management on `WIN-DC01`.

-   Practiced GPO creation, linking, scope, inheritance, and Resultant

    Set of Policy (RSoP) validation.

-   Created `WINLAB - Workstation Baseline`.

-   Linked the workstation baseline to `WINLAB\Workstations`.

-   Verified on `WIN-CL01` that the workstation baseline and inherited

    `Default Domain Policy` were applied to the computer.

-   Created `WINLAB - User Baseline`.

-   Linked the user baseline to `WINLAB\Users`.

-   Configured the user policy `Prevent changing desktop background`.

-   Forced user-side policy processing with

    `gpupdate /target:user /force`.

-   Verified with `gpresult /r` that Alice Morgan received

    `WINLAB - User Baseline`.

-   Confirmed the policy's visible effect by verifying that Windows 11

    desktop-background controls were disabled while the existing

    wallpaper remained unchanged.

-   Used GPMC Group Policy Results from `WIN-DC01` to collect remote

    RSoP information for `WIN-CL01`.

-   Troubleshot an initial remote Group Policy Results RPC failure.

-   Verified RPC endpoint connectivity and the `RpcSs` and `Winmgmt`

    services.

-   Identified disabled Windows Management Instrumentation (WMI)

    firewall rules as the blocker for remote RSoP.

-   Enabled the required WMI firewall access and successfully collected

    Group Policy Results remotely.

-   Initiated a centralized Group Policy Update from the

    `WINLAB\Workstations` OU.

-   Troubleshot remote update error `8007071a`.

-   Identified disabled Domain-profile

    `Remote Scheduled Tasks Management` RPC firewall rules.

-   Enabled only the Domain-profile Remote Scheduled Tasks Management

    rules, leaving Private/Public equivalents disabled.

-   Successfully triggered a remote Group Policy Update from `WIN-DC01`

    to `WIN-CL01`.

-   Verified the resulting refresh on `WIN-CL01` with `gpresult /r`.

-   Distinguished the remote-management requirements for Group Policy

    Results (WMI) from Remote Group Policy Update (Scheduled Tasks/RPC).

### Member Server Deployment

-   Created the `WIN-SRV01` VirtualBox VM.
-   Installed Windows Server 2025 Standard Evaluation (Desktop
    Experience).
-   Configured 4 GB RAM, 2 vCPUs, EFI, and a 50 GB dynamically allocated
    VDI.
-   Connected the server to the internal `win-lab` VirtualBox network.
-   Installed VirtualBox Guest Additions.
-   Configured Spanish (Spain) keyboard input while retaining the
    English Windows environment.
-   Configured Num Lock for the current user and logon environment.
-   Observed the initial APIPA address because no DHCP service was
    available on `win-lab`.
-   Configured static IPv4 address `10.20.20.30/24`.
-   Configured `10.20.20.10` (`WIN-DC01`) as the DNS server.
-   Left the default gateway unset on the isolated internal network.
-   Verified IPv4 connectivity to `WIN-DC01`.
-   Verified `winlab.test` DNS resolution.
-   Verified LDAP connectivity to the Domain Controller.
-   Renamed the server from its generated Windows name to `WIN-SRV01`.
-   Verified Domain Controller discovery with
    `nltest /dsgetdc:winlab.test`.
-   Successfully joined `WIN-SRV01` to `winlab.test` using PowerShell
    `Add-Computer`.
-   Verified `WIN-SRV01` as an Active Directory `MemberServer`.
-   Verified domain logon as `WINLAB\Administrator`.
-   Moved the `WIN-SRV01` computer object from the default `Computers`
    container into `WINLAB\Servers`.
-   Verified the final Distinguished Name:
    `CN=WIN-SRV01,OU=Servers,OU=WINLAB,DC=winlab,DC=test`.
-   Verified the domain secure channel with
    `Test-ComputerSecureChannel -Verbose`.
-   Confirmed the network profile changed from `Unidentified network` to
    `winlab.test`.
-   Created the `00-win-srv01-domain-member` snapshot on `WIN-SRV01`.
-   Created the `06-win-srv01-domain-joined` checkpoint snapshot on
    `WIN-DC01`.

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

-   `docs/05-group-policy.md` --- Computer and user GPO baselines,

    policy processing, `gpupdate`, `gpresult`, RSoP, remote Group Policy

    Results, firewall troubleshooting, and centralized Group Policy

    Update

## Current Status

**Member-server foundation deployed and validated.**

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

-   `GG-IT` nested into `DL-IT-Modify` as the foundation of the AGDLP

    permissions model.

-   `WIN-CL01` deployed, domain joined, and placed in

    `WINLAB\Workstations`.

-   Domain authentication successfully tested with `alice.morgan` and

    `bob.carter`.

-   `WIN-DC01` verified as the domain logon server.

-   `WINLAB - Workstation Baseline` linked to `WINLAB\Workstations` and

    successfully applied to `WIN-CL01`.

-   `WINLAB - User Baseline` linked to `WINLAB\Users` and successfully

    applied to Alice Morgan.

-   User-side policy enforcement visibly validated on Windows 11.

-   Local policy refresh and verification validated with `gpupdate` and

    `gpresult`.

-   Remote Group Policy Results from `WIN-DC01` to `WIN-CL01`

    operational.

-   Required WMI firewall access identified for remote RSoP.

-   Remote Group Policy Update from `WIN-DC01` to `WIN-CL01`

    operational.

-   Required Domain-profile Remote Scheduled Tasks Management RPC

    firewall access identified and enabled.

-   Private/Public Remote Scheduled Tasks Management rules remain

    disabled.

-   `01-domain-joined` snapshot exists as the previous major checkpoint.

## Next Phase

**File Services, Storage, and AGDLP Permissions**

Immediate next steps:

-   Add a dedicated virtual data disk to `WIN-SRV01`.
-   Initialize, partition, format, and mount the data volume.
-   Install and configure Windows File Services.
-   Create a realistic departmental SMB share.
-   Configure NTFS and share permissions through the existing
    `DL-IT-Modify` Domain Local group.
-   Validate the AGDLP path
    `Accounts → Global → Domain Local → Permissions` using Alice and
    Bob.
-   Practice accessing and administering the share from `WIN-CL01`.
-   Introduce Group Policy Preferences for centrally mapping the domain
    share after the file service is operational.
-   Provide controlled Internet connectivity for the internal
    client/server network in a later networking phase.
