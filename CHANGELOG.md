# Changelog

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
