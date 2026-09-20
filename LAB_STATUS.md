# Lab Status

## Current Phase

**Phase 6 --- Member Server Deployment --- FOUNDATION COMPLETE**

The lab now includes a functioning Windows Server 2025 member server,
`WIN-SRV01`, integrated into the `winlab.test` Active Directory domain.

The server baseline has been validated: static networking, AD DNS
discovery, PowerShell domain join, Servers OU placement, and the domain
secure channel are all operational.

The next work within Phase 6 is:

**File Services, Storage, and AGDLP Permissions**

------------------------------------------------------------------------\*\*

## Current State

### Lab Platform

``` text

Host:        MSI Creator M16 A12UD

Host OS:     Windows 11

Hypervisor:  VirtualBox 7.2.x
```

### Virtual Machines

\*\*
-----------------------------------------------------------------------\*\*

VM State Role

\*\* ----------------------- -----------------------
-----------------------\*\*

`WIN-DC01` Deployed / operational Domain Controller, AD

                                                  DS, DNS

`WIN-SRV01` Planned Member server /

                                                  infrastructure services

`WIN-CL01` Deployed / domain Windows 11 domain

                          joined / validated      workstation

------------------------------------------------------------------------

### WIN-DC01

``` text

OS:              Windows Server 2025 Standard Evaluation

Hostname:        WIN-DC01

Domain:          winlab.test

NetBIOS domain:  WINLAB

FQDN:            WIN-DC01.winlab.test
```

`WIN-DC01` is currently:

-   The first and only Domain Controller.

-   A Global Catalog.

-   The authoritative DNS server for the lab domain.

-   The holder of all FSMO roles.

-   The central Group Policy administration system for the lab.

### WIN-CL01

``` text

OS:              Windows 11 Pro

Hostname:        WIN-CL01

IPv4:            10.20.20.20/24

DNS server:      10.20.20.10

Domain:          winlab.test

Domain role:     MemberWorkstation

AD location:     WINLAB\Workstations
```

`WIN-CL01` is currently:

-   Connected to the VirtualBox Internal Network `win-lab`.

-   Configured with a static IPv4 address.

-   Using `WIN-DC01` as its DNS server.

-   Successfully joined to the `winlab.test` Active Directory domain.

-   Represented by a computer object in the `WINLAB\Workstations` OU.

-   Successfully validated using the domain accounts `alice.morgan` and

    `bob.carter`.

-   Using `WIN-DC01` as the verified domain logon server.

-   Receiving the `WINLAB - Workstation Baseline` computer GPO.

-   Able to process the `WINLAB - User Baseline` for users in

    `WINLAB\Users`.

-   Configured for remote Group Policy Results through the required WMI

    firewall access.

-   Configured for remote Group Policy Update through Domain-profile

    Remote Scheduled Tasks Management RPC firewall access.

-   Able to use the local `localadmin` account with explicit

    local-account syntax such as `.\localadmin`.

-   Not yet configured with a default gateway, so Internet connectivity

    is currently unavailable from the client.

### WIN-SRV01

``` text
OS:              Windows Server 2025 Standard Evaluation
Hostname:        WIN-SRV01
IPv4:            10.20.20.30/24
DNS server:      10.20.20.10
Default gateway: none
Domain:          winlab.test
Domain role:     MemberServer
AD location:     WINLAB\Servers
```

`WIN-SRV01` is currently:

-   Connected to the VirtualBox Internal Network `win-lab`.
-   Configured with static IPv4 address `10.20.20.30/24`.
-   Using `WIN-DC01` (`10.20.20.10`) as its DNS server.
-   Configured without a default gateway on the isolated lab network.
-   Successfully joined to `winlab.test` using PowerShell
    `Add-Computer`.
-   Represented by a computer object in the `WINLAB\Servers` OU.
-   Verified as an Active Directory `MemberServer`.
-   Successfully validated with `WINLAB\Administrator`.
-   Able to discover `WIN-DC01.winlab.test` with
    `nltest /dsgetdc:winlab.test`.
-   Verified for IPv4, DNS, and LDAP connectivity to `WIN-DC01`.
-   Verified with `Test-ComputerSecureChannel -Verbose`; the secure
    channel is healthy.
-   Using the `winlab.test` domain network profile.
-   Ready for dedicated storage and File Services configuration.

### Networking

`WIN-DC01` has two network interfaces:

``` text

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

``` text

Ethernet

  VirtualBox: Internal Network (win-lab)

  IPv4:       10.20.20.20/24 (static)

  Gateway:    none

  DNS:        10.20.20.10

  Purpose:    Active Directory domain workstation
```

`WIN-SRV01` uses the internal network:

``` text
Ethernet
  VirtualBox: Internal Network (win-lab)
  IPv4:       10.20.20.30/24 (static)
  Gateway:    none
  DNS:        10.20.20.10
  Purpose:    Active Directory member server
```

Internal lab subnet:

``` text

10.20.20.0/24
```

Active Directory DNS server:

``` text

