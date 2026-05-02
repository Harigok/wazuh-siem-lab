# 🛡️ Wazuh SIEM & XDR Home Lab

> **Platform:** Wazuh v4.7.5 | **Environment:** Oracle VirtualBox | **OS:** Ubuntu Server (Manager) + Windows 11 (Agent) | **Date:** April 27, 2026

A complete walkthrough of deploying and configuring the Wazuh open-source SIEM & XDR platform from scratch — including every command used, every mistake made, and how each was fixed.

---

## 📋 Table of Contents

1. [Pre-Setup — GitHub Account Creation](#1-pre-setup--github-account-creation)
2. [VirtualBox Environment](#2-virtualbox-environment)
3. [Wazuh Installation on Ubuntu Server](#3-wazuh-installation-on-ubuntu-server)
4. [Finding the Server IP & Accessing the Dashboard](#4-finding-the-server-ip--accessing-the-dashboard)
5. [Dashboard Login & Password Recovery](#5-dashboard-login--password-recovery)
6. [Windows Agent Enrollment (kakashi)](#6-windows-agent-enrollment-kakashi)
7. [Agent Connected & Security Events Live](#7-agent-connected--security-events-live)

---

## 1. Pre-Setup — GitHub Account Creation

Before starting the lab, a GitHub account was created to document the project.

![GitHub account verification screen](Screenshot%202026-04-27%20184806.png)

| ✅ Command / Action | ❌ Mistake | 🔧 Fix |
|---|---|---|
| Navigated to github.com to create account | — | — |
| Completed phone/QR verification step | "Waiting for verification" loop on screen | Scanned QR code again with camera app and completed steps on phone |

---

## 2. VirtualBox Environment

Oracle VirtualBox 7.2 was used to host the virtual machines. The lab uses a linked clone of an Ubuntu Server base image alongside a Kali Linux VM.

![VirtualBox Manager showing ubuntu VM selected and VBox.log open](Screenshot%202026-04-27%20213125.png)

| ✅ Command / Action | ❌ Mistake | 🔧 Fix |
|---|---|---|
| Opened Oracle VirtualBox Manager | `ubuntu server` and `ubuntu` VMs both showed "Powered Off" initially | Started the `ubuntu` VM (the active Wazuh host) |
| Reviewed VBox.log to confirm VM hardware state | — | — |
| Host specs noted: HP Victus Gaming Laptop, 16GB RAM, Windows 11, Secure Boot ENABLED | Secure Boot enabled — potential risk for unsigned kernel modules | Did not block Wazuh install; noted for future reference |

---

## 3. Wazuh Installation on Ubuntu Server

Wazuh was installed using the official assisted install script on the Ubuntu VM inside VirtualBox.

![Terminal: sudo apt install update error, then sudo apt update running successfully with wazuh-install files visible](Screenshot%202026-04-27%20211441.png)

![Terminal: Wazuh install log showing indexer, manager, Filebeat, dashboard all starting successfully — credentials shown at end](Screenshot%202026-04-27%20212619.png)

![Same install log — wider terminal view confirming full summary with admin password](Screenshot%202026-04-27%20212622.png)

| ✅ Command Used | ❌ Mistake | 🔧 Fix |
|---|---|---|
| `sudo apt install update` | ❌ **Wrong command** — `update` is not a package name, `install` takes a package | Corrected to `sudo apt update` |
| `sudo apt update` | Warnings: `univese` component misspelt in sources.list | Warnings only — did not block install |
| `ls` | Confirmed `wazuh-install-files.tar` and `wazuh-install.sh` present | — |
| `bash wazuh-install.sh` | — | — |

**Install completed at 15:52:25. Auto-generated credentials:**
```
User:     admin
Password: azo5Uf*0*ASb22T9SggVb4L+qSV5I9WY
Dashboard URL: https://<wazuh-dashboard-ip>:443
```

---

## 4. Finding the Server IP & Accessing the Dashboard

After install, the server IP was needed to open the Wazuh dashboard in a browser.

![Terminal: ip a output showing IP 10.72.74.13, and Bing search open in browser for that IP](Screenshot%202026-04-27%20213432.png)

![Wazuh login page at https://10.72.74.13 — first attempt showing "Invalid username or password" error](Screenshot%202026-04-27%20213802.png)

| ✅ Command Used | ❌ Mistake | 🔧 Fix |
|---|---|---|
| `ip a` | — | Got correct IP: `10.72.74.13` |
| `http://10.72.74.15` (typed in terminal) | ❌ **Two mistakes**: wrong IP (`10.72.74.15` vs `.13`) AND shell can't open URLs | Used browser, typed correct IP `https://10.72.74.13` |
| Searched `http://10.72.74.13` in Bing | ❌ **Mistake** — private LAN IPs don't resolve on the internet | Typed the IP directly in the browser address bar |
| Opened `https://10.72.74.13/app/login` | First login attempt: "Invalid username or password" | Needed to retrieve and reset password from password file |

---

## 5. Dashboard Login & Password Recovery

The auto-generated admin password was rejected. Multiple commands were tried to retrieve and reset it.

![Terminal: cat wazuh-install-files/wazuh-passwords.txt — Permission denied, then sudo cat succeeds showing all service credentials](Screenshot%202026-04-27%20214332.png)

![Same credentials file — full terminal view with all indexer and API passwords](Screenshot%202026-04-27%20214341.png)

![Wazuh login — still showing "Invalid username or password" after pasting password](Screenshot%202026-04-27%20214504.png)

![Terminal: failed wazuh-password-tools.sh (wrong name), then Qwerty@2005 rejected for complexity, then Qwerty*2005 accepted](Screenshot%202026-04-27%20214907.png)

![Terminal: password reset success message, ip a, ufw firewall rules added for 1514 and 1515, then systemctl restart wazuh-manger typo error](Screenshot%202026-04-27%20215905.png)

| ✅ Command Used | ❌ Mistake | 🔧 Fix |
|---|---|---|
| `cat wazuh-install-files/wazuh-passwords.txt` | ❌ **Permission denied** — file is root-owned | `sudo cat wazuh-install-files/wazuh-passwords.txt` |
| `sudo cat wazuh-install-files/wazuh-passwords.txt` | ✅ Showed all credentials | — |
| Copy-pasted `azo5Uf*0*ASb22T9SggVb4L+qSV5I9WY` into dashboard login | ❌ **Still "Invalid username or password"** — special characters likely mangled in copy-paste | Decided to reset the password using the password tool |
| `sudo tar -xvf wazuh-install-files.tar wazuh-install-files/wazuh-password.txt` | ❌ **"Not found in archive"** — wrong filename (singular `password` vs plural `passwords`) | Ran twice, both failed |
| `sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-password-tools.sh --changeall` | ❌ **Command not found** — wrong script name (`wazuh-password-tools.sh`) | Corrected to `wazuh-passwords-tools.sh` |
| `sudo .../wazuh-passwords-tools.sh -u admin -p Qwerty@2005` | ❌ **Password rejected** — complexity rule: must contain a symbol from `.+?-`; `@` not accepted | Tried next password |
| `sudo .../wazuh-passwords-tools.sh -u admin -p Qwertyx2005` | ❌ **Still rejected** — no special symbol at all | Changed to use `*` |
| `sudo .../wazuh-passwords-tools.sh -u admin -p Qwerty*2005` | ✅ **"Password changed."** — INFO: Generating password hash. WARNING: Remember to update password in Wazuh dashboard and Filebeat nodes. | — |
| `sudo ufw allow 1515/tcp` | — | Rules updated (v4 + v6) |
| `sudo ufw allow 1514/tcp` | — | Rules updated (v4 + v6) |
| `sudo systemctl restart wazuh-manger` | ❌ **Typo** — `wazuh-manger` → "Unit not found" | Corrected: `sudo systemctl restart wazuh-manager` |

---

## 6. Windows Agent Enrollment (kakashi)

After successfully logging into the Wazuh dashboard, a Windows 11 agent named **kakashi** was enrolled from the host machine.

![Wazuh dashboard Deploy Agent page — package selection showing Linux (RPM/DEB), Windows MSI, macOS options](Screenshot%202026-04-27%20215123.png)

![PowerShell (non-admin): Invoke-WebRequest command run twice — both fail with "Access to path C:\wazuh-agent is denied"](Screenshot%202026-04-27%20215424.png)

![Split screen: Admin PowerShell on left running commands, ChatGPT fix on right saying close PowerShell and reopen as Administrator](Screenshot%202026-04-27%20215615.png)

![Admin PowerShell: Invoke-WebRequest succeeds silently, then NET STARTwazuhSvc typo shows NET syntax help, then NET START WazuhSvc — "The Wazuh service was started successfully"](Screenshot%202026-04-27%20215621.png)

![Admin PowerShell: Get-Service wazuhsvc shows Status Running, Test-NetConnection to 10.72.74.13 port 1514 shows TcpTestSucceeded: True](Screenshot%202026-04-27%20215728.png)

![Wazuh Agents dashboard — agent kakashi ID 001 listed, status grey dot (pending) — not yet active](Screenshot%202026-04-27%20215609.png)

| ✅ Command Used | ❌ Mistake | 🔧 Fix |
|---|---|---|
| Dashboard → Deploy new agent → Windows MSI 32/64-bit | — | — |
| `Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.5-1.msi -OutFile ${env:tmp}\wazuh-agent; msiexec.exe /i ${env:tmp}\wazuh-agent /q WAZUH_MANAGER='10.72.74.13' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='kakashi' WAZUH_REGISTRATION_SERVER='10.72.74.13'` (in regular PowerShell) | ❌ **"Access to the path 'C:\wazuh-agent' is denied"** — PowerShell was not elevated | Closed PowerShell, reopened as Administrator (right-click → Run as Administrator) |
| Same `Invoke-WebRequest` command in Admin PowerShell | ✅ Succeeded silently — agent MSI downloaded and installed | — |
| `NET STARTwazuhSvc` | ❌ **Typo** — no space before `wazuhSvc` → showed full NET syntax help | Corrected to `NET START WazuhSvc` |
| `NET START WazuhSvc` | ✅ "The Wazuh service is starting. The Wazuh service was started successfully." | — |
| `Get-Service wazuhsvc` | ✅ Status: **Running**, DisplayName: Wazuh | — |
| `Test-NetConnection 10.72.74.13 -Port 1514` | ✅ **TcpTestSucceeded: True** — agent talking to manager on port 1514 | — |

---

## 7. Agent Connected & Security Events Live

After confirming the service and network connectivity, the agent appeared in the Wazuh dashboard as fully **Active** with live security events flowing in.

![Wazuh Agents page — kakashi ID 001, Windows 11 Home, IP 10.72.74.240, v4.7.5, node01, status green dot Active, 100% agents coverage](Screenshot%202026-04-27%20220047.png)

![Wazuh Security Events for kakashi — donut charts for Top 5 alerts (Logon failure, CIS benchmarks), Top 5 rule groups (sca, ossec, windows_security), PCI DSS requirements — plus live alerts table showing MITRE T1078 and T1531 techniques](Screenshot%202026-04-27%20220311.png)

**Agent summary:**

| Field | Value |
|---|---|
| Agent ID | 001 |
| Agent Name | kakashi |
| OS | Microsoft Windows 11 Home Single Language 10.0.26200.8246 |
| IP Address | 10.72.74.240 |
| Wazuh Version | v4.7.5 |
| Cluster Node | node01 |
| Status | 🟢 **Active** |
| Agents Coverage | 100% |

**Live security alerts detected immediately after agent connect:**

| Time | MITRE Technique | Tactic | Description | Level | Rule ID |
|---|---|---|---|---|---|
| 22:02:10 | T1078, T1531 | Defense Evasion, Persistence, Privilege Escalation, Initial Access, Impact | Logon failure - Unknown user or bad password | 5 | 60122 |
| 22:02:07 | T1078, T1531 | Defense Evasion, Persistence, Privilege Escalation | Logon failure - Unknown user or bad password | 5 | 60122 |
| 22:02:03 | T1078, T1531 | Defense Evasion, Persistence, Privilege Escalation | Logon failure - Unknown user or bad password | 5 | 60122 |
| 22:02:03 | T1078 | Defense Evasion, Persistence, Initial Access | Windows logon success | 3 | 60106 |
| 21:59:45 | — | — | Wazuh agent started | 3 | 503 |

**Top rule groups active:** `sca` · `ossec` · `windows` · `windows_security` · `authentication_failed`

---

## 🔑 Key Lessons & Mistakes Summary

| # | Area | Mistake Made | Correct Approach |
|---|---|---|---|
| 1 | Linux apt | `sudo apt install update` | `sudo apt update` — update is not a package |
| 2 | File permissions | `cat` on root-owned file → Permission denied | Always use `sudo cat` for protected files |
| 3 | File naming | `wazuh-password.txt` (singular) | Correct filename is `wazuh-passwords.txt` (plural) |
| 4 | Script naming | `wazuh-password-tools.sh` | Correct name is `wazuh-passwords-tools.sh` |
| 5 | Password symbols | `Qwerty@2005` rejected — `@` not in allowed symbol set | Use `.`, `+`, `?`, `-`, or `*` as the symbol |
| 6 | Wazuh service name | `wazuh-manger` (typo) | Correct: `wazuh-manager` |
| 7 | Shell vs browser | Typed `http://10.72.74.15` into terminal | Use a browser; also had wrong IP (`.15` vs `.13`) |
| 8 | LAN IP in search | Searched private IP `10.72.74.13` in Bing | Private IPs are not resolvable externally — open directly in browser |
| 9 | PowerShell permissions | Ran agent install in regular PowerShell → Access Denied | Must right-click → Run as Administrator |
| 10 | NET command typo | `NET STARTwazuhSvc` (no space) | `NET START WazuhSvc` |

---

## 🧰 Tools Used

- **Oracle VirtualBox 7.2** — Virtualization host
- **Ubuntu Server (focal)** — Wazuh Manager
- **Windows 11 Home** — Wazuh Agent (named: kakashi)
- **Wazuh v4.7.5** — SIEM & XDR platform
- **PowerShell (Admin)** — Windows agent enrollment
- **ChatGPT** — Consulted to troubleshoot the PowerShell "Access Denied" error
- **MITRE ATT&CK Framework** — Technique mapping (T1078: Valid Accounts, T1531: Account Access Removal)
- **UFW** — Ubuntu firewall (opened ports 1514, 1515)

---

## 📚 Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Quickstart Install](https://documentation.wazuh.com/current/quickstart.html)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Wazuh GitHub](https://github.com/wazuh/wazuh)

---

## 🏷️ Tags

`cybersecurity` `siem` `wazuh` `blue-team` `soc` `xdr` `virtualbox` `ubuntu` `windows-11` `mitre-attack` `home-lab` `threat-detection` `opensearch` `powershell`
