# Windows Infrastructure Administration Lab

Hands-on lab for building, administering, automating, troubleshooting, and recovering a realistic Windows domain environment.

## Objectives

- Windows Server administration
- Active Directory Domain Services
- DNS and DHCP
- Group Policy
- Windows networking
- File and storage services
- PowerShell administration and automation
- Security, monitoring, backup, and recovery
- Remote administration
- Realistic Windows infrastructure operations

## Lab Platform

- Host: MSI Creator M16 A12UD
- Host OS: Windows 11
- Hypervisor: VirtualBox

## Initial Architecture

| VM           | OS             | Initial State | Future Role                             |
| ------------ | -------------- | ------------- | --------------------------------------- |
| WIN-DC01     | Windows Server | Standalone    | AD DS, DNS, Domain Controller           |
| WIN-SRV01    | Windows Server | Standalone    | Member server / infrastructure services |
| WIN-CLIENT01 | Windows 11     | Standalone    | Domain workstation                      |

## Project Structure

- `docs/` — Lab documentation
- `scripts/` — PowerShell and automation scripts
- `LAB_STATUS.md` — Current lab state
- `CHANGELOG.md` — Project history

## Status

**Phase 1 — Windows Administration Foundations**

Repository initialized. Infrastructure deployment has not started yet.
