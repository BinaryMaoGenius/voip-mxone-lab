<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,20,25&height=200&section=header&text=VoIP%20MX-ONE%20Lab&fontSize=48&fontColor=ffffff&animation=fadeIn&fontAlignY=40&desc=Mitel%20MX-ONE%20IPBX%20Deployment%20%E2%80%94%20SIP%20%7C%20Virtualization%20%7C%20Troubleshooting&descAlignY=60&descSize=14&descColor=cccccc" width="100%" />
</div>

<div align="center">
  <img src="https://img.shields.io/badge/Mitel_MX--ONE-IPBX-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white" />
  <img src="https://img.shields.io/badge/VMware_Workstation-Pro-607078?style=for-the-badge&logo=vmware&logoColor=white" />
  <img src="https://img.shields.io/badge/Protocol-SIP_UDP%2F5060-7C3AED?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Linux-SUSE_Enterprise-73BA25?style=for-the-badge&logo=suse&logoColor=white" />
  <img src="https://img.shields.io/badge/Status-Completed-4ADE80?style=for-the-badge" />
</div>

<br/>

---

## Overview

<nowiki>Full deployment of a professional enterprise VoIP infrastructure using **Mitel MiVoice MX-ONE Service Node**, carried out as a 3-lab academic mini-project at UIPE (Université Internationale Privée d'Excellence), Bamako — academic year 2025-2026.</nowiki>

The project covers the complete lifecycle of an enterprise telephony system: virtualization, configuration, and fault diagnosis.

> 📄 **Full technical report:** [`Rapport_Technique_VoIP_MX-ONE_KONE_Emmanuel.pdf`](./Rapport_Technique_VoIP_MX-ONE_KONE_Emmanuel.pdf)
>
> 🎬 **10-minute demonstration video:** [Download from GitHub Releases](../../releases/latest)

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              VMware Workstation Pro                 │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │     SUSE Linux Enterprise Server            │  │
│  │                                              │  │
│  │   ┌──────────────────────────────────┐      │  │
│  │   │   Mitel MiVoice MX-ONE           │      │  │
│  │   │   Service Node Manager (SNM)     │      │  │
│  │   │   https://192.168.101.10         │      │  │
│  │   └──────────────────────────────────┘      │  │
│  │                                              │  │
│  │   eth0: 192.168.101.10/24 (management)      │  │
│  │   eth1: telephony interface                 │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  VMnet0 (Bridged) ──── VMnet8 (NAT)                │
└─────────────────────────────────────────────────────┘
         │
         ▼
  MicroSIP Softphone
  Extension 5001-5003
  SIP UDP/5060 | RTP 10000-20000
```

---

## Lab Summary

| Lab | Objective | Duration | Status |
|-----|-----------|----------|--------|
| **Lab 1** | MX-ONE Installation & Virtualization | 4h | ✅ Completed |
| **Lab 2** | SIP Extensions + SIP Trunk Configuration | 6h | ✅ Completed |
| **Lab 3** | VoIP Fault Diagnosis & Troubleshooting | 4h | ✅ Completed |

---

## Lab 1 — Installation & Virtualization

**Objective:** Deploy the MX-ONE virtual appliance on VMware Workstation and achieve network connectivity.

**Steps performed:**
- Installed VMware Workstation Pro and configured two virtual networks: `VMnet0` (Bridged) and `VMnet8` (NAT)
- Imported the MX-ONE OVA appliance (4GB RAM, 2 vCPU, dual NIC: eth0 management + eth1 telephony)
- Configured network via the MX-ONE initialization script — `eth0` assigned `192.168.101.10/24`
- Validated connectivity: `ping 192.168.101.1` and `ping 8.8.8.8`
- Accessed the **Service Node Manager (SNM)** web interface at `https://192.168.101.10`

**Success criteria:**

| Criteria | Result |
|----------|--------|
| VM started without errors | ✅ |
| Network connectivity established | ✅ |
| SNM accessible via HTTPS | ✅ |

---

## Lab 2 — Advanced Configuration

**Objective:** Create SIP extensions, configure the softphone, validate internal calls, and set up a simulated SIP Trunk.

### SIP Extensions

4 users created in the **MX-ONE Provisioning Manager** under department `LAB_UIE`:

| User ID | Last Name | First Name | Extension | Department |
|---------|-----------|------------|-----------|------------|
| Ass | Diarra | Assanatou | 5001 | LAB_UIE |
| Adja | Sidibe | Adja | 5002 | LAB_UIE |
| Dri | Diaou | Drissa | 5003 | LAB_UIE |
| bmg | Kone | Emmanuel | — | LAB_UIE |

### Softphone Configuration (MicroSIP)

```
SIP Server (Domain) : 192.168.101.10
Username            : 5001
Authentication ID   : 5001
Transport           : UDP
SIP Port            : 5060
Status              : Registered / Online
```

### SIP Trunk (Simulated)

- Number plan configured: prefix `90` → external destination
- Route `SIP_Trunk_Sim` created in `Telephony > External Lines > Route`

```
Route Name          : SIP_Trunk_Sim
Signaling Type      : SIP
Outgoing Protocol   : UDP
Remote Port         : 5060
Remote Proxy IP     : 192.168.101.254
Accepted Call Type  : Remote IP
Incoming Match IP   : 192.168.101.254
Invite URI          : sip:+?@192.168.101.254
```

**Success criteria:**

| Criteria | Result |
|----------|--------|
| Extensions created and configured | ✅ |
| Different profiles assigned | ✅ |
| Softphone registered and operational | ✅ |
| Internal calls successful (5001↔5002↔5003) | ✅ |
| SIP Route configured | ✅ |
| External numbering plan (prefix 90) | ✅ |

---

## Lab 3 — Troubleshooting

**Objective:** Diagnose and resolve 3 common VoIP failure scenarios using MX-ONE diagnostic tools.

### Scenario 1 — Unregistered Extension

**Symptom:** Softphone shows `Registration Failed` — no calls possible.

**Diagnostic steps:**
```bash
ping 192.168.101.10                    # Test network connectivity
tail -f /var/log/eri_sn/sip.log        # Read SIP logs
netstat -tulnp | grep 5060             # Check firewall port
```
**Root cause & fix:** Incorrect SIP authentication parameters + UDP 5060 blocked on firewall. Corrected credentials and opened port.

---

### Scenario 2 — No Dial Tone

**Symptom:** No dial tone after going off-hook.

**Diagnostic steps:**
```bash
# Check Media Server status in SNM: Services > Media Server
tail -f /var/log/eri_sn/media.log      # Media server logs
systemctl restart eri-media            # Restart media service
systemctl status eri-media             # Verify service is running
```
**Root cause & fix:** Media Server service down. Restarted via `systemctl`, DSP resources confirmed available.

---

### Scenario 3 — One-Way or No Audio

**Symptom:** Call established (SIP signaling works) but no audio in one or both directions.

**Diagnostic steps:**
```bash
# Verify RTP port range 10000-20000 in MX-ONE config
# Capture RTP traffic:
wireshark -i eth0 -f "udp portrange 10000-20000"
# Check negotiated codecs in SIP logs (G.711, G.729)
```
**Root cause & fix:** RTP ports 10000-20000 UDP blocked on firewall.
```bash
iptables -A INPUT -p udp --dport 10000:20000 -j ACCEPT
```
Wireshark capture confirmed RTP stream transmission after fix.

**Success criteria:**

| Criteria | Result |
|----------|--------|
| Correct root cause identification | ✅ |
| Diagnostic tools used | ✅ |
| Issues effectively resolved | ✅ |

---

## Tech Stack

| Component | Technology | Role |
|-----------|-----------|------|
| Hypervisor | VMware Workstation Pro | Virtualization |
| MX-ONE OS | SUSE Linux Enterprise Server | Host system |
| IPBX | Mitel MiVoice MX-ONE | Service Node |
| Admin Interface | Service Node Manager (SNM) | Web-based admin at 192.168.101.10 |
| Softphone | MicroSIP + Zoiper | SIP terminal for testing |
| Signaling | SIP UDP/5060 | VoIP signaling protocol |
| Media | RTP 10000–20000 UDP | Audio transport |
| Packet Analysis | Wireshark | RTP/SIP capture and diagnosis |

---

## Key Skills Demonstrated

- 🖥️ **Virtualization** — Telecom appliance deployment on VMware (OVA import, dual NIC config)
- 📞 **IPBX Administration** — Mitel MX-ONE Service Node Manager, Provisioning Manager
- 🔧 **SIP Protocol** — Extension creation, SIP Trunk configuration, numbering plan
- 🔍 **VoIP Troubleshooting** — SIP log analysis, RTP capture, firewall diagnosis
- 🐧 **Linux Systems** — SUSE Enterprise Server, `systemctl`, `iptables`, `netstat`, `tail`
- 📡 **Network Analysis** — Wireshark packet capture and protocol analysis

---

## Project Info

| Field | Value |
|-------|-------|
| Module | Introduction à la Télécommunication |
| Institution | Université Internationale Privée d'Excellence (UIPE), Bamako 🇲🇱 |
| Supervisor | M. Amadou Bagayoko |
| Year | 2025-2026 |
| Author | Emmanuel Elisé Koné (`bmg`) |

---

<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,20,25&height=100&section=footer" width="100%" />
</div>
