<div align="center">

# 🛡️ Enterprise Attack Simulation & Defensive Monitoring Lab

**A full-cycle SOC simulation: Red Team attack → SIEM detection → automated Blue Team response**

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=flat-square)
![MITRE](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-red?style=flat-square)
![Kali](https://img.shields.io/badge/Attacker-Kali%20Linux-557C94?style=flat-square)
![Ubuntu](https://img.shields.io/badge/Target-Ubuntu%20Server-E95420?style=flat-square)
![License](https://img.shields.io/badge/Purpose-Lab%20%2F%20Learning-green?style=flat-square)

</div>

---

## Overview

This project documents the construction of a home enterprise security lab designed to simulate real-world attacks and demonstrate detection and incident response capabilities using the Wazuh SIEM platform. The environment replicates attack patterns found in corporate networks and showcases how security tooling can detect, correlate, and automatically respond to malicious activity.

---

## Architecture

| Role | System | Purpose |
|------|--------|---------|
| ⚔️ Attacker | Kali Linux | Offensive operations (Red Team) |
| 🖥️ Target | Ubuntu Server | SSH service exposed as attack surface |
| 📊 SIEM | Wazuh | Log collection, correlation, active response |
| 🔑 Service | OpenSSH | Monitored authentication endpoint |

---

## Phase 1 — Red Team: Brute Force Attack

A credential brute-force attack was launched against the SSH service using Hydra, simulating an external threat actor attempting to gain unauthorized access.

````bash
hydra -l root -P rockyou.txt ssh://192.168.1.106
````

**What happens:** Hydra generates hundreds of authentication attempts per minute, all of which are written to the target's authentication log at `/var/log/auth.log`.

---

## Phase 2 — Blue Team: Detection & Monitoring

The Wazuh agent on the target server continuously forwards logs to the SIEM. Custom rules were created in `local_rules.xml` to identify the attack pattern:

````xml
<!-- Rule 100200: Single SSH login failure -->
<rule id="100200" level="5">
  <if_matched_sid>5716</if_matched_sid>
  <description>SSH authentication failure detected.</description>
  <mitre><id>T1110</id></mitre>
</rule>

<!-- Rule 100201: Brute force pattern (5 failures in 60s) -->
<rule id="100201" level="10" frequency="5" timeframe="60">
  <if_matched_sid>100200</if_matched_sid>
  <description>SSH brute force attack detected.</description>
  <mitre><id>T1110</id></mitre>
</rule>
````

**MITRE ATT&CK mapping:** [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)

---

## Phase 3 — Active Response: Automated Blocking

Wazuh's Active Response module was configured to automatically block the attacker's IP upon rule 100201 triggering — no human intervention required.

| Parameter | Value |
|-----------|-------|
| Script | `firewall-drop` |
| Mechanism | `nftables` firewall rule |
| Block duration | 180 seconds |
| Attacker result | `ssh: connect to host [...] port 22: Connection timed out` |

---

## Results

The Wazuh dashboard provides centralized visibility across the full attack lifecycle:

- **Detection** — SSH brute-force attacks identified in real time
- **Correlation** — Events mapped to MITRE ATT&CK technique T1110
- **Response** — Automatic IP blocking via nftables within seconds of detection
- **Visibility** — Full audit trail available in the SIEM dashboard

---

## Tech Stack

- [Wazuh](https://wazuh.com/) — open-source SIEM & XDR
- [Hydra](https://github.com/vanhauser-thc/thc-hydra) — network login brute-forcer
- [MITRE ATT&CK](https://attack.mitre.org/) — adversary tactics & techniques framework
- `nftables` — Linux kernel firewall used for IP blocking
- OpenSSH — target service

---

## Disclaimer

> This lab was built in an isolated private network for educational purposes only. All attacks were conducted against systems I own and control. Do not use these techniques against systems without explicit authorization.
