# Changelog

## 2026-09-20

### Group Policy Administration

-   Created the `WINLAB - Workstation Baseline` Group Policy Object.

-   Linked `WINLAB - Workstation Baseline` to the `WINLAB\Workstations`

    OU.

-   Verified computer-side Group Policy processing on `WIN-CL01`.

-   Confirmed with `gpresult /scope computer /r` that `WIN-CL01`

    receives:

    -   `WINLAB - Workstation Baseline`

    -   `Default Domain Policy`

-   Used Group Policy Results in GPMC from `WIN-DC01` to test remote

    Resultant Set of Policy (RSoP) collection.

-   Troubleshot an initial remote Group Policy Results RPC failure.

-   Verified TCP port 135 connectivity from `WIN-DC01` to `WIN-CL01`.

-   Verified the `RpcSs` and `Winmgmt` services were running on

    `WIN-CL01`.

-   Identified disabled Windows Management Instrumentation (WMI)

    firewall rules as the blocker for remote Group Policy Results.

-   Enabled the required WMI firewall access on `WIN-CL01`.

-   Successfully generated remote Group Policy Results for

    `WINLAB\alice.morgan` on `WIN-CL01`.

-   Created the `WINLAB - User Baseline` Group Policy Object.

-   Linked `WINLAB - User Baseline` to the `WINLAB\Users` OU.

-   Configured the user Administrative Template policy:

    `User Configuration → Policies → Administrative Templates → Control Panel → Personalization → Prevent changing desktop background`.

-   Set `Prevent changing desktop background` to `Enabled`.

-   Verified the pre-policy state in which Alice Morgan could still

    access the Windows 11 desktop-background controls.

-   Forced user-side policy processing locally with:

``` powershell

gpupdate /target:user /force
```

-   Verified with `gpresult /r` that `WINLAB\alice.morgan` received

    `WINLAB - User Baseline`.

-   Confirmed that Alice's existing wallpaper remained in place while

    the desktop-background controls became disabled.

-   Initiated a centralized Group Policy Update from `WIN-DC01` against

    the `WINLAB\Workstations` OU.

-   Troubleshot remote Group Policy Update error `8007071a`.

-   Inspected the Remote Scheduled Tasks Management and Remote Service

    Management firewall rule groups on `WIN-CL01`.

-   Identified disabled Domain-profile Remote Scheduled Tasks Management

    RPC rules as the blocker.

-   Enabled only the Domain-profile Remote Scheduled Tasks Management

    rules:

``` powershell

Get-NetFirewallRule -DisplayGroup "Remote Scheduled Tasks Management" |

    Where-Object { $\_.Profile -eq "Domain" } |

    Enable-NetFirewallRule
```

-   Verified that the Domain-profile RPC and RPC-EPMAP rules were

    enabled while the Private/Public equivalents remained disabled.

-   Left Remote Service Management rules disabled because they were not

    required for the operation.

-   Retried the centralized Group Policy Update from `WIN-DC01`.

-   Successfully refreshed Group Policy remotely on `WIN-CL01`.

-   Verified the remote refresh from Alice Morgan's session with

    `gpresult /r`.

-   Confirmed the policy refresh timestamp of `9/20/2026 at 8:19:50 AM`

    and `WIN-DC01.winlab.test` as the policy source.

-   Demonstrated the distinction between:

    -   Computer Configuration scoped through the computer object's OU.

    -   User Configuration scoped through the user object's OU.

    -   Remote Group Policy Results using WMI.

    -   Remote Group Policy Update using Remote Scheduled Tasks/RPC.

-   Added `docs/05-group-policy.md`.

-   Updated `README.md` and `LAB_STATUS.md` to reflect completion of the

    Group Policy foundation.

-   Completed Phase 5 --- Group Policy Administration.

-   Prepared the lab for Phase 6 --- Member Server, File Services, and

    AGDLP Permissions.

### Member Server Deployment --- WIN-SRV01

-   Created the `WIN-SRV01` VirtualBox virtual machine.
-   Installed Windows Server 2025 Standard Evaluation (Desktop
    Experience).
-   Configured the VM with 4 GB RAM, 2 vCPUs, EFI, and a 50 GB
    dynamically allocated VDI.
-   Connected `WIN-SRV01` to the VirtualBox Internal Network `win-lab`.
-   Installed VirtualBox Guest Additions.
-   Configured Spanish (Spain) keyboard input while retaining the
    English Windows environment.
-   Corrected the user input-method configuration so the Spanish
    keyboard layout was active in the current user profile.
-   Configured Num Lock startup behavior for the current user and logon
    environment.
