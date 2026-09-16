# 02 — Active Directory and DNS

## Purpose

This phase establishes the first Windows Server infrastructure node for the lab.

`WIN-DC01` is configured as the first Domain Controller for the `winlab.test` forest and domain. It also provides authoritative DNS for the Active Directory environment.

The server uses two network interfaces:

- A VirtualBox NAT interface for Internet connectivity.
- An isolated `win-lab` interface for Active Directory and future domain clients.

The final configuration ensures that Active Directory DNS advertises the Domain Controller only through the internal lab network.

---

## Final Architecture

```text
Windows 11 Host
        |
    VirtualBox
        |
     WIN-DC01
  Windows Server 2025
        |
        +-- Ethernet
        |     VirtualBox NAT
        |     IPv4: 10.0.2.15 (DHCP)
        |     Default gateway: 10.0.2.2
        |     Internet access
        |     DNS registration: DISABLED
        |
        +-- Ethernet 2
              VirtualBox Internal Network: win-lab
              IPv4: 10.20.20.10/24
              Default gateway: none
              AD/DNS network
              DNS registration: ENABLED

Active Directory domain: winlab.test
NetBIOS domain: WINLAB
Domain Controller: WIN-DC01
DC FQDN: WIN-DC01.winlab.test
Internal subnet: 10.20.20.0/24
DNS server: 10.20.20.10
```

---

## 1. Server Preparation

The Windows Server VM was renamed to:

```powershell
Rename-Computer -NewName "WIN-DC01"
```

After reboot:

```powershell
hostname
```

Expected result:

```text
WIN-DC01
```

Windows updates were installed before promoting the server to a Domain Controller.

---

## 2. Internal Lab Network

A second VirtualBox network adapter was added to `WIN-DC01`.

VirtualBox configuration:

```text
Adapter 1: NAT
Adapter 2: Internal Network
Network name: win-lab
```

Inside Windows Server, the internal adapter appears as:

```text
Ethernet 2
```

A static IPv4 address was assigned:

```powershell
New-NetIPAddress `
    -InterfaceAlias "Ethernet 2" `
    -IPAddress 10.20.20.10 `
    -PrefixLength 24
```

No default gateway is configured on this interface.

The NAT interface remains responsible for the server's default route and Internet connectivity.

---

## 3. Install Active Directory Domain Services

Verify that the AD DS role is available:

```powershell
Get-WindowsFeature AD-Domain-Services
```

Install AD DS and its management tools:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Verify the forest deployment cmdlet:

```powershell
Get-Command Install-ADDSForest
```

---

## 4. Create the Active Directory Forest

The first forest and domain were created with:

```powershell
Install-ADDSForest `
    -DomainName "winlab.test" `
    -InstallDNS
```

A Directory Services Restore Mode (DSRM) password was configured during promotion.

The server rebooted automatically after promotion.

The resulting environment is:

```text
Forest:       winlab.test
Domain:       winlab.test
NetBIOS:      WINLAB
Domain mode:  Windows2025Domain
Forest mode:  Windows2025Forest
DC:           WIN-DC01.winlab.test
```

---

## 5. Verify Active Directory

Domain information:

```powershell
Get-ADDomain
```

Forest information:

```powershell
Get-ADForest
```

Domain Controller information:

```powershell
Get-ADDomainController
```

`WIN-DC01` is:

- The first Domain Controller.
- A Global Catalog.
- The DNS server for the domain.
- The holder of all FSMO roles in the current single-DC lab.

The Domain Controller's internal IPv4 address is:

```text
10.20.20.10
```

---

## 6. Verify Active Directory DNS

The LDAP Domain Controller discovery record can be queried with:

```powershell
Resolve-DnsName `
    -Type SRV `
    _ldap._tcp.dc._msdcs.winlab.test
