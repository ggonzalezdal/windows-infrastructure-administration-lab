# Member Server Deployment --- WIN-SRV01

## Overview

Phase 6 introduced the first Windows Server member server into the
`winlab.test` Active Directory environment.

`WIN-SRV01` was installed as Windows Server 2025 Standard Evaluation
with Desktop Experience, configured with a static IPv4 address, joined
to the domain, moved into the appropriate Organizational Unit, and
validated as a healthy domain member.

This establishes the baseline for future infrastructure services such as
file services, SMB shares, storage management, and permission
administration.

------------------------------------------------------------------------

## Server Information

  Setting              Value
  -------------------- ------------------------------------------
  Hostname             `WIN-SRV01`
  Operating System     Windows Server 2025 Standard Evaluation
  Installation         Desktop Experience
  Domain               `winlab.test`
  Domain Role          Member Server
  IPv4 Address         `10.20.20.30/24`
  Default Gateway      None
  DNS Server           `10.20.20.10`
  VirtualBox Network   `win-lab`
  AD Location          `OU=Servers,OU=WINLAB,DC=winlab,DC=test`

## Virtual Machine Configuration

`WIN-SRV01` was created in VirtualBox with:

-   4 GB RAM
-   2 virtual CPUs
-   EFI enabled
-   50 GB dynamically allocated VDI system disk
-   Windows Server 2025 Evaluation ISO
-   One network adapter connected to the `win-lab` Internal Network

No NAT adapter was configured. The server therefore communicates
directly with the isolated Windows lab network and currently has no
default route to the Internet.

## Windows Server Installation

The following edition was installed:

`Windows Server 2025 Standard Evaluation (Desktop Experience)`

Desktop Experience provides the graphical Windows Server administration
environment, including Server Manager and the standard management tools.

The built-in local `Administrator` account was configured during
installation.

## VirtualBox Guest and Usability Configuration

VirtualBox Guest Additions were installed.

Windows remains in English while using the Spanish (Spain) keyboard
layout.

The default input method was configured with:

``` powershell
Set-WinDefaultInputMethodOverride -InputTip "0C0A:0000040A"
```

The current user's actual input method was then updated:

``` powershell
$lang = Get-WinUserLanguageList
$lang[0].InputMethodTips.Clear()
$lang[0].InputMethodTips.Add("0C0A:0000040A")
Set-WinUserLanguageList $lang -Force
```

Verification:

``` powershell
Get-WinUserLanguageList
Get-WinDefaultInputMethodOverride
```

NumLock was enabled for the logon environment and current user:

``` powershell
Set-ItemProperty -Path "Registry::HKEY_USERS\.DEFAULT\Control Panel\Keyboard" `
    -Name InitialKeyboardIndicators -Value "2"

Set-ItemProperty -Path "HKCU:\Control Panel\Keyboard" `
    -Name InitialKeyboardIndicators -Value "2"
```

## Initial Network State

Before static configuration, Windows assigned an APIPA address:

``` text
169.254.139.122
```

The network profile appeared as `Unidentified network`.

This was expected because the isolated `win-lab` network did not provide
DHCP to the server.

## Static IPv4 and DNS Configuration

The server was assigned `10.20.20.30/24`:

``` powershell
New-NetIPAddress `
    -InterfaceAlias "Ethernet" `
    -IPAddress 10.20.20.30 `
    -PrefixLength 24
```

No default gateway was configured.

The domain controller was configured as the DNS server:

``` powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -ServerAddresses 10.20.20.10
```

Final configuration:

``` text
IPv4:    10.20.20.30/24
Gateway: None
DNS:     10.20.20.10
```

## Connectivity Testing

IPv4 connectivity to the domain controller was verified:

``` powershell
Test-Connection 10.20.20.10 -Count 2
ping 10.20.20.10
```

DNS resolution was verified:

``` powershell
Resolve-DnsName winlab.test
```

Result:

``` text
winlab.test -> 10.20.20.10
```

LDAP connectivity was verified:

``` powershell
Test-NetConnection WIN-DC01 -Port 389
```

The LDAP test succeeded. During this test Windows selected the domain
controller's link-local IPv6 address, which did not prevent successful
LDAP communication.

## Computer Rename

The initial automatically generated hostname was:

``` text
WIN-8J5PDU25GOV
```

The server was renamed before joining the domain:

``` powershell
Rename-Computer -NewName "WIN-SRV01" -Restart
```

After reboot:

``` powershell
hostname
whoami
```

returned:

``` text
WIN-SRV01
win-srv01\administrator
```

## Active Directory Domain Discovery

Before joining the domain, domain controller discovery was tested:

``` powershell
nltest /dsgetdc:winlab.test
```

The server successfully discovered:

``` text
DC:          \\WIN-DC01.winlab.test
Address:     \\10.20.20.10
Domain:      winlab.test
Forest:      winlab.test
Site:        Default-First-Site-Name
```

The returned flags confirmed services including PDC, Global Catalog,
LDAP, Kerberos KDC, DNS, time services, and a writable domain
controller.

## Domain Join

`WIN-SRV01` was joined to Active Directory using PowerShell:

``` powershell
Add-Computer `
    -DomainName "winlab.test" `
    -Credential "WINLAB\Administrator"
