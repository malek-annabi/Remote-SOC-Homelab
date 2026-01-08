# Remote SOC Homelab – Architecture & Security Documentation

## Executive Summary

This project documents the design and operation of a personal **Remote Security Operations Center (SOC) homelab**, built to realistically simulate enterprise blue‑team operations while remaining fully compliant with corporate security policies.

The environment is hosted in a **home lab** and securely accessed from a **corporate office network** using a hardened, split‑tunnel VPN architecture. Particular attention was paid to **risk management**, **traffic isolation**, and **policy alignment**, ensuring that no corporate assets, credentials, or internet traffic are exposed or routed through the lab.

The SOC stack includes industry‑standard tooling such as **Wazuh (SIEM/XDR)**, **Graylog (log management)**, **MISP (threat intelligence)**, and **DFIR‑IRIS (incident response)**, deployed across multiple Linux and Windows endpoints to mirror real production diversity.

Red‑team activities involving live malware were **intentionally paused** following expert consultation, due to the risk of accidental compromise or lateral movement on the corporate network. The project therefore focuses on **blue‑team and purple‑team activities**, including detection engineering, log correlation, threat intelligence ingestion, and secure remote analyst access.

This homelab demonstrates not only technical implementation skills, but also **professional security judgment**, showcasing how to balance realism, learning objectives, and enterprise risk constraints.

---

## 1. Purpose & Context
This document describes the design, deployment, and security decisions behind a personal SOC homelab accessed remotely from a corporate network.

**Primary objectives:**
- Enable secure remote access to a home SOC lab
- Avoid policy violations or security risks on a corporate network
- Simulate real-world SOC operations (blue / purple)
- Maintain strong isolation between environments

---

## 2. High-Level Architecture

### 2.1 Physical Locations
- **Home environment**: SOC infrastructure hosted on a desktop hypervisor
- **Office environment**: Corporate network with FortiGate (IPS, AV, Web Filtering, SSL inspection)

Remote access is achieved via a hardened, split-tunnel VPN.

---

## 3. Hardware Overview

### 3.1 Home Desktop (SOC Host)
- CPU: Intel i7-7700
- RAM: 32 GB
- GPU: NVIDIA GTX 1080 Ti
- Storage:
  - NVMe 230 GB (Windows 10 – host OS)
  - SATA SSD 500 GB
  - HDD 1 TB
- Hypervisor: VMware Workstation Pro

### 3.2 Laptop (Remote Analyst Workstation)
- CPU: Intel i7-8750H
- RAM: 32 GB
- GPU: NVIDIA GTX 1050 Ti Mobile
- Storage:
  - NVMe 230 GB
  - HDD 1 TB
- OS:
  - Windows 11 (primary)
  - Kali Linux (dual-boot)

---

## 4. Virtual Infrastructure (Home SOC)

### 4.1 Virtual Machines

| VM | OS | Resources | Role |
|----|----|----------|------|
| MISP | Ubuntu Server 22.04 | 2 CPU / 4 GB | Threat Intelligence Platform (Docker) |
| Wazuh Manager | Ubuntu Server 24.04 | 4 CPU / 8 GB | SIEM / XDR (All-in-One) |
| Windows Server | Windows Server 2025 | 2 CPU / 4 GB | Endpoint with Wazuh Agent |
| Linux Endpoint | Ubuntu Server 24.04 | 2 CPU / 2 GB | Test endpoint with Wazuh Agent |
| DFIR-IRIS | Ubuntu Server 24.04 | 2 CPU / 4 GB | Incident Response Platform |
| Graylog | Ubuntu Server 20.04 | 2 CPU / 4 GB | Centralized Log Management |
| WireGuard | Ubuntu Server 24.04 | 2 CPU / 2 GB | Secure Remote Access Gateway |
| **Pi-hole** | **Ubuntu Server 22.04** | **1 CPU / 1–2 GB** | **Private DNS, Ad & Malicious Domain Blocking** |

----|----|----------|------|
| MISP | Ubuntu Server 22.04 | 2 CPU / 4 GB | Threat Intelligence Platform (Docker) |
| Wazuh Manager | Ubuntu Server 24.04 | 4 CPU / 8 GB | SIEM / XDR (All-in-One) |
| Windows Server | Windows Server 2025 | 2 CPU / 4 GB | Endpoint with Wazuh Agent |
| Linux Endpoint | Ubuntu Server 24.04 | 2 CPU / 2 GB | Test endpoint with Wazuh Agent |
| DFIR-IRIS | Ubuntu Server 24.04 | 2 CPU / 4 GB | Incident Response Platform |
| Graylog | Ubuntu Server 20.04 | 2 CPU / 4 GB | Centralized Log Management |
| WireGuard | Ubuntu Server 24.04 | 2 CPU / 2 GB | Secure Remote Access Gateway |

---

## 5. Network Design

