# Group Policy Administration

## Overview

This phase introduced Group Policy administration in the `winlab.test`
domain. The lab implemented separate computer and user baselines,
validated policy processing locally and remotely, and troubleshot the
Windows Firewall requirements for centralized Group Policy
administration.

## Environment

-   Domain: `winlab.test`
-   Domain Controller: `WIN-DC01.winlab.test`
-   Domain workstation: `WIN-CL01.winlab.test`
-   Workstation IP: `10.20.20.20`
-   DNS / Domain Controller IP: `10.20.20.10`
-   Test user: `WINLAB\\alice.morgan`

Relevant OU structure:

``` text
winlab.test
└── WINLAB
    ├── Groups
    ├── Servers
    ├── Users
    │   ├── Alice Morgan
    │   └── Bob Carter
    └── Workstations
        └── WIN-CL01
```

## Computer Group Policy Baseline

A GPO named `WINLAB - Workstation Baseline` was created and linked to:

``` text
OU=Workstations,OU=WINLAB,DC=winlab,DC=test
```

The link was enabled and not enforced.

Computer policy processing was verified on `WIN-CL01`:

``` powershell
gpresult /scope computer /r
```

The result confirmed:

``` text
COMPUTER SETTINGS
CN=WIN-CL01,OU=Workstations,OU=WINLAB,DC=winlab,DC=test

Applied Group Policy Objects
    WINLAB - Workstation Baseline
    Default Domain Policy
```

This confirmed that the workstation object was in the correct OU and
receiving the expected computer policies.

## Remote Group Policy Results

Group Policy Results in GPMC was used from `WIN-DC01` to collect
Resultant Set of Policy (RSoP) information for `WIN-CL01`.

The initial attempt failed with an RPC connectivity error. Basic RPC
connectivity was verified from the domain controller:

``` powershell
Test-NetConnection WIN-CL01 -Port 135
```

The RPC endpoint mapper was reachable. On `WIN-CL01`, the RPC and WMI
services were also verified:

``` powershell
Get-Service Winmgmt, RpcSs | Select-Object Name, Status, StartType
```

Both services were running.

The WMI firewall rules were inspected:

``` powershell
Get-NetFirewallRule -DisplayGroup "Windows Management Instrumentation (WMI)" |
    Select-Object DisplayName, Enabled, Profile, Direction
```

The required WMI firewall rules were disabled. After enabling the
appropriate WMI firewall access, Group Policy Results successfully
collected RSoP information remotely from `WIN-DC01`.

A result set for `WINLAB\\alice.morgan` on `WIN-CL01` was successfully
generated in GPMC.

## User Group Policy Baseline

A second GPO was created:

``` text
WINLAB - User Baseline
```

It was linked specifically to:

``` text
OU=Users,OU=WINLAB,DC=winlab,DC=test
```

This separated user and computer policy scope:

``` text
WIN-CL01
    ↓
OU=Workstations
    ↓
WINLAB - Workstation Baseline

Alice Morgan / Bob Carter
    ↓
OU=Users
    ↓
WINLAB - User Baseline
```

Computer policies follow the computer object's OU placement, while user
policies follow the user object's OU placement.

## User Policy Test: Prevent Desktop Background Changes

The following Administrative Template setting was configured:

``` text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Personalization
                └── Prevent changing desktop background
```

The setting was set to `Enabled`.

Alice Morgan already had a custom desktop background before the policy
was applied. The existing wallpaper remained in place while the user
lost the ability to modify the background.

While logged on as `WINLAB\\alice.morgan`, user policy was refreshed
manually:

``` powershell
gpupdate /target:user /force
```

The command completed successfully. The resultant policy was then
checked:

``` powershell
gpresult /r
```

The output confirmed:

``` text
USER SETTINGS
CN=Alice Morgan,OU=Users,OU=WINLAB,DC=winlab,DC=test

Group Policy was applied from:
WIN-DC01.winlab.test

Applied Group Policy Objects
    WINLAB - User Baseline
```

After policy processing, the Windows 11 desktop background controls were
disabled while Alice's existing wallpaper remained unchanged.

## Remote Group Policy Update

After validating local policy processing, a remote policy refresh was
initiated centrally from `WIN-DC01`.

