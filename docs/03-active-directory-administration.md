# Phase 3 — Active Directory Administration

## Objective

Build the initial Active Directory organizational and identity structure for the Windows Infrastructure Administration Lab and practice common administration tasks using both **Active Directory Users and Computers (ADUC)** and **PowerShell**.

This phase establishes the directory structure and identities that will later be used by domain-joined workstations, Group Policy, and resource permissions.

---

## Environment

- **Domain:** `winlab.test`
- **Domain Controller:** `WIN-DC01`
- **DC IP address:** `10.20.20.10`
- **Management tools:**
  - Active Directory Users and Computers (`dsa.msc`)
  - Active Directory PowerShell module

---

## 1. Exploring the Active Directory Structure

Active Directory Users and Computers was opened with:

```text
dsa.msc
```

The default domain structure was inspected, including:

- `Builtin`
- `Computers`
- `Domain Controllers`
- `ForeignSecurityPrincipals`
- `Managed Service Accounts`
- `Users`

`WIN-DC01` was confirmed inside the **Domain Controllers** OU and operating as a Global Catalog server.

### Containers vs Organizational Units

An important distinction was established:

- Default objects such as `Users` and `Computers` are **containers**.
- `Domain Controllers` is an **Organizational Unit (OU)**.
- Custom OUs provide a structured way to organize directory objects and can later be targeted by Group Policy.

---

## 2. Custom OU Structure

A custom root OU named `WINLAB` was created under the domain.

The following child OUs were then created:

```text
winlab.test
└── WINLAB
    ├── Groups
    ├── Servers
    ├── Users
    └── Workstations
```

Protection from accidental deletion was left enabled when creating the OUs.

The resulting structure was verified with PowerShell:

```powershell
Get-ADOrganizationalUnit -Filter *
```

A cleaner view can be produced with:

```powershell
Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

To search only inside the custom hierarchy:

```powershell
Get-ADOrganizationalUnit -Filter * `
    -SearchBase "OU=WINLAB,DC=winlab,DC=test" |
    Select-Object Name, DistinguishedName
```

Example distinguished name:

```text
OU=Users,OU=WINLAB,DC=winlab,DC=test
```

This identifies the exact location of the OU inside the directory tree.

---

## 3. Domain User Administration

### Alice Morgan — ADUC

The first lab user was created through Active Directory Users and Computers:

- **Name:** Alice Morgan
- **sAMAccountName:** `alice.morgan`
- **UPN:** `alice.morgan@winlab.test`
- **OU:** `WINLAB/Users`
- **Account:** Enabled
- **Password:** User must change password at next logon

The account was inspected with:

```powershell
Get-ADUser alice.morgan -Properties * |
    Select-Object Name, GivenName, Surname,
                  SamAccountName, UserPrincipalName,
                  Enabled, DistinguishedName
```

Resulting DN:

```text
CN=Alice Morgan,OU=Users,OU=WINLAB,DC=winlab,DC=test
```

### Bob Carter — PowerShell

A second user was created entirely through PowerShell.

The password was collected as a `SecureString`:

```powershell
$password = Read-Host "Enter temporary password" -AsSecureString
```

The account was then created:

```powershell
New-ADUser `
    -Name "Bob Carter" `
    -GivenName "Bob" `
    -Surname "Carter" `
    -SamAccountName "bob.carter" `
    -UserPrincipalName "bob.carter@winlab.test" `
    -Path "OU=Users,OU=WINLAB,DC=winlab,DC=test" `
    -AccountPassword $password `
    -Enabled $true `
    -ChangePasswordAtLogon $true
```

Verification:

```powershell
Get-ADUser bob.carter |
    Select-Object Name, SamAccountName, Enabled, DistinguishedName
```

Resulting DN:

```text
CN=Bob Carter,OU=Users,OU=WINLAB,DC=winlab,DC=test
```

---

## 4. Account Lifecycle Operations

Bob's account was used to practice common identity administration operations.

### Disable an account

```powershell
Disable-ADAccount -Identity bob.carter
```

Verification:

```powershell
Get-ADUser bob.carter |
    Select-Object Name, Enabled
```

### Re-enable an account

```powershell
Enable-ADAccount -Identity bob.carter
```

### Reset a password

```powershell
$newPassword = Read-Host "Enter new temporary password" -AsSecureString
```

```powershell
Set-ADAccountPassword `
    -Identity bob.carter `
    -Reset `
    -NewPassword $newPassword
```

Force password change at next logon:

```powershell
Set-ADUser bob.carter -ChangePasswordAtLogon $true
```

Disabling an account preserves the AD object and its memberships while preventing authentication. This is different from deleting the account.

---

## 5. User Attributes and Queries

Bob's directory attributes were modified with:

```powershell
Set-ADUser bob.carter `
    -Department "IT" `
    -Title "Systems Administrator" `
    -Company "WINLAB"
```

Verification:

```powershell
Get-ADUser bob.carter -Properties Department,Title,Company |
    Select-Object Name,Department,Title,Company
```

Active Directory users can also be queried using filters. For example:

```powershell
Get-ADUser -Filter 'Department -eq "IT"' -Properties Department |
    Select-Object Name,SamAccountName,Department