-   Identified the initial APIPA address `169.254.139.122` caused by the
    absence of DHCP on `win-lab`.
-   Configured static IPv4 address `10.20.20.30/24`.
-   Configured `WIN-DC01` (`10.20.20.10`) as the DNS server.
-   Left the default gateway unset on the isolated internal network.
-   Verified IPv4 connectivity between `WIN-SRV01` and `WIN-DC01`.
-   Verified `winlab.test` DNS resolution to `10.20.20.10`.
-   Verified LDAP connectivity to `WIN-DC01` on TCP port 389.
-   Renamed the server from the automatically generated hostname
    `WIN-8J5PDU25GOV` to `WIN-SRV01`.
-   Verified the local Administrator identity changed to
    `WIN-SRV01\Administrator` after the rename.
-   Verified Domain Controller discovery with
    `nltest /dsgetdc:winlab.test`.
-   Confirmed discovery of `WIN-DC01.winlab.test` at `10.20.20.10`.
-   Successfully joined `WIN-SRV01` to `winlab.test` using PowerShell
    `Add-Computer`.
-   Restarted the server and authenticated as `WINLAB\Administrator`.
-   Verified `WIN-SRV01` as an Active Directory `MemberServer`.
-   Verified the `WIN-SRV01` computer object was initially created in
    the default `Computers` container.
-   Moved the computer object into
    `OU=Servers,OU=WINLAB,DC=winlab,DC=test` using `Move-ADObject`.
-   Verified the final Distinguished Name:
    `CN=WIN-SRV01,OU=Servers,OU=WINLAB,DC=winlab,DC=test`.
-   Confirmed the server's network profile changed from
    `Unidentified network` to `winlab.test`.
-   Verified the domain trust relationship with
    `Test-ComputerSecureChannel -Verbose`.
-   Confirmed the secure channel between `WIN-SRV01` and `winlab.test`
    is healthy.
-   Created the `00-win-srv01-domain-member` VirtualBox snapshot.
-   Created the `06-win-srv01-domain-joined` VirtualBox snapshot on
    `WIN-DC01`.
-   Added `docs/06-member-server.md`.
-   Updated `README.md` and `LAB_STATUS.md` to reflect the deployed
    member-server baseline.
-   Completed the member-server foundation portion of Phase 6.
-   Prepared `WIN-SRV01` for dedicated storage, File Services, SMB
    shares, and full AGDLP permission validation.

## 2026-09-18

### Domain Client Deployment and Domain Join

-   Deployed the first Windows 11 Pro domain client, `WIN-CL01`, in

    VirtualBox.

-   Installed VirtualBox Guest Additions and configured guest

    integration.

-   Created the initial `00-fresh-windows-install` snapshot.

-   Renamed the workstation to `WIN-CL01`.

-   Connected `WIN-CL01` to the internal VirtualBox network `win-lab`.

-   Identified the initial APIPA configuration caused by the absence of

    DHCP on the internal network.

-   Configured static IPv4 address `10.20.20.20/24`.

-   Configured `WIN-DC01` (`10.20.20.10`) as the client's DNS server.

-   Verified connectivity between `WIN-CL01` and `WIN-DC01`.

-   Verified DNS resolution for `WIN-DC01.winlab.test`.

-   Verified Active Directory LDAP SRV service discovery.

-   Verified Windows Domain Controller discovery with

    `nltest /dsgetdc:winlab.test`.

-   Verified domain credentials against `WIN-DC01`.

-   Troubleshot a PowerShell `Add-Computer` domain-join failure and

    subsequent hang.

-   Confirmed that DNS, DC discovery, required connectivity, and domain

    credentials were operational.

-   Completed the domain join successfully through Windows System

    Properties (`sysdm.cpl`).

-   Restarted `WIN-CL01` and successfully authenticated as

    `WINLAB\Administrator`.

-   Verified `WIN-CL01` as a `MemberWorkstation` in the `winlab.test`

    domain.

-   Verified the distinction between local (`.\localadmin`) and domain

    (`WINLAB\user`) authentication.

-   Verified the `WIN-CL01` computer object in Active Directory Users

    and Computers.

-   Moved the `WIN-CL01` computer object from the default `Computers`

    container into `WINLAB\Workstations`.

-   Successfully completed Alice Morgan's first interactive domain login

    on `WIN-CL01`.

-   Observed enforcement of the domain password policy when Alice's

    initial replacement password was rejected for not meeting policy

    requirements.

-   Verified Alice Morgan's domain identity as `winlab\alice.morgan`.

-   Verified `WINLAB` as Alice Morgan's user domain and `WIN-DC01` as

    her logon server.

