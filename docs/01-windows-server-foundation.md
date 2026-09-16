# 01 — Windows Server Foundation

## Purpose

This phase establishes the base Windows Server virtual machine and networking required for the Windows Infrastructure Administration Lab.

The goal is to prepare `WIN-DC01` as a stable Windows Server 2025 system before installing Active Directory Domain Services.

---

## 1. Lab Platform

```text
Host:        MSI Creator M16 A12UD
Host OS:     Windows 11
Hypervisor:  VirtualBox 7.2.x
Guest:       Windows Server 2025 Standard Evaluation
VM:          WIN-DC01
```

`WIN-DC01` will become the first Domain Controller and DNS server for the lab.

---

## 2. Virtual Machine Preparation

Windows Server 2025 Standard Evaluation was installed in VirtualBox.

VirtualBox Guest Additions were installed and verified.

Integration features configured:

- Bidirectional clipboard.
- Host/guest shared folder.

Shared folder configuration:

```text
Host path:   C:\VM-Share
Guest path:  \\VBOXSVR\VM-Share
```

The shared folder was configured with auto-mount enabled and read-only disabled.

---

## 3. Server Identity

The original automatically generated Windows hostname was replaced with the lab naming convention:

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

---

## 4. Regional and Console Configuration

The Windows display language was retained in English.

Spanish keyboard input was configured as the default keyboard layout:

```powershell
Set-WinDefaultInputMethodOverride -InputTip "0C0A:0000040A"
```

Num Lock startup behavior was also configured so that it remains enabled after reboot.

The default keyboard registry value was set to:

```text
HKEY_USERS\.DEFAULT\Control Panel\Keyboard
InitialKeyboardIndicators = 2
```

These settings were verified after reboot.

---

## 5. Windows Updates

Pending Windows updates were installed before adding infrastructure roles.

This provided a clean and updated baseline before Domain Controller promotion.

---

## 6. Network Architecture

`WIN-DC01` uses two network adapters.

```text
WIN-DC01
 |
 +-- Ethernet
 |    VirtualBox NAT
 |    IPv4: 10.0.2.15/24 (DHCP)
 |    Gateway: 10.0.2.2
 |    Purpose: Internet connectivity
 |
 +-- Ethernet 2
      VirtualBox Internal Network: win-lab
      IPv4: 10.20.20.10/24 (static)
      Gateway: none
      Purpose: Windows infrastructure/domain network
```

The design deliberately separates Internet connectivity from internal domain traffic.

---

## 7. Internal Lab Network

A second VirtualBox adapter was added:

```text
Attached to:  Internal Network
Network name: win-lab
Adapter type: Intel PRO/1000 MT Desktop
```

Windows detected it as:

```text
Ethernet 2
```

Because an isolated VirtualBox Internal Network has no DHCP service by default, the interface initially received an APIPA address (`169.254.x.x`).

A static address was then configured:

```powershell
New-NetIPAddress `
    -InterfaceAlias "Ethernet 2" `
    -IPAddress 10.20.20.10 `
    -PrefixLength 24
```

No default gateway was assigned to `Ethernet 2`.

The resulting internal subnet is:

```text
10.20.20.0/24
```

---

## 8. Routing Design

Only the NAT interface owns a default gateway.

```text
Ethernet
  10.0.2.15
  Default gateway: 10.0.2.2

Ethernet 2
  10.20.20.10
  Default gateway: none
```

This ensures:

```text
Internet traffic -> NAT interface
Lab/domain traffic -> win-lab interface
```

Future Windows domain clients will attach to `win-lab`.

---

## 9. Connectivity Verification

External IPv4 connectivity was tested with:

```powershell
Test-NetConnection 8.8.8.8
```

HTTPS connectivity and name resolution were tested with:

```powershell
Test-NetConnection microsoft.com -Port 443
```

Traffic correctly used the NAT interface and source address `10.0.2.15`.

---

## 10. Foundation State

At the end of this phase:

```text
WIN-DC01
├── Windows Server 2025 installed
├── Guest Additions installed
├── Host integration configured
├── Hostname configured
├── Keyboard/console settings configured
├── Windows updates installed
├── NAT Internet connectivity operational
└── Internal win-lab network operational
     └── 10.20.20.10/24
```

The server is ready for Active Directory Domain Services installation and promotion.

---

## Phase Status

**Windows Server Foundation: COMPLETE**

Next phase:

**Active Directory Domain Services and DNS**