```

This demonstrates how PowerShell can scale administration beyond individual accounts.

---

## 6. Security Groups

### Global Security Group — GG-IT

A Global Security group was created to represent IT department users:

```powershell
New-ADGroup `
    -Name "GG-IT" `
    -SamAccountName "GG-IT" `
    -GroupCategory Security `
    -GroupScope Global `
    -Path "OU=Groups,OU=WINLAB,DC=winlab,DC=test" `
    -Description "IT department users"
```

Alice and Bob were added as members:

```powershell
Add-ADGroupMember -Identity "GG-IT" -Members alice.morgan,bob.carter
```

Membership verification:

```powershell
Get-ADGroupMember "GG-IT"
```

Expected membership:

```text
GG-IT
├── Alice Morgan
└── Bob Carter
```

### Moving an AD Object

During the exercise, `GG-IT` was initially created in the `Users` OU because the wrong `-Path` was supplied.

Rather than deleting and recreating the object, it was moved to the correct OU:

```powershell
Move-ADObject `
    -Identity "CN=GG-IT,OU=Users,OU=WINLAB,DC=winlab,DC=test" `
    -TargetPath "OU=Groups,OU=WINLAB,DC=winlab,DC=test"
```

This demonstrated that the `-Path` parameter determines where an AD object is created; Active Directory does not automatically place a group into an OU named `Groups`.

---

## 7. Group Scopes and AGDLP

The lab introduced the **AGDLP** permissions model:

```text
A → G → DL → P
```

Where:

- **A — Accounts:** users such as Alice and Bob
- **G — Global Groups:** groups representing roles or departments
- **DL — Domain Local Groups:** groups representing access to resources
- **P — Permissions:** permissions assigned to the Domain Local group

The current lab model is:

```text
Alice Morgan ─┐
              ├── GG-IT ──→ DL-IT-Modify ──→ Future resource permission
Bob Carter ───┘
```

### Domain Local Security Group

A Domain Local group was created:

```powershell
New-ADGroup `
    -Name "DL-IT-Modify" `
    -SamAccountName "DL-IT-Modify" `
    -GroupCategory Security `
    -GroupScope DomainLocal `
    -Path "OU=Groups,OU=WINLAB,DC=winlab,DC=test" `
    -Description "Modify access to IT resources"
```

The Global group was nested inside it:

```powershell
Add-ADGroupMember `
    -Identity "DL-IT-Modify" `
    -Members "GG-IT"
```

Verification:

```powershell
Get-ADGroupMember "DL-IT-Modify"
```

The direct member is `GG-IT`, while Alice and Bob are indirect members through that Global group.

This separates **identity/role membership** from **resource permissions** and prepares the lab for future file-share and NTFS permission exercises.

---

## 8. PowerShell Commands Practiced

| Command | Purpose |
|---|---|
| `Get-ADOrganizationalUnit` | Query organizational units |
| `Get-ADUser` | Query domain users |
| `New-ADUser` | Create domain users |
| `Set-ADUser` | Modify user properties |
| `Disable-ADAccount` | Disable an account |
| `Enable-ADAccount` | Enable an account |
| `Set-ADAccountPassword` | Set or reset an AD account password |
| `New-ADGroup` | Create an AD group |
| `Add-ADGroupMember` | Add users/groups to a group |
| `Get-ADGroupMember` | Inspect group membership |
| `Move-ADObject` | Move an AD object between containers/OUs |
| `Select-Object` | Select properties for cleaner PowerShell output |

---

## 9. Current Active Directory State

```text
winlab.test
└── WINLAB
    ├── Groups
    │   ├── GG-IT
    │   │   ├── Alice Morgan
    │   │   └── Bob Carter
    │   │
    │   └── DL-IT-Modify
    │       └── GG-IT
    │
    ├── Servers
    │
    ├── Users
    │   ├── Alice Morgan
    │   └── Bob Carter
    │
    └── Workstations
```

The `Servers` and `Workstations` OUs are currently empty and ready for future domain members.

---

## Phase 3 Result

Phase 3 established a clean Active Directory organizational model and introduced practical identity and group administration.

The lab now has:

- A custom OU hierarchy
- Domain user accounts
- Global and Domain Local security groups
- Nested group membership
- Basic account lifecycle administration
- User attribute management
- PowerShell-based AD querying and administration
- An initial AGDLP permissions model

The Active Directory environment is now ready for its first domain-joined workstation.

---

## Next Phase

### Windows Client Deployment and Domain Join

Next steps:

1. Create `WIN-CL01`.
2. Attach it to the `win-lab` network.
3. Configure its DNS server as `10.20.20.10` (`WIN-DC01`).
4. Verify DNS and domain controller connectivity.
5. Join `WIN-CL01` to `winlab.test`.
6. Reboot the workstation.
7. Log in using a domain account such as Alice or Bob.
8. Move the resulting `WIN-CL01` computer object into:

```text
OU=Workstations,OU=WINLAB,DC=winlab,DC=test
```

This will provide the first managed Windows client and prepare the lab for **Group Policy administration**.