In Group Policy Management, the `Workstations` OU was right-clicked and
`Group Policy Update...` was selected. GPMC detected one computer in the
OU: `WIN-CL01.winlab.test`.

The first remote update attempt failed with:

``` text
Error Code: 8007071a
```

### Remote Scheduled Tasks Firewall Investigation

The relevant firewall rules on `WIN-CL01` were inspected:

``` powershell
Get-NetFirewallRule |
    Where-Object {
        $_.DisplayGroup -like "*Remote Scheduled Tasks*" -or
        $_.DisplayGroup -like "*Remote Service Management*"
    } |
    Select-Object DisplayName, DisplayGroup, Enabled, Profile, Direction
```

The Domain-profile Remote Scheduled Tasks Management rules were
disabled:

``` text
Remote Scheduled Tasks Management (RPC)        False  Domain  Inbound
Remote Scheduled Tasks Management (RPC-EPMAP)  False  Domain  Inbound
```

Only the Domain-profile rules were enabled:

``` powershell
Get-NetFirewallRule -DisplayGroup "Remote Scheduled Tasks Management" |
    Where-Object { $_.Profile -eq "Domain" } |
    Enable-NetFirewallRule
```

Verification:

``` powershell
Get-NetFirewallRule -DisplayGroup "Remote Scheduled Tasks Management" |
    Select-Object DisplayName, Enabled, Profile, Direction
```

Result:

``` text
Remote Scheduled Tasks Management (RPC)        True   Domain           Inbound
Remote Scheduled Tasks Management (RPC-EPMAP)  True   Domain           Inbound
Remote Scheduled Tasks Management (RPC)        False  Private, Public  Inbound
Remote Scheduled Tasks Management (RPC-EPMAP)  False  Private, Public  Inbound
```

The `Remote Service Management` rules were not enabled because they were
not required for this operation.

## Successful Centralized Policy Refresh

The remote Group Policy Update was retried from `WIN-DC01`.

GPMC reported:

``` text
Completed (1 of 1)

Succeeded (1)
WIN-CL01.winlab.test
```

The result was subsequently verified on `WIN-CL01` as Alice:

``` powershell
gpresult /r
```

The output showed:

``` text
Last time Group Policy was applied: 9/20/2026 at 8:19:50 AM
Group Policy was applied from:      WIN-DC01.winlab.test

Applied Group Policy Objects
    WINLAB - User Baseline
```

The timestamp corresponded to the remote refresh initiated from the
domain controller.

## Key Lessons

-   A GPO does not affect users or computers until it is linked to an
    appropriate site, domain, or OU.
-   Computer Configuration follows the location of the computer object
    in Active Directory.
-   User Configuration follows the location of the user object in Active
    Directory.
-   `gpupdate /target:user /force` forces user-side policy processing
    locally.
-   `gpresult /r` and `gpresult /scope computer /r` provide direct
    evidence of applied GPOs.
-   GPMC Group Policy Results provides centralized RSoP reporting.
-   Remote RSoP and Remote Group Policy Update use different
    remote-management mechanisms and can have different firewall
    requirements.
-   WMI firewall access was required for successful remote Group Policy
    Results.
-   Remote Scheduled Tasks Management RPC access was required for
    successful GPMC Remote Group Policy Update.
-   Firewall access should be enabled only for the required profile and
    service rather than broadly opening management services.

## Final State

At the end of this phase:

-   `WINLAB - Workstation Baseline` is linked to `WINLAB\\Workstations`.
-   `WIN-CL01` successfully receives the workstation computer baseline.
-   `WINLAB - User Baseline` is linked to `WINLAB\\Users`.
-   Alice Morgan successfully receives the user baseline.
-   `Prevent changing desktop background` is enforced for Alice.
-   Local user policy refresh with `gpupdate` works.
-   Local `gpresult` verification works.
-   Remote Group Policy Results from `WIN-DC01` works.
-   Remote Group Policy Update from `WIN-DC01` to `WIN-CL01` works.
-   Required WMI and Remote Scheduled Tasks firewall access has been
    identified and configured for remote administration.

The lab now has a functioning foundation for centrally managed Windows
workstation and user configuration through Active Directory Group
Policy.