10.20.20.10
```

### Active Directory

The following infrastructure is operational:

-   Forest: `winlab.test`

-   Domain: `winlab.test`

-   NetBIOS domain: `WINLAB`

-   Domain/forest functional level: Windows Server 2025

-   AD DS installed and operational

-   DNS installed and AD-integrated

-   AD SRV records verified

-   First Windows client successfully joined to the domain

-   First Windows client placed in the workstation OU

-   Standard domain-user authentication validated from the client

-   Computer and user Group Policy processing validated

Custom Organizational Unit structure:

``` text

winlab.test

└── WINLAB

    ├── Groups

    ├── Servers
    │   └── WIN-SRV01
    ├── Users

    │   ├── Alice Morgan

    │   └── Bob Carter

    └── Workstations

        └── WIN-CL01
```

Current security group design:

``` text

Alice Morgan ─┐

              ├── GG-IT ──> DL-IT-Modify ──> Future resource permission

Bob Carter ───┘
```

-   `GG-IT` --- Global Security group representing IT users.

-   `DL-IT-Modify` --- Domain Local Security group intended for Modify

    access to IT resources.

-   Alice Morgan and Bob Carter are members of `GG-IT`.

-   `GG-IT` is nested inside `DL-IT-Modify`.

-   AGDLP (`Accounts → Global → Domain Local → Permissions`) has been

    implemented through the group-nesting stage.

### Domain Client Validation

`WIN-CL01` is configured with:

``` text

IPv4: 10.20.20.20/24

DNS:  10.20.20.10
```

Connectivity, DNS resolution, LDAP SRV discovery, Domain Controller

discovery, domain membership, and standard-user authentication have all

been validated.

The workstation computer object is located at:

``` text

winlab.test

└── WINLAB

    └── Workstations

        └── WIN-CL01
```

Interactive authentication has been successfully tested with:

``` text

WINLAB\alice.morgan

WINLAB\bob.carter
```

`WIN-DC01` is verified as the domain logon server.

The earlier PowerShell `Add-Computer` domain-join failure/hang remains

documented; the domain join itself was successfully completed through

Windows System Properties.

### Member Server Validation

`WIN-SRV01` is configured with:

``` text
IPv4: 10.20.20.30/24
DNS:  10.20.20.10
```

Connectivity to `WIN-DC01`, DNS resolution for `winlab.test`, LDAP
connectivity, Domain Controller discovery, domain membership, and the
computer secure channel have all been validated.

Domain membership was confirmed with:

``` powershell
Get-ComputerInfo | Select-Object CsName, CsDomain, CsDomainRole
```

Result:

``` text
CsName    CsDomain    CsDomainRole
WIN-SRV01 winlab.test MemberServer
```

The secure channel was verified with:

``` powershell
Test-ComputerSecureChannel -Verbose
```

Result:

``` text
True
```

The server computer object is located at:

``` text
CN=WIN-SRV01,OU=Servers,OU=WINLAB,DC=winlab,DC=test
```

### Group Policy

Two lab GPOs are currently deployed.

#### Workstation Baseline

``` text

WINLAB - Workstation Baseline
```

Linked to:

``` text

WINLAB\Workstations
```

Computer policy processing was verified on `WIN-CL01` with:

``` powershell

gpresult /scope computer /r
```

Applied computer GPOs included:

``` text

WINLAB - Workstation Baseline

Default Domain Policy
```

#### User Baseline

``` text

WINLAB - User Baseline
```

Linked to:

``` text

WINLAB\Users
```

The following user policy was configured:

``` text

User Configuration

└── Policies

    └── Administrative Templates

        └── Control Panel

            └── Personalization

                └── Prevent changing desktop background = Enabled
```

Alice's user policy was refreshed locally with:

``` powershell

gpupdate /target:user /force
```

and verified with:

``` powershell

gpresult /r
```

The result confirmed:

``` text

Applied Group Policy Objects

    WINLAB - User Baseline
```

The Windows 11 desktop background controls were visibly disabled while

Alice's existing wallpaper remained unchanged.

### Remote Group Policy Administration

Remote Group Policy Results was tested from `WIN-DC01`.

The initial attempt failed because the required WMI firewall rules on

`WIN-CL01` were disabled. RPC connectivity and the `RpcSs` and `Winmgmt`

services were verified before identifying the firewall configuration as

the blocker.

After enabling the required WMI firewall access, GPMC successfully

generated remote RSoP information for Alice on `WIN-CL01`.

Remote Group Policy Update was then tested from:

``` text

WINLAB

└── Workstations

    └── Group Policy Update...
```

The first attempt failed with:

``` text

Error Code: 8007071a
```

The Domain-profile Remote Scheduled Tasks Management rules were found

disabled.

Only the required Domain-profile rules were enabled:

``` powershell

Get-NetFirewallRule -DisplayGroup "Remote Scheduled Tasks Management" |

    Where-Object { $\_.Profile -eq "Domain" } |

    Enable-NetFirewallRule
