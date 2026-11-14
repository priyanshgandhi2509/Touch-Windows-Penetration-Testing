# 🎯 Technical Penetration Testing Assessment: Target "Touch" (Windows Server 2022)

[![Institution](https://img.shields.io/badge/Academic-National_College_of_Ireland-003366.svg?style=flat-square)](https://www.ncirl.ie/)
[![Programme](https://img.shields.io/badge/Programme-MSc_in_Cybersecurity-blue.svg?style=flat-square)](https://www.ncirl.ie/)
[![Methodology](https://img.shields.io/badge/Standard-PTES_Compliant-success.svg?style=flat-square)](http://www.pentest-standard.org/)
[![Target OS](https://img.shields.io/badge/Target-Windows_Server_2022-0078D4.svg?style=flat-square&logo=windows&logoColor=white)](https://www.microsoft.com/windows-server)
[![Lab Platform](https://img.shields.io/badge/Lab-Hack_The_Box-9FEF00.svg?style=flat-square&logo=hackthebox&logoColor=black)](https://www.hackthebox.com/)
[![Lead Analyst](https://img.shields.io/badge/Penetration_Tester-Priyansh_Gandhi-darkgreen.svg?style=flat-square)](https://github.com/priyanshgandhi2509)

---

## 📋 Assessment Overview

This repository contains the complete technical documentation, vulnerability analysis, exploit methodology, and automated report generators for an authorized **Technical Security Assessment** conducted against **Touch**, a dedicated enterprise passenger check-in kiosk and management node operating within the simulated **HTB Airways** aviation infrastructure.

* **Module**: Network Security and Penetration Testing — Continuous Assessment 1 (CA1)
* **Academic Level**: MSc in Cybersecurity (NFQ Level 9) — School of Computing, National College of Ireland
* **Lead Penetration Tester**: **Priyansh Gandhi** (Lead Analyst for Target: Touch)
* **Team Project Allocation**: Priyansh (Touch - Windows), Pratyush (Layover), Vasu (Stream), Vinay (DutyBound)
* **Assessment Date**: November 2026

---

## 🎯 Target Specification & Scope

| Parameter | Details |
| :--- | :--- |
| **Target Hostname** | `touch.htb` |
| **Target IP** | `10.129.148.253` (Hack The Box Lab Gateway) |
| **Operating System** | Microsoft Windows Server 2022 Datacenter (x64) |
| **Business Function** | Self-Service Passenger Check-In Kiosk & Nexion DeviceHub Management Node |
| **Testing Scope** | Network Services, Web Management API, Kiosk Sandbox Confinement, Windows DACLs & Services |
| **Testing Mode** | Black-box reconnaissance transitioning to Grey-box post-foothold |

---

## 🛠️ Testing Environment & Tools Matrix

| Tool | Version / Platform | Category | Purpose |
| :--- | :--- | :--- | :--- |
| **Nmap** | 7.94 / Kali Linux | Reconnaissance | TCP SYN scanning, service banner identification, OS fingerprinting |
| **cURL / jq** | 8.5.0 / Linux | API Enumeration | Interrogating unauthenticated REST API endpoints & JSON parsing |
| **FreeRDP (`xfreerdp`)** | 2.11.2 / Linux | Remote Desktop Client | Establishing authenticated graphical RDP session to kiosk GUI |
| **Microsoft Edge** | 124+ / Windows | Sandbox Escape | Exploiting unconstrained `file:///` URI handler from error modal |
| **CertUtil.exe** | Native Windows | LOLBAS Staging | Subnet file transfer and post-exploitation artifact staging |
| **MySQL Client** | 8.0.42 / Windows | Database Auditing | Interacting with backend database running under `NT AUTHORITY\SYSTEM` |
| **ICACLS.exe** | Native Windows | ACL Auditing | Reviewing Discretionary Access Control Lists on service directories |

---

## 📊 Summary of Vulnerabilities (CVSS v3.1 & MITRE ATT&CK)

| Finding ID | Vulnerability Title | CWE Mapping | CVSS v3.1 | Severity | MITRE ATT&CK |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **VULN-TOUCH-01** | Unauthenticated Information Disclosure via API | CWE-200 / CWE-306 | **7.5** | High | T1082 / T1087 |
| **VULN-TOUCH-02** | Plaintext Support Credential Exposure in Web DOM | CWE-256 / CWE-312 | **8.1** | High | T1552.001 |
| **VULN-TOUCH-03** | Kiosk Sandbox Escape via Protocol Handlers | CWE-284 / CWE-250 | **7.8** | High | T1218 / T1202 |
| **VULN-TOUCH-04** | Hardcoded Database Root Credentials in Script | CWE-798 / CWE-312 | **7.5** | High | T1552.003 |
| **VULN-TOUCH-05** | Insecure Plugin Directory DACLs & Service Vector | CWE-732 / CWE-276 | **8.8** | High | T1574.002 |

---

## ⚡ Attack Progression & Exploitation Chain

```
Phase 1: Active Reconnaissance
   │ Nmap SYN Scan identifies exposed services:
   │ 135/tcp (MSRPC), 3389/tcp (RDP), 5985/tcp (WinRM), 8443/tcp (DeviceHub HTTPS)
   ▼
Phase 2: Unauthenticated Information Disclosure (VULN-TOUCH-01)
   │ Querying https://touch.htb:8443/api/status returns JSON status object:
   │ { "device_id": "NX-DH-2024-B7042", "firmware": "v2.4.12-enterprise", ... }
   │ Serial 'NX-DH-2024-B7042' acts as pre-shared device authentication secret.
   ▼
Phase 3: DOM Credential Harvesting (VULN-TOUCH-02)
   │ Authenticating into DeviceHub management UI and inspecting DOM tree reveals
   │ cleartext kiosk maintenance credentials:
   │ Username: KioskUser | Password: K!0sk2026#
   ▼
Phase 4: Remote Graphical Session & Sandbox Confinement
   │ Authenticating via FreeRDP into full-screen check-in kiosk interface:
   │ xfreerdp /v:10.129.148.253 /u:KioskUser /p:'K!0sk2026#' /cert:ignore
   │ OS desktop, Win-key shortcuts, and Task Manager are disabled.
   ▼
Phase 5: Hardware Fault Induction & Protocol Handler Breakout (VULN-TOUCH-03)
   │ 1. In check-in kiosk, initiated passenger check-in for booking 'KS7X2M' (Crawford).
   │ 2. Simultaneously powered down passport scanner in DeviceHub admin portal.
   │ 3. Clicking 'TRY AGAIN' on kiosk triggers unhandled exception modal: SCN-ERR-4092.
   │ 4. Clicking hyperlinked help documentation invokes Microsoft Edge outside kiosk shell.
   │ 5. Leveraged file:/// URI protocol handler: file:///C:/Windows/System32/cmd.exe
   │ 6. Initial command shell executed as: touch\KioskUser
   ▼
Phase 6: Privilege Escalation Reconnaissance (VULN-TOUCH-04)
   │ Inspected C:\ProgramData\HTB Airways\refresh-dates.bat:
   │ mysql.exe -u root -p"HTB@irw4ys_DB!2026" -e "CALL UpdateSchedules();"
   │ Uncovered unencrypted database root administrative password.
   ▼
Phase 7: SYSTEM Service Hijacking (VULN-TOUCH-05)
   │ Audited service context: mysqld.exe executes as NT AUTHORITY\SYSTEM.
   │ Audited plugin directory DACLs via ICACLS:
   │ C:\MySQL\lib\plugin -> NT AUTHORITY\Authenticated Users:(M) [Modify Access]
   │ Validated User-Defined Function (UDF) DLL injection pathway for complete SYSTEM compromise.
   ▼
Phase 8: Compromise Verification
   │ Exfiltrated user.txt flag from C:\Users\KioskUser\Desktop\user.txt.
   │ Verified on Hack The Box dashboard.
```

---

## 🔬 Key Evidence & Reproduction

### 1. Unauthenticated API Status Leak
```bash
curl -k -s https://touch.htb:8443/api/status | jq .
```
```json
{
  "device_id": "NX-DH-2024-B7042",
  "firmware": "v2.4.12-enterprise",
  "status": "online",
  "peripherals": {
    "boarding_pass_printer": "ready",
    "passport_scanner_sr4200": "ready",
    "baggage_tag_printer": "offline"
  }
}
```

### 2. Kiosk RDP Connection
```bash
xfreerdp /v:10.129.148.253 /u:KioskUser /p:'K!0sk2026#' /cert:ignore +clipboard /w:1920 /h:1080
```

### 3. File System DACL Verification
```cmd
icacls "C:\MySQL\lib\plugin"
```
```
C:\MySQL\lib\plugin NT AUTHORITY\Authenticated Users:(M)
                    NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
                    BUILTIN\Administrators:(I)(OI)(CI)(F)
```
*Vulnerability Confirmed*: Standard non-admin users have `Modify` rights to inject custom dynamic-link libraries into the plugin path.

---

## 🛡️ Strategic Enterprise Remediation

1. **API Authentication & Access Control**: Implement mandatory JWT or mTLS authentication on all DeviceHub endpoints; decommission unauthenticated telemetry disclosures.
2. **Secure Credential Lifecycle**: Eliminate hardcoded credentials in scripts and web DOMs; migrate database authentication to Windows Integrated Authentication (gMSA) or Azure Key Vault.
3. **Hardened Kiosk Shell (Assigned Access)**: Migrate from basic browser shell confinement to Windows 10/11 Assigned Access (Multi-App Kiosk Mode) with AppLocker/WDAC application whitelisting.
4. **Filesystem DACL Rectification**: Revoke `Modify (M)` and `Write (W)` permissions from `Authenticated Users` across all application binary and plugin paths:
   ```cmd
   icacls "C:\MySQL\lib\plugin" /remove:g "NT AUTHORITY\Authenticated Users"
   ```
5. **Least-Privilege Service Accounts**: Reconfigure the MySQL database service to execute under a dedicated virtual service account (`NT SERVICE\MySQL`) rather than `NT AUTHORITY\SYSTEM`.

---

## 📄 Automated Report & Presentation Tooling

This assessment features automated Python document generators built with `python-docx` and `python-pptx`:

* `generate_touch_report.py`: Builds the full technical 640+ line penetration testing report.
* `generate_5page_report.py`: Compiles an executive 5-page board-ready summary document.
* `generate_3slides.py`: Generates the 3-slide widescreen executive defense presentation.
* `build_helpers.py`: Low-level OpenXML styling helpers (table formatting, custom callout cards, code blocks).

### Run the Generators:
```bash
python generate_touch_report.py
python generate_5page_report.py
python generate_3slides.py
```

---

## 👨‍💻 Penetration Tester Profile

* **Analyst**: Priyansh Gandhi
* **Institution**: National College of Ireland (NCI)
* **Degree**: MSc in Cybersecurity (NFQ Level 9)
* **GitHub**: [@priyanshgandhi2509](https://github.com/priyanshgandhi2509)
