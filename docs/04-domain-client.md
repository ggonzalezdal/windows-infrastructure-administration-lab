# Domain Client --- WIN-CL01

## Objective

Deploy the first Windows 11 client workstation and join it to the
`winlab.test` Active Directory domain.

## Environment

  Component           Configuration
  ------------------- -------------------------
  Client              WIN-CL01
  OS                  Windows 11 Pro
  Domain Controller   WIN-DC01
  AD domain           winlab.test
  NetBIOS domain      WINLAB
  Client IPv4         10.20.20.20/24
  DNS server          10.20.20.10
  Network             win-lab / 10.20.20.0/24

## 1. Windows 11 Client Installation

A new VirtualBox VM was created for the Windows client. Windows 11 Pro
was installed using the local administrator account `localadmin`.

VirtualBox Guest Additions were installed and display/integration issues
were corrected.

Initial snapshot:

``` text
00-fresh-windows-install
```

## 2. Static Network Configuration

The client initially received an APIPA `169.254.x.x` address because the
isolated `win-lab` network does not currently provide DHCP.

The Ethernet interface was identified with:

``` powershell
Get-NetAdapter
```

A static IPv4 address was configured:

``` powershell
New-NetIPAddress `
  -InterfaceAlias "Ethernet" `
  -IPAddress 10.20.20.20 `
  -PrefixLength 24
```

Active Directory DNS was configured:

``` powershell
Set-DnsClientServerAddress `
  -InterfaceAlias "Ethernet" `
  -ServerAddresses 10.20.20.10
```

Final configuration:

``` text
WIN-CL01
IPv4: 10.20.20.20/24
DNS:  10.20.20.10
```

No default gateway is currently configured, so Internet access is not
yet available from WIN-CL01.

## 3. Active Directory DNS Verification

Connectivity to the Domain Controller was verified:

``` powershell
ping 10.20.20.10
```

DNS resolution:

``` powershell
Resolve-DnsName WIN-DC01.winlab.test
```

Result:

``` text
WIN-DC01.winlab.test -> 10.20.20.10
```

Active Directory LDAP service discovery:

``` powershell
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.winlab.test
```

The SRV record identified `WIN-DC01.winlab.test` on LDAP TCP/389.

Windows Domain Controller Locator was verified with:

``` powershell
nltest /dsgetdc:winlab.test
```

WIN-CL01 successfully discovered WIN-DC01 as a writable Domain
Controller providing DNS, LDAP, Kerberos, Global Catalog and time
services.

## 4. Computer Rename

The generated Windows hostname was replaced with the lab hostname:

``` powershell
Rename-Computer -NewName "WIN-CL01"
Restart-Computer
```

Verification:

``` powershell
hostname
```

Result:

``` text
WIN-CL01
```

## 5. Domain Join Troubleshooting

The initial PowerShell domain join was attempted with:

``` powershell
Add-Computer -DomainName "winlab.test" -Credential "WINLAB\Administrator"
```

The command initially returned `Access is denied`; a later attempt hung.

Useful checks confirmed that the underlying AD infrastructure was
functioning.

### Domain Controller discovery

``` powershell
nltest /dsgetdc:winlab.test
```

Result: successful.

### Domain authentication

Credentials were tested directly against WIN-DC01:

``` powershell
net use \\WIN-DC01\IPC$ /user:WINLAB\Administrator *
```

Result:

``` text
The command completed successfully.
```

Cleanup:

``` powershell
net use \\WIN-DC01\IPC$ /delete
```

DNS, DC discovery and domain credentials were therefore verified as
working. The root cause of the PowerShell-specific `Add-Computer`
failure was not determined.

## 6. Successful Domain Join

The join was completed using the standard Windows System Properties
interface.

GUI path:

``` text
Settings
-> System
-> About
-> Domain or workgroup
-> Computer Name
-> Change
```

Alternative shortcut:

``` text
Win + R -> sysdm.cpl
```

Configuration:

``` text
Member of: Domain
Domain: winlab.test
Credentials: WINLAB\Administrator
```

Windows confirmed:

``` text
Welcome to the winlab.test domain.
```

WIN-CL01 was restarted.

## 7. Domain Login Verification

At the login screen, `Other user` was selected and the domain account
`WINLAB\Administrator` was used.

Identity verification:

``` powershell
whoami
```

``` text
winlab\administrator
```

Domain environment:

``` powershell
$env:USERDOMAIN
```

``` text
WINLAB
```

Computer domain membership:

``` powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsDomainRole
```

``` text
CsName    CsDomain      CsDomainRole
------    --------      ------------
WIN-CL01  winlab.test   MemberWorkstation
```

WIN-CL01 is successfully operating as an Active Directory member
workstation.

## 8. Local vs Domain Authentication

After joining the domain, local and domain accounts can be specified
explicitly.

Local account:

``` text
.\localadmin
```

or:

``` text
WIN-CL01\localadmin
```

Domain account:

``` text
WINLAB\Administrator
```

A domain user may also use UPN format:

``` text
user@winlab.test
```

The `.\` prefix means authentication against the local computer rather
than Active Directory.

## 9. Computer Object and Domain User Verification

The `WIN-CL01` computer object was verified from `WIN-DC01` using Active
Directory Users and Computers.

After the domain join, the computer account initially appeared in the
default Active Directory `Computers` container.

The computer object was moved into the workstation OU:

``` text
winlab.test
└── WINLAB
    └── Workstations
        └── WIN-CL01