-   Successfully completed Bob Carter's interactive domain login on

    `WIN-CL01`.

-   Verified Bob Carter's domain identity as `winlab\bob.carter`.

-   Verified `WINLAB` as Bob Carter's user domain and `WIN-DC01` as his

    logon server.

-   Confirmed that standard domain users can authenticate successfully

    on the domain-joined workstation.

-   Documented that `WIN-CL01` currently has no default gateway and

    therefore no Internet connectivity.

-   Created the `01-domain-joined` VirtualBox snapshot.

-   Added and finalized `docs/04-domain-client.md`.

-   Updated `README.md` and `LAB_STATUS.md` to reflect computer-object

    placement and standard domain-user validation.

-   Completed Phase 4 --- Domain Client Deployment and Domain Join.

-   Prepared the lab for Phase 5 --- Group Policy Administration.

## 2026-09-17

### Active Directory Administration

-   Explored the default `winlab.test` Active Directory structure using

    Active Directory Users and Computers (ADUC).

-   Distinguished built-in Active Directory containers from

    Organizational Units (OUs).

-   Created the custom `WINLAB` OU hierarchy:

    -   `Users`

    -   `Groups`

    -   `Servers`

    -   `Workstations`

-   Inspected Organizational Units and Distinguished Names with

    `Get-ADOrganizationalUnit`.

-   Created the `Alice Morgan` domain user through ADUC.

-   Created the `Bob Carter` domain user with `New-ADUser`.

-   Inspected user properties with `Get-ADUser`.

-   Practiced disabling and re-enabling domain accounts with

    `Disable-ADAccount` and `Enable-ADAccount`.

-   Practiced administrative password resets with

    `Set-ADAccountPassword`.

-   Configured password change at next logon with `Set-ADUser`.

-   Added directory attributes to user accounts and practiced filtering

    users with PowerShell.

-   Created the `GG-IT` Global Security group.

-   Added Alice Morgan and Bob Carter to `GG-IT`.

-   Practiced moving an Active Directory object between OUs with

    `Move-ADObject`.

-   Created the `DL-IT-Modify` Domain Local Security group.

-   Nested `GG-IT` inside `DL-IT-Modify`.

-   Introduced and implemented the group-nesting portion of the AGDLP

    (`Accounts → Global → Domain Local → Permissions`) model.

-   Added `docs/03-active-directory-administration.md`.

-   Updated `README.md` and `LAB_STATUS.md` to reflect completion of the

    Active Directory administration foundation.

-   Prepared the lab for deployment of the first domain-joined Windows

    client, `WIN-CL01`.

## 2026-09-16

### Active Directory and DNS

-   Installed Active Directory Domain Services and management tools on

    `WIN-DC01`.

-   Created the `winlab.test` Active Directory forest and domain.

-   Configured `WINLAB` as the NetBIOS domain name.

-   Promoted `WIN-DC01` as the first Domain Controller.

-   Installed and configured AD-integrated DNS.

-   Verified domain, forest, Global Catalog, FSMO roles, and AD DNS SRV

    records.

-   Investigated DNS registration behavior on the multihomed Domain

    Controller.

-   Disabled DNS registration on the VirtualBox NAT interface.

-   Isolated unwanted NAT/IPv6 record publication to the DNS Server

    service.

-   Restricted DNS Server listening to the internal AD address

    `10.20.20.10`.

-   Verified after a full reboot that `WIN-DC01.winlab.test` publishes

    only `10.20.20.10`.

-   Added `docs/01-windows-server-foundation.md`.

-   Added `docs/02-active-directory-dns.md`.

-   Updated `README.md` and `LAB_STATUS.md` to reflect the deployed

    infrastructure.

## 2026-09-14

### Windows Server Foundation

-   Initialized Windows Infrastructure Administration Lab.

-   Created initial repository structure.

-   Added project documentation and Git exclusions.

-   Created the `WIN-DC01` VirtualBox virtual machine.

-   Installed Windows Server 2025 Standard Evaluation.

-   Installed VirtualBox Guest Additions.

-   Enabled bidirectional clipboard integration.

-   Configured shared folder access between the Windows 11 host and

    `WIN-DC01`.

-   Renamed the server to `WIN-DC01`.

-   Configured Spanish keyboard input while retaining the English

    Windows display language.

-   Configured Num Lock startup behavior.

-   Installed Windows updates before infrastructure role deployment.

-   Configured the VirtualBox NAT interface for Internet connectivity.

-   Added the `win-lab` VirtualBox Internal Network interface.

-   Assigned static IPv4 address `10.20.20.10/24` to the internal

    interface.

-   Verified Internet and HTTPS connectivity through the NAT interface.
