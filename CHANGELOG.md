# Changelog

## 2026-09-16

### Active Directory and DNS

- Installed Active Directory Domain Services and management tools on `WIN-DC01`.
- Created the `winlab.test` Active Directory forest and domain.
- Configured `WINLAB` as the NetBIOS domain name.
- Promoted `WIN-DC01` as the first Domain Controller.
- Installed and configured AD-integrated DNS.
- Verified domain, forest, Global Catalog, FSMO roles, and AD DNS SRV records.
- Investigated DNS registration behavior on the multihomed Domain Controller.
- Disabled DNS registration on the VirtualBox NAT interface.
- Isolated unwanted NAT/IPv6 record publication to the DNS Server service.
- Restricted DNS Server listening to the internal AD address `10.20.20.10`.
- Verified after a full reboot that `WIN-DC01.winlab.test` publishes only `10.20.20.10`.
- Added `docs/01-windows-server-foundation.md`.
- Added `docs/02-active-directory-dns.md`.
- Updated `README.md` and `LAB_STATUS.md` to reflect the deployed infrastructure.

## 2026-09-14

### Windows Server Foundation

- Initialized Windows Infrastructure Administration Lab.
- Created initial repository structure.
- Added project documentation and Git exclusions.
- Created the `WIN-DC01` VirtualBox virtual machine.
- Installed Windows Server 2025 Standard Evaluation.
- Installed VirtualBox Guest Additions.
- Enabled bidirectional clipboard integration.
- Configured shared folder access between the Windows 11 host and `WIN-DC01`.
- Renamed the server to `WIN-DC01`.
- Configured Spanish keyboard input while retaining the English Windows display language.
- Configured Num Lock startup behavior.
- Installed Windows updates before infrastructure role deployment.
- Configured the VirtualBox NAT interface for Internet connectivity.
- Added the `win-lab` VirtualBox Internal Network interface.
- Assigned static IPv4 address `10.20.20.10/24` to the internal interface.
- Verified Internet and HTTPS connectivity through the NAT interface.