```

Verified state:

``` text

Remote Scheduled Tasks Management (RPC)        True   Domain           Inbound

Remote Scheduled Tasks Management (RPC-EPMAP)  True   Domain           Inbound

Remote Scheduled Tasks Management (RPC)        False  Private, Public  Inbound

Remote Scheduled Tasks Management (RPC-EPMAP)  False  Private, Public  Inbound
```

The remote Group Policy Update was retried and succeeded:

``` text

Completed (1 of 1)

Succeeded (1)

WIN-CL01.winlab.test
```

The refresh was verified from Alice's session:

``` text

Last time Group Policy was applied: 9/20/2026 at 8:19:50 AM

Group Policy was applied from:      WIN-DC01.winlab.test

Applied Group Policy Objects

    WINLAB - User Baseline
```

This established working centralized policy inspection and refresh from

the Domain Controller.

### DNS

The multihomed Domain Controller DNS issue remains resolved.

The NAT interface is excluded from DNS client registration:

``` powershell

Set-DnsClient `

    -InterfaceAlias "Ethernet" `

    -RegisterThisConnectionsAddress $false
```

The DNS Server listens only on the internal AD interface:

``` text

10.20.20.10
```

Final authoritative host record:

``` text

WIN-DC01.winlab.test -> 10.20.20.10
```

The configuration was verified successfully after a full server reboot.

**------------------------------------------------------------------------**

## Completed Documentation

``` text

docs/

├── 01-windows-server-foundation.md

├── 02-active-directory-dns.md

├── 03-active-directory-administration.md

├── 04-domain-client.md
├── 05-group-policy.md
└── 06-member-server.md
```

`01-windows-server-foundation.md` documents the Windows Server VM, Guest

Additions, host integration, server identity, updates, and two-NIC

networking foundation.

`02-active-directory-dns.md` documents AD DS installation, forest/domain

creation, DNS configuration, multihomed DNS troubleshooting, and final

validation.

`03-active-directory-administration.md` documents the custom OU

hierarchy, domain users, security groups, PowerShell administration,

object management, nested groups, and AGDLP foundations.

`04-domain-client.md` documents the Windows 11 client deployment, static

networking, AD DNS and Domain Controller discovery, domain join,

computer-object placement, standard domain-user authentication,

local/domain account distinction, and domain-join troubleshooting.

`05-group-policy.md` documents computer and user GPO baselines, policy

processing, `gpupdate`, `gpresult`, RSoP, remote Group Policy Results,

WMI/firewall troubleshooting, Remote Scheduled Tasks/RPC requirements,

and centralized Group Policy Update.

`06-member-server.md` documents the `WIN-SRV01` deployment, Windows
Server installation, static networking, AD DNS/DC discovery, PowerShell
domain join, Servers OU placement, secure-channel validation, and
baseline snapshots.

**------------------------------------------------------------------------**

## Next Step

Continue **Phase 6 --- File Services, Storage, and AGDLP Permissions**.

Immediate work:

-   Add a dedicated virtual data disk to `WIN-SRV01`.
-   Initialize, partition, format, and mount the data volume.
-   Install and configure Windows File Services.
-   Create a realistic departmental SMB share.
-   Assign NTFS and share permissions through `DL-IT-Modify`.
-   Validate the complete AGDLP path:
    `Accounts → GG-IT → DL-IT-Modify → Permission`.
-   Test access from `WIN-CL01` using Alice and Bob.
-   Introduce Group Policy Preferences for centrally mapping the domain
    share after file services are operational.
-   Provide controlled Internet connectivity for the internal
    client/server network in a later networking phase.

------------------------------------------------------------------------\*\*

## Checkpoint

**Current stable checkpoint: Active Directory domain, domain
workstation, Group Policy foundation, and Windows Server member-server
baseline operational and validated.**

Validated infrastructure:

``` text

WIN-DC01

10.20.20.10

AD DS + DNS + Group Policy administration

     |

     | winlab.test

     |

     +-- WINLAB

          |

          +-- Users

          |    |

          |    +-- Alice Morgan

          |    +-- Bob Carter

          |    |

          |    +-- WINLAB - User Baseline

          |

          +-- Groups

          |    |

          |    +-- GG-IT

          |         |

          |         +--> DL-IT-Modify

          |

          +-- Servers

          |    |

          |    +-- WIN-SRV01 (planned)

          |

          +-- Workstations

               |

               +-- WIN-CL01

                    10.20.20.20

                    MemberWorkstation

                    WINLAB - Workstation Baseline

                    Remote RSoP verified

                    Remote GP Update verified
```

Relevant VirtualBox checkpoints include:

``` text
WIN-CL01
  00-fresh-windows-install
  01-domain-joined
  03-group-policy-complete

WIN-DC01
  05-group-policy-complete
  06-win-srv01-domain-joined

WIN-SRV01
  00-win-srv01-domain-member
```

The current checkpoint is the clean member-server baseline before adding
a dedicated data disk or installing/configuring File Services.