```

The SRV record points clients to:

```text
WIN-DC01.winlab.test
LDAP port 389
```

These SRV records are essential because Active Directory clients use DNS to locate Domain Controllers and AD services.

---

## 7. Multihomed Domain Controller DNS Problem

Because `WIN-DC01` has two network interfaces, DNS initially registered addresses from both networks.

The hostname was resolving to:

```text
10.20.20.10   # internal AD network
10.0.2.15     # VirtualBox NAT network
```

IPv6 addresses associated with the other interfaces were also registered.

This was undesirable because future domain clients on `win-lab` should discover the Domain Controller through:

```text
10.20.20.10
```

and not through the VirtualBox NAT interface.

### Initial DNS registration fix

DNS registration was disabled on the NAT interface:

```powershell
Set-DnsClient `
    -InterfaceAlias "Ethernet" `
    -RegisterThisConnectionsAddress $false
```

Verification:

```powershell
Get-DnsClient -InterfaceAlias "Ethernet" |
    Select-Object InterfaceAlias, RegisterThisConnectionsAddress
```

Expected:

```text
Ethernet    False
```

Manual DNS registration with:

```powershell
ipconfig /registerdns
```

correctly registered only `10.20.20.10`.

Restarting Netlogon also did not recreate the NAT record:

```powershell
Restart-Service Netlogon
```

However, after a full reboot, the unwanted NAT IPv4 and IPv6 records returned.

---

## 8. Isolate the Registration Source

The Netlogon DNS registration file was inspected:

```powershell
Get-Content C:\Windows\System32\Config\netlogon.dns
```

The unwanted NAT and IPv6 addresses appeared commented out, while `10.20.20.10` was active.

This confirmed that Netlogon understood the desired NIC registration configuration.

The unwanted DNS records were then manually removed and only the DNS Server service was restarted:

```powershell
Restart-Service DNS
```

The unwanted records immediately returned.

This isolated the remaining registration behavior to the DNS Server service.

---

## 9. Final DNS Server Fix

The DNS Server was listening on addresses from both interfaces.

The available listening addresses were inspected with:

```powershell
$dns = Get-DnsServerSetting -All
$dns.ListeningIPAddress
```

The DNS Server configuration object was then restricted to the internal AD address:

```powershell
$dns = Get-DnsServerSetting -All
$dns.ListeningIPAddress = @("10.20.20.10")
Set-DnsServerSetting -InputObject $dns
```

Verification:

```powershell
(Get-DnsServerSetting -All).ListeningIPAddress
```

Final result:

```text
10.20.20.10
```

The DNS Server now listens only on the internal Active Directory interface.

The NAT NIC remains available for outbound Internet connectivity but is no longer used to advertise the Domain Controller through DNS.

---

## 10. Final Verification

After cleaning the old DNS records, restarting DNS, and performing a complete server reboot:

```powershell
Get-DnsServerResourceRecord `
    -ZoneName "winlab.test" `
    -Name "WIN-DC01"
```

Final result:

```text
WIN-DC01    A    10.20.20.10
```

The unwanted records did not return after reboot.

### Final State

```text
WIN-DC01.winlab.test -> 10.20.20.10
```

The Domain Controller is now advertised exclusively through the internal `win-lab` Active Directory network.

---

## Key Lessons

A multihomed Domain Controller requires careful DNS configuration.

Disabling DNS client registration on the external/NAT NIC prevents normal dynamic registration from that interface:

```powershell
Set-DnsClient `
    -InterfaceAlias "Ethernet" `
    -RegisterThisConnectionsAddress $false
```

In this lab, that alone was not sufficient because the DNS Server service was also listening on all available interfaces.

Restricting DNS Server to the internal AD interface completed the fix:

```powershell
$dns = Get-DnsServerSetting -All
$dns.ListeningIPAddress = @("10.20.20.10")
Set-DnsServerSetting -InputObject $dns
```

The final design separates responsibilities cleanly:

```text
NAT NIC       -> Internet connectivity
Internal NIC  -> Active Directory + DNS
```

---

## Phase Status

**Active Directory and DNS foundation: COMPLETE**

Validated after reboot:

- Windows Server 2025 Domain Controller operational.
- `winlab.test` forest and domain operational.
- AD-integrated DNS operational.
- Internal DC address: `10.20.20.10`.
- NAT interface excluded from DNS registration.
- DNS Server restricted to the internal AD interface.
- `WIN-DC01.winlab.test` resolves only to `10.20.20.10`.

Next phase:

**Active Directory Administration — Organizational Units, users, groups, and identity management.**