### 5.1 Internal LAN
- Subnet: **10.10.0.0/24**
- All SOC VMs reside on this network
- **Pi-hole DNS IP**: `10.10.0.53`

### 5.2 VPN Network
- Technology: **WireGuard**
- VPN Subnet: **10.200.0.0/24**
- Server IP: **10.200.0.1**
- Client IP (Laptop): **10.200.0.10**

### 5.3 DNS Flow

- SOC VMs use **Pi-hole** as primary DNS
- Pi-hole forwards queries to trusted upstream resolvers (Cloudflare / Quad9)
- Remote laptop may optionally use Pi-hole DNS **only when connected via VPN**

---

## 6. VPN Design & Security Decisions

### 6.1 Why WireGuard
- Minimal attack surface
- UDP-based, low-noise traffic
- No certificate-based TLS inspection
- Resistant to false positives on corporate IDS/IPS

### 6.2 Split Tunnel Strategy

Client configuration restricts routing to:
- **10.10.0.0/24 only**

This ensures:
- No corporate internet traffic over VPN
- No policy violations
- Reduced detection footprint

### 6.3 NAT & Forwarding
- Source NAT for VPN clients
- Explicit forwarding rules
- No inbound access from LAN to VPN clients

---

## 7. Policy & Risk Management

### 7.1 Red Team Activities (Paused)

Red team activities involving live malware are currently **on hold** due to:
- Corporate network exposure
- Risk of accidental lateral movement
- Human error considerations

Decision taken after consultation with internal security experts.

### 7.2 Approved Activities
- Detection engineering
- Log collection and correlation
- Threat intelligence ingestion
- Purple-team simulations without malware

---

## 8. Blue Team Focus Areas

- Wazuh rule tuning
- Sysmon and auditd baselining
- Graylog parsing pipelines
- MISP feed curation
- Alert quality and false positive reduction
- **DNS-based detection and filtering via Pi-hole**

---

## 8.1 DNS Security (Pi-hole)

Pi-hole is deployed as a dedicated DNS security layer within the SOC LAN.

**Functions:**
- Ad and tracker blocking
- Blocking known malicious and phishing domains
- DNS-level visibility for SOC endpoints

**Security considerations:**
- Lab-only DNS usage (no corporate DNS interception)
- Static IP assignment
- Moderate, high-quality blocklists to avoid false positives

---

- Wazuh rule tuning
- Sysmon and auditd baselining
- Graylog parsing pipelines
- MISP feed curation
- Alert quality and false positive reduction

---

## 9. Operational Best Practices

- No malware execution on corporate-connected devices
- VPN client installed on host OS only
- No VPN inside attacker or C2 VMs
- Regular snapshots before major changes

---

## 10. Future Enhancements (Approved Scope)

- WireGuard key rotation
- Wazuh integration for VPN events
- SOC dashboards (Graylog / Grafana)
- Documented purple-team playbooks

---

## 11. Conclusion

This homelab demonstrates a realistic, enterprise‑aligned SOC environment with strong emphasis on security, policy compliance, and professional risk management.

---

## Appendix A – GitHub README Version

# Remote SOC Homelab

## Overview
This repository documents a personal **Remote SOC Homelab** designed to simulate real-world blue-team operations while maintaining strict separation from corporate networks.

The lab is accessed remotely via a **split-tunnel WireGuard VPN**, allowing secure analyst access without routing corporate internet traffic or violating organizational security policies.

## Architecture Diagram (High-Level)

```
        [ Corporate Network ]
                 |
           (FortiGate Firewall)
                 |
           [ Analyst Laptop ]
           (WireGuard Client)
                 |
        ===== Encrypted VPN =====
                 |
         [ WireGuard Server ] 10.200.0.1
                 |
        -------------------------
        |     Home SOC LAN      |
        |      10.10.0.0/24     |
        |                       |
        |  [ Pi-hole ] 10.10.0.53
        |      |                |
        |  DNS Filtering        |
        |      |                |
        |  [ Wazuh ]  [ Graylog ]
        |     |          |      |
        |  [ Endpoints / IR / MISP ]
        -------------------------
```

## Key Objectives
- Secure remote SOC access
- Enterprise-realistic monitoring and detection
- Policy-compliant lab operation
- Blue / Purple team focus

## SOC Stack
- **Wazuh** – SIEM / XDR
- **Graylog** – Centralized logging
- **MISP** – Threat intelligence platform
- **DFIR-IRIS** – Incident response management
- **Pi-hole** – Private DNS & malicious domain blocking

## Security Principles
- No full-tunnel VPN
- No malware execution on corporate-connected devices
- VPN client installed on host OS only
- Lab-only DNS via Pi-hole
- Red-team malware activity paused pending authorization

## Disclaimer
This project is for educational and defensive security purposes only. All activities are conducted on personally owned infrastructure with strict adherence to organizational security policies.

