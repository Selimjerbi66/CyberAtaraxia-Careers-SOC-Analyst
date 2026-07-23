<div align="center">
  <img src="https://github.com/Selimjerbi66/CyberAtaraxia-Suite/blob/main/cyberataraxia_new_logo.png?raw=true" width="180" alt="CyberAtaraxia Logo"/>
  <h1>CyberAtaraxia Careers — SOC Analyst</h1>

  <p>
    A free, hands-on 2-week home lab program to go from cybersecurity beginner to job-ready SOC Analyst fundamentals — part of the <strong>CyberAtaraxia Suite</strong> by <strong>Selim JERBI</strong>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Status-In%20Development-blue?style=for-the-badge" />
    <img src="https://img.shields.io/badge/License-Open%20Source-green?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Cost-100%25%20Free%20Tools-success?style=for-the-badge" />
    <a href="https://github.com/Selimjerbi66">
      <img src="https://img.shields.io/badge/Made%20by-Selim%20JERBI-blueviolet?style=for-the-badge" />
    </a>
  </p>
  <p>
    🇬🇧 <a href="README.md">English</a> &nbsp;|&nbsp; 🇫🇷 <a href="README_FR.md">Français</a>
  </p>
</div>

---

## 🧭 About This Career Path

**CyberAtaraxia Careers** is a new line within the CyberAtaraxia Suite: guided, hands-on **career paths** rather than single tools — each one designed to take a beginner from zero to a demonstrable, portfolio-ready skillset using only free and open-source resources.

**SOC Analyst** is the first entry in this line. It's not a course you read — it's a project you *run*: you build a small home SOC, attack your own lab safely, and then chase that attack through every stage a real Tier 1/2 SOC analyst would — detection, hunting, incident response, forensics, malware/phishing analysis, and threat intelligence — ending with a real, written incident report.

No cybersecurity background required. No paid tools. No cloud subscriptions.

---

## 🎯 Main Goal

Take you from beginner to someone with **hands-on, practical reps across nearly the entire blue team pipeline** — not just reading about SIEM, MITRE ATT&CK, forensics, and IR, but actually doing each one against real (safely simulated) attacks, end to end:

> detect something → hunt for it → respond to it → investigate it → write it up

Everything is built around **one continuous storyline** rather than disconnected exercises: you build a lab, attack your own machine (via Atomic Red Team simulations), and follow that attack through every stage a real SOC analyst would.

---

## 🏁 What You'll Walk Away With

| # | Deliverable | What it proves |
|---|---|---|
| 1 | A working **home SOC lab** (pfSense + Windows/Linux victims + Kali attacker + Wazuh/Security Onion SIEM), isolated from your real network | You can stand up a monitored environment from scratch |
| 2 | Custom **Sigma detection rules**, tuned and mapped to MITRE ATT&CK techniques | You can go from "no visibility" to "detected and mapped" |
| 3 | One fully **documented incident** — triaged, contained, investigated (memory dump, disk artifacts, timeline, IOCs) | You can run the full IR lifecycle, not just talk about it |
| 4 | Two incident write-ups (technical + executive) and a reusable **playbook** | You can communicate to both engineers and management |
| 5 | A **GitHub repo** with your rules, notebook, report, and playbook | A concrete artifact to show in interviews |

This isn't "I learned some tools" — it's a live, demonstrable lab plus one polished case study proving you can go from raw logs to a written incident report on your own.

---

## 🧰 Hardware & Budget

| Item | Needed? | Notes |
|---|---|---|
| **PC** | Required | 16GB RAM min (32GB comfortable), 250GB+ free SSD. Runs the hypervisor + all VMs. Lower spec? See *Low-Spec Mode* below. |
| **Router + Switch** | Optional | For physically segmenting the lab off your home LAN (VLAN/subnet). Skip it and do everything with virtual switches instead — zero cost, zero risk. |
| **Everything else** | Free | 100% open-source software (see below). |

---

## 🛠️ Tools Used (all free)

- **Hypervisor:** VirtualBox (easy) or Proxmox (more "real infra")
- **Router/Firewall:** pfSense
- **SIEM:** Wazuh (lighter) or Security Onion (heavier, includes Zeek/Suricata/ATT&CK mapping)
- **Endpoint visibility:** Sysmon (Windows), auditd (Linux)
- **Detection rules:** Sigma
- **ATT&CK mapping:** MITRE ATT&CK Navigator
- **Attack simulation:** Atomic Red Team
- **Forensics:** Autopsy, KAPE, Volatility 3
- **Network analysis:** Wireshark, Zeek, Suricata
- **Malware analysis:** PEStudio, strings/exiftool, isolated sandbox VM
- **Threat intel/OSINT:** VirusTotal, AbuseIPDB, AlienVault OTX
- **Case management:** TheHive (optional) or a structured markdown log
- **Documentation:** Obsidian / plain markdown "SOC notebook"

---

## 🗺️ Core Lab Architecture

```
[Attacker VM: Kali]  --->  [Virtual Router/Firewall: pfSense]  --->  [Victim VMs: Windows 10/11, Ubuntu]
                                        │
                              [SIEM VM: Wazuh or Security Onion]  <── collects logs/agents from everything
```