```

This places the workstation within the administrative OU structure and
prepares it for workstation-specific Group Policy configuration.

### Domain User Login --- Alice Morgan

The domain user `alice.morgan` successfully logged into `WIN-CL01`.

During the first interactive login, Active Directory required the
initial password to be changed. A password that did not satisfy the
domain password policy was rejected, demonstrating enforcement of the
domain password requirements.

After setting a compliant password, authentication succeeded.

Verification:

``` powershell
whoami
hostname
$env:USERDOMAIN
$env:LOGONSERVER
```

Result:

``` text
winlab\alice.morgan
WIN-CL01
WINLAB
\\WIN-DC01
```

Windows also created a local user profile for the domain account:

``` text
C:\Users\alice.morgan
```

### Domain User Login --- Bob Carter

The domain user `bob.carter` was also tested interactively on
`WIN-CL01`.

Verification:

``` powershell
whoami
hostname
$env:USERDOMAIN
$env:LOGONSERVER
```

Result:

``` text
winlab\bob.carter
WIN-CL01
WINLAB
\\WIN-DC01
```

Both domain users can therefore authenticate successfully on the
domain-joined workstation using `WIN-DC01` as their logon server.

## 10. Snapshot

Snapshot created after successful domain membership and domain-login
verification:

``` text
01-domain-joined
```

Description:

``` text
WIN-CL01 configured with static IP 10.20.20.20/24 and AD DNS 10.20.20.10. Successfully joined to winlab.test and verified domain login as WINLAB\Administrator. Domain role: MemberWorkstation.
```

## Current State

``` text
WIN-DC01
10.20.20.10
AD DS + DNS
     |
     | winlab.test
     |
WIN-CL01
10.20.20.20
MemberWorkstation
     |
     +-- AD object: WINLAB\Workstations
     +-- alice.morgan domain login verified
     +-- bob.carter domain login verified
```

### Completed

-   Windows 11 Pro client deployed
-   VirtualBox Guest Additions installed
-   Static IPv4 configuration completed
-   AD DNS configured
-   DNS and AD service discovery verified
-   Client renamed to WIN-CL01
-   WIN-CL01 joined to `winlab.test`
-   Domain authentication verified
-   Local/domain login distinction verified
-   `WIN-CL01` computer object verified in Active Directory
-   `WIN-CL01` moved to `WINLAB\Workstations`
-   Interactive domain login verified for `alice.morgan`
-   Interactive domain login verified for `bob.carter`
-   Domain logon server verified as `WIN-DC01`
-   Domain password policy observed during first user login
-   `01-domain-joined` snapshot created

### Pending

-   Provide controlled Internet access to the lab network
-   Begin Group Policy administration
-   Investigate PowerShell `Add-Computer` behavior on a future client