```

The PowerShell domain join completed successfully and the server was
restarted.

Unlike the earlier `WIN-CL01` troubleshooting exercise, the PowerShell
domain join worked immediately on `WIN-SRV01`.

## Domain Membership Verification

After reboot, domain membership was verified:

``` powershell
Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsDomainRole
```

Result:

``` text
CsName       : WIN-SRV01
CsDomain     : winlab.test
CsDomainRole : MemberServer
```

The logged-in identity was:

``` powershell
whoami
```

``` text
winlab\administrator
```

## Active Directory Computer Object

The domain join initially created the computer account in the default
Computers container:

``` text
CN=WIN-SRV01,CN=Computers,DC=winlab,DC=test
```

From `WIN-DC01`, it was moved into the custom Servers OU:

``` powershell
Get-ADComputer WIN-SRV01 |
    Move-ADObject -TargetPath "OU=Servers,OU=WINLAB,DC=winlab,DC=test"
```

Verification:

``` powershell
Get-ADComputer WIN-SRV01 |
    Select-Object Name, DistinguishedName
```

Final location:

``` text
CN=WIN-SRV01,OU=Servers,OU=WINLAB,DC=winlab,DC=test
```

The move was also verified in Active Directory Users and Computers.

## Final Network Verification

After the domain join, `Get-NetIPConfiguration` showed:

``` text
Interface:       Ethernet
Network Profile: winlab.test
IPv4:            10.20.20.30
Default Gateway: None
DNS:             10.20.20.10
```

The network profile had changed from `Unidentified network` to
`winlab.test`.

## Secure Channel Verification

The trust relationship between `WIN-SRV01` and Active Directory was
tested with:

``` powershell
Test-ComputerSecureChannel -Verbose
```

Result:

``` text
True
```

PowerShell reported that the secure channel between the local computer
and the `winlab.test` domain was in good condition.

## Final Topology

``` text
Windows 11 Host
└── VirtualBox
    └── win-lab — 10.20.20.0/24
        │
        ├── WIN-DC01 — 10.20.20.10
        │   ├── Active Directory Domain Services
        │   ├── DNS
        │   └── Domain: winlab.test
        │
        ├── WIN-CL01 — 10.20.20.20
        │   ├── Windows 11 domain workstation
        │   └── OU: WINLAB\Workstations
        │
        └── WIN-SRV01 — 10.20.20.30
            ├── Windows Server 2025
            ├── Domain member server
            └── OU: WINLAB\Servers
```

## Validation Checklist

-   Windows Server 2025 Standard Evaluation installed
-   Desktop Experience installed
-   VirtualBox Guest Additions installed
-   Spanish keyboard configured
-   NumLock configured
-   Hostname changed to `WIN-SRV01`
-   Static IPv4 `10.20.20.30/24`
-   DNS configured as `10.20.20.10`
-   Connectivity with `WIN-DC01` verified
-   `winlab.test` DNS resolution verified
-   LDAP connectivity verified
-   Active Directory domain controller discovery verified
-   PowerShell domain join successful
-   Domain role reported as `MemberServer`
-   Computer account created in Active Directory
-   Computer object moved to `WINLAB\Servers`
-   Secure channel with `winlab.test` verified
-   Network profile identified as `winlab.test`

## Snapshots

### WIN-SRV01

``` text
00-win-srv01-domain-member
```

Clean member-server baseline before installing additional server roles
or adding dedicated data storage.

### WIN-DC01

``` text
06-win-srv01-domain-joined
```

Active Directory checkpoint after `WIN-SRV01` was successfully joined
and moved into the Servers OU.

## Next Steps

The next stage will build infrastructure services on top of this
member-server baseline:

1.  Add a dedicated virtual data disk to `WIN-SRV01`.
2.  Initialize, partition, and format the disk.
3.  Configure Windows File Services.
4.  Create SMB shares.
5.  Configure NTFS and share permissions.
6.  Apply the existing AGDLP model:
    `Accounts -> Global Groups -> Domain Local Groups -> Permissions`.
7.  Test access from `WIN-CL01` using domain users.
8.  Integrate shared resources with Group Policy where appropriate.

`WIN-SRV01` is now ready to become the first dedicated infrastructure
member server in the `winlab.test` environment.