All networks are virtual/isolated (host-only or internal network mode) — critical before anything malicious ever touches a VM.

---

## 📅 The 14-Day Path

<details>
<summary><strong>Day 1 — Lab Foundation</strong></summary>

Install hypervisor, set up isolated internal networks, deploy pfSense, Windows + Ubuntu victim VMs, Kali attacker VM.
**Covers:** SOC architecture basics, network fundamentals
</details>

<details>
<summary><strong>Day 2 — SIEM Stand-Up</strong></summary>

Deploy Wazuh/Security Onion, install agents/Sysmon/auditd on victims, confirm normalized log flow.
**Covers:** SIEM fundamentals, log sources, data normalization
</details>

<details>
<summary><strong>Day 3 — Queries & Dashboards</strong></summary>

Write basic queries (KQL/Wazuh query language), build 2–3 dashboards, save recurring searches.
**Covers:** Basic SIEM queries, dashboards & visualization
</details>

<details>
<summary><strong>Day 4 — MITRE ATT&CK Mapping</strong></summary>

Run 5–10 Atomic Red Team tests, map resulting logs to ATT&CK Navigator, flag blind spots.
**Covers:** ATT&CK framework, technique mapping, blind-spot identification
</details>

<details>
<summary><strong>Day 5 — Detection Rule Creation</strong></summary>

Write and import 3–5 Sigma rules for undetected techniques, re-test, tune out a false positive.
**Covers:** Detection engineering, baselining, tuning
</details>

<details>
<summary><strong>Day 6 — Threat Hunting</strong></summary>

Form 2 hypotheses, hunt proactively via SIEM queries, turn a finding into a new rule.
**Covers:** Hypothesis-driven hunting, hunt execution, IOC-based hunting
</details>

<details>
<summary><strong>Day 7 — Incident Simulation & IR</strong></summary>

Run a multi-stage attack chain, triage as a live incident, contain, document each IR lifecycle stage.
**Covers:** Full IR lifecycle, alert triage, containment
</details>

<details>
<summary><strong>Day 8 — Evidence Collection & Windows Forensics</strong></summary>

Collect memory dump then disk image, pull artifacts with KAPE, analyze with Autopsy.
**Covers:** Evidence collection, chain of custody, Windows forensics
</details>

<details>
<summary><strong>Day 9 — Memory Forensics & Timeline Analysis</strong></summary>

Analyze the memory dump with Volatility 3, build a cross-source timeline, ID patient zero.
**Covers:** Memory analysis, timeline analysis
</details>

<details>
<summary><strong>Day 10 — Phishing Analysis</strong></summary>

Craft a realistic phishing sample, analyze headers (SPF/DKIM/DMARC), safely check URL reputation.
**Covers:** Email header analysis, malicious content detection, social engineering recognition
</details>

<details>
<summary><strong>Day 11 — Malware Analysis Basics</strong></summary>

Static analysis (hash, strings, PE structure) and optional contained dynamic detonation; extract IOCs.

> ⚠️ Only ever in a fully isolated, disposable VM — no shared folders, no internet bridge.

**Covers:** Static/dynamic malware analysis, IOC extraction
</details>

<details>
<summary><strong>Day 12 — Network Security Monitoring</strong></summary>

Capture traffic with Wireshark/Zeek/Suricata, correlate an IDS alert back to the PCAP.
**Covers:** Network traffic analysis, IDS, network artifacts
</details>

<details>
<summary><strong>Day 13 — Threat Intel Enrichment</strong></summary>

Run collected IOCs through VirusTotal/AbuseIPDB/OTX, write a tactical vs. strategic intel note.
**Covers:** Threat intelligence fundamentals, OSINT
</details>

<details>
<summary><strong>Day 14 — Documentation & Playbook</strong></summary>

Write technical + executive incident reports, build a reusable playbook, publish everything to GitHub.
**Covers:** Incident reports, playbook creation, documentation
</details>

---

## 🐢 Low-Spec Mode

Got 8GB RAM or less? Run one VM at a time: attack the Windows VM, export logs, shut it down, then bring up the SIEM VM to ingest and analyze. Slower, but every exercise stays doable.

---

## ⚠️ Note on Scope

This path doesn't cover cloud security monitoring (AWS/Azure/GCP logging) or full SOAR automation — those need paid cloud accounts or enterprise tooling. A **Phase 2** using free-tier AWS/Azure logging may follow as a future CyberAtaraxia Careers entry.

---

## 👤 About the Developer

- 🎓 Network Engineering student at **Polytech Dijon**, specializing in Cybersecurity
- 🔵 Blue Teamer | ICT Auditor | Network Administrator
- 🏢 Cybersecurity Intern at **Axem Belgium**
- 🌱 Currently pursuing **Blue Team L1 · ISO 27001/2022 · CCNA**

<p align="center">
  <a href="https://linkedin.com/in/selim-jerbi-b355a0202">
    <img src="https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  &nbsp;
  <a href="mailto:selimjerbi@gmail.com">
    <img src="https://img.shields.io/badge/Gmail-Contact-D14836?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
</p>

<div align="center">
  <sub>CyberAtaraxia Careers — Open Source · Built with purpose by Selim JERBI</sub>
</div>
