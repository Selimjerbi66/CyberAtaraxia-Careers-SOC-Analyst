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

**SOC Analyst** is the first entry in this line. It's not a course you read — it's a project you *run*: you build a small home lab, attack your own machine (via Atomic Red Team simulations), and then chase that attack through every stage a real Tier 1/2 SOC analyst would — detection, hunting, incident response, forensics, malware/phishing analysis, and threat intelligence — ending with a real, written incident report.

No cybersecurity background required. No paid tools. No cloud subscriptions. **No central SIEM server required** — this path uses lightweight, file-based detection tools instead, so the whole lab fits in just 3 VMs.

---

## 🎯 Main Goal

Take you from beginner to someone with **hands-on, practical reps across nearly the entire blue team pipeline** — not just reading about detection engineering, MITRE ATT&CK, forensics, and IR, but actually doing each one against real (safely simulated) attacks, end to end:

> detect something → hunt for it → respond to it → investigate it → write it up

Everything is built around **one continuous storyline** rather than disconnected exercises: you build a lab, attack your own machine (via Atomic Red Team simulations), and follow that attack through every stage a real SOC analyst would.

---

## 🏁 What You'll Walk Away With

| # | Deliverable | What it proves |
|---|---|---|
| 1 | A working **home lab** (Windows + Linux + Kali attacker) — just 3 VMs, isolated from your real network | You can stand up a monitored environment from scratch, without needing heavy infrastructure |
| 2 | Custom **Sigma detection rules**, tuned and mapped to MITRE ATT&CK techniques, run with a lightweight local engine | You can go from "no visibility" to "detected and mapped" without a SIEM |
| 3 | One fully **documented incident** — triaged, contained, investigated (memory dump, disk artifacts, timeline, IOCs) | You can run the full IR lifecycle, not just talk about it |
| 4 | Two incident write-ups (technical + executive) and a reusable **playbook** | You can communicate to both engineers and management |
| 5 | A **GitHub repo** with your rules, notebook, report, and playbook | A concrete artifact to show in interviews |

This isn't "I learned some tools" — it's a live, demonstrable lab plus one polished case study proving you can go from raw logs to a written incident report on your own.

---

## 🧰 Hardware & Budget

| Item | Needed? | Notes |
|---|---|---|
| **PC / host machine** | Required | 8–16GB RAM workable (16GB comfortable), **~80–100GB free storage is enough** since there's no SIEM server VM. Runs your hypervisor of choice — ESXi, VirtualBox, or Proxmox all work fine for this program. Lower spec? See *Low-Spec Mode* below. |
| **Router + Switch** | Optional | Only useful if you want to physically segment the lab from your home LAN. Not required — see *Networking, No Firewall Appliance Needed* below. |
| **Everything else** | Free | 100% open-source software (see below). |

---

## 🔌 Networking — No Firewall Appliance Needed

Earlier drafts of this plan included pfSense as a virtual router/firewall for realism. **It's optional and has been removed from the core path** — it adds a real ESXi/hypervisor resource and configuration burden for zero required syllabus coverage. Every single day below works without it.

**What to do instead:**
- Create one **isolated internal network** for your lab — in ESXi this is a vSwitch/port group with **no physical uplink** (no NIC attached to it); in VirtualBox use "Internal Network" mode; in Proxmox use a Linux bridge with no physical interface attached.
- Put every VM (Windows victim, Ubuntu victim, Kali attacker) on that same isolated network with static private IPs (e.g. `10.10.10.10/24`, `.20`, `.30`).
- You'll need internet access briefly during setup (OS updates, installing Sysmon/Zircolite, downloading Atomic Red Team, Sigma rules). Two easy ways to handle this without a firewall VM:
  - Temporarily attach a VM to a NAT/bridged network to download what you need, then move its virtual NIC back to the isolated internal network before running any attack simulation.
  - Or keep a **separate "staging" port group** with internet access purely for downloads, and only move VMs to the isolated segment once they're fully patched and provisioned.
- **Take a clean snapshot of every VM immediately after this initial setup.** This is your single most important safety net for the whole two weeks — you will want to roll back after the malware analysis day and after the incident simulation. To save disk space, keep only **one active snapshot per VM** at a time (consolidate/delete older ones once a day's work is documented), and provision virtual disks **thin-provisioned** rather than thick.
- If you later want the "real infrastructure" feel of a virtual router/firewall in the traffic path, **OPNsense** is a lighter, often easier install than pfSense and can be added as a nice-to-have upgrade once the core lab is stable — never a Day 1 blocker.

---

## 🛠️ Tools Used (all free, no server/agent stack required)

- **Hypervisor:** ESXi, VirtualBox, or Proxmox — whichever you can get running reliably
- **Endpoint visibility:** Sysmon (Windows) with a community config (e.g. SwiftOnSecurity's or Olaf Hartong's `sysmon-modular`), auditd (Linux) — logs stay local, exported on demand instead of streamed to a SIEM
- **Detection engine (no server needed):** **Zircolite** — runs Sigma rules directly against exported EVTX/JSON logs from your own machine, no manager, no agent, no indexing cluster. **Chainsaw** or **Hayabusa** are drop-in alternatives if you want to compare tools
- **Detection rules:** Sigma (rules used natively by Zircolite — no format conversion needed)
- **ATT&CK mapping:** MITRE ATT&CK Navigator (web-based, free)
- **Attack simulation:** Atomic Red Team (`Invoke-AtomicRedTeam` PowerShell module)
- **Forensics:** KAPE, Autopsy, Volatility 3, Magnet RAM Capture or DumpIt (memory acquisition), FTK Imager Lite (disk imaging)
- **Network analysis:** Wireshark, Zeek, Suricata (with the free ET Open ruleset) — run temporarily on the Kali VM when needed, no dedicated monitoring VM required
- **Malware analysis:** PEStudio, `strings`/`sha256sum`, Process Monitor/Process Explorer/Autoruns, an isolated sandbox VM
- **Threat intel/OSINT:** VirusTotal, AbuseIPDB, AlienVault OTX
- **Case management:** A structured markdown log (TheHive dropped — another server you don't need for this path)
- **Documentation:** Obsidian / plain markdown "SOC notebook"

> 💾 **Why this matters for storage:** dropping the SIEM manager/agent stack removes the single biggest disk consumer in the original plan (constant log indexing + a dedicated server VM). Zircolite itself is a single binary under 50MB, and you only keep the small JSON output — not the raw exported logs — once you've reviewed a day's results.

---

## 🗺️ Core Lab Architecture

```
[Attacker VM: Kali]  ─┐
[Victim VM: Windows]  ─┼──  isolated internal vSwitch / port group (no physical uplink)
[Victim VM: Ubuntu]   ─┘
```

Just 3 VMs, one flat isolated network segment, static IPs, no internet once provisioning is done. No SIEM/server VM. Detection happens **on-demand**: export logs from Windows/Ubuntu after a simulation, run Zircolite against them (from any of the 3 VMs, or your host machine if you install it there), review the JSON output, then delete the raw export.

---

## 📅 The 14-Day Path

<details>
<summary><strong>Day 1 — Lab Foundation</strong></summary>

- Install/confirm your hypervisor is stable (ESXi/VirtualBox/Proxmox).
- Create the isolated internal network (vSwitch/port group with no uplink, or "Internal Network" mode in VirtualBox).
- Provision three VMs on a temporary internet-connected segment first: **Windows 10/11**, **Ubuntu Server 22.04**, **Kali Linux**. Fully patch each one. Create their virtual disks **thin-provisioned**.
- Assign static IPs once provisioned (e.g. `10.10.10.10` Windows, `.20` Ubuntu, `.30` Kali), then move all three VMs' virtual NICs onto the isolated segment.
- Verify connectivity: `ping` between all three VMs; confirm no VM can reach the real internet from the isolated segment.
- **Snapshot every VM now** — label it clearly, e.g. `clean-baseline-day1`. This is your only active snapshot per VM for now.
- Draw your network diagram (IPs, VM roles) into your SOC notebook.

**Covers:** SOC architecture basics, network fundamentals
</details>

<details>
<summary><strong>Day 2 — Log Sources & Local Detection Engine Setup</strong></summary>

- Install **Sysmon** on the Windows VM with a community config (`sysmon64.exe -i sysmonconfig.xml`) — this is what gives you rich process/network/registry telemetry instead of default Windows logging.
- Configure **auditd** rules on the Ubuntu VM to watch key files and syscalls, e.g.:
  ```
  -w /etc/passwd -p wa -k identity
  -w /etc/shadow -p wa -k identity
  -a always,exit -F arch=b64 -S execve -k exec
  ```
- Limit log growth so nothing eats your disk: cap Windows Event Log channel sizes (`wevtutil sl <log> /ms:<bytes>`, e.g. 50–100MB per channel instead of the default) and configure auditd rotation in `/etc/audit/auditd.conf` (`max_log_file`, `num_logs`).
- Install **Zircolite** (single binary, download from GitHub releases) — no install needed beyond that, and no agent to deploy on the victim VMs.
- Do a first test: export the Windows Security/Sysmon channels to EVTX (`wevtutil epl Microsoft-Windows-Sysmon/Operational sysmon_export.evtx`), run Zircolite against it with a default Sigma ruleset, and confirm you get a JSON output with matches.
- Move both victim VMs back to the isolated segment once telemetry is confirmed flowing into local logs.

**Covers:** SIEM-equivalent fundamentals, log sources, data normalization — without a SIEM server
</details>

<details>
<summary><strong>Day 3 — Manual Queries & Log Review</strong></summary>

- Learn to query exported logs directly instead of a dashboard. On Windows, use `Get-WinEvent` with XPath filters, e.g.:
  - Failed logons: `Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]"`
  - Successful logons: `Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]"`
  - Process creation (Sysmon Event ID 1): `Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]"`
  - Network connection (Sysmon Event ID 3): same log, `EventID=3`
- On Ubuntu, use `ausearch -k identity` / `ausearch -k exec` to pull the equivalent auditd events.
- Instead of Wazuh-style dashboards, build **3 saved review routines** in your SOC notebook: (1) failed logons by source, (2) process creations by parent/child image, (3) outbound connections timeline — each just a saved PowerShell/`ausearch` one-liner plus a note on what "normal" looks like.
- Export a small sample of each into CSV (`Export-Csv`) so you have something concrete to reuse during hunting later, then delete the raw EVTX exports to save space.

**Covers:** Basic log analysis, manual "query" equivalents, baseline documentation
</details>

<details>
<summary><strong>Day 4 — MITRE ATT&CK Mapping</strong></summary>

- Install the `Invoke-AtomicRedTeam` PowerShell module on the Windows victim.
- Run these specific tests one at a time, exporting the relevant log channel and checking with Zircolite (or a manual `Get-WinEvent` query) after each:
  - `Invoke-AtomicTest T1059.001 -TestNumbers 1` (PowerShell execution)
  - `Invoke-AtomicTest T1003.001 -TestNumbers 1` (LSASS credential dumping simulation)
  - `Invoke-AtomicTest T1547.001 -TestNumbers 1` (Registry Run Key persistence)
- For each test, find the matching event(s): Sysmon Event ID 1 for process creation, Event ID 10 for process access on the LSASS test, Event ID 13 for registry value set on the persistence test.
- Open the **MITRE ATT&CK Navigator** (mitre-attack.github.io/attack-navigator), create a new layer, and color each tested technique: green = detected, red = no visibility.

**Covers:** ATT&CK framework, technique mapping, blind-spot identification
</details>

<details>
<summary><strong>Day 5 — Detection Rule Creation</strong></summary>

- Write a Sigma rule for each red (undetected) technique from Day 4. Example — detecting LSASS access consistent with credential dumping:
  ```yaml
  title: Suspicious LSASS Process Access
  logsource:
    category: process_access
    product: windows
  detection:
    selection:
      TargetImage|endswith: '\lsass.exe'
      GrantedAccess: '0x1010'
    condition: selection
  level: high
  ```
- Drop the rule straight into your Zircolite rules folder — **no conversion needed**, Zircolite consumes Sigma natively.
- Re-run the corresponding Atomic test from Day 4, export the relevant log, run Zircolite, and confirm the new match appears at the expected level.
- **Tune one rule**: find a legitimate action triggering a false positive (e.g. an antivirus scanner also accessing LSASS) and add an exclusion condition to the Sigma rule.

**Covers:** Detection engineering, baselining, tuning
</details>

<details>
<summary><strong>Day 6 — Threat Hunting</strong></summary>

- Hypothesis 1: *"An attacker is using LOLBins to download files."* Hunt query: export process creation logs, `grep`/`Select-String` for `certutil.exe` with `urlcache` or `-decode` in the command line.
- Hypothesis 2: *"There's PowerShell activity outside normal working hours."* Hunt query: export `powershell.exe` process creation events, filter to outside 09:00–17:00 in your spreadsheet/CSV.
- Use a structured hunt log template for each: **Hypothesis → Data sources used → Query run → Findings → Verdict (confirmed/ruled out)**.
- Turn at least one confirmed finding into a new Sigma rule dropped into your Zircolite ruleset, so it's caught automatically next time you export and scan logs.

**Covers:** Hypothesis-driven hunting, hunt execution, IOC-based hunting
</details>

<details>
<summary><strong>Day 7 — Incident Simulation & IR</strong></summary>

- Run a chained sequence to simulate a real intrusion: `T1204.002` (user opens a malicious document, simulated manually) → `T1059.001` (PowerShell execution) → `T1547.001` (Registry Run Key persistence).
- Practice the full IR lifecycle against it:
  - **Detection**: export the relevant log channel, run Zircolite with your Day 5 rules, confirm the match.
  - **Triage**: assign a severity using a simple matrix (asset criticality × technique impact).
  - **Containment**: isolate the VM — detach its vNIC or move its port group to a "quarantine" segment with no connectivity.
  - **Eradication**: remove the persistence registry key, kill the malicious process.
  - **Recovery**: restore from your Day 1 snapshot, or manually verify the system is clean.
  - **Lessons Learned**: note what was easy/hard to detect and why.
- Log everything using a simple ticket template: Incident ID, timestamp, detection source, severity, actions taken, who/when.

**Covers:** Full IR lifecycle, alert triage, containment
</details>

<details>
<summary><strong>Day 8 — Evidence Collection & Windows Forensics</strong></summary>

- **Before** cleaning up Day 7's incident: capture memory first (volatile data first) using **Magnet RAM Capture** or **DumpIt**.
- Then capture a disk image using **FTK Imager Lite**, or export the VM's virtual disk directly since this is a lab.
- Run **KAPE** for a full artifact sweep in one command:
  ```
  kape.exe --tsource C: --target KapeTriage --tdest C:\out
  ```
  This pulls Prefetch, Event Logs, registry hives, MFT, USN Journal, Amcache, and ShimCache in one pass.
- Load the collected artifacts into **Autopsy**: create a new case, add the disk image/logical files, and explore the Registry Explorer and timeline modules.
- Specifically check: `SYSTEM`/`SOFTWARE` hives for Run keys, `Amcache.hve` for execution evidence, and `.pf` Prefetch files for execution timestamps.
- Once you've extracted what you need, move the raw memory dump and disk image off the VM's own disk (external drive or delete after copying findings into your notebook) — these files are large and don't need to live in the lab long-term.

**Covers:** Evidence collection, chain of custody, Windows forensics
</details>

<details>
<summary><strong>Day 9 — Memory Forensics & Timeline Analysis</strong></summary>

- Analyze the Day 8 memory dump with **Volatility 3**:
  - `vol -f memdump.raw windows.pslist` and `windows.pstree` — process listing/parentage
  - `windows.netscan` — network connections at time of capture
  - `windows.malfind` — detect possible code injection
  - `windows.cmdline` — command lines of running processes
- Cross-reference the `powershell.exe` process chain from Day 7 in these outputs.
- Build a simple timeline by combining KAPE's exported CSVs with Volatility's timestamped output in a spreadsheet, sorted chronologically.
- Identify the **patient zero** event (the first malicious action) and the full scope of what it touched.

**Covers:** Memory analysis, timeline analysis
</details>

<details>
<summary><strong>Day 10 — Phishing Analysis</strong></summary>

- Build a realistic test phishing email yourself — either a raw `.eml` file you construct manually, or using a free mail sandbox (e.g. Mailtrap/Mailhog). Deliberately mismatch the `From`/`Return-Path` and make SPF fail.
- Manually read the headers: look at the `Authentication-Results` header for `spf=fail`/`dkim=fail`, and compare the displayed sender against the actual originating server in `Received` headers.
- Defang before documenting anything: `http` → `hxxp`, `.` → `[.]`.
- Check the (fake/test) URL and domain reputation using VirusTotal's URL scan or urlscan.io — without visiting it directly in a browser.

**Covers:** Email header analysis, malicious content detection, social engineering recognition
</details>

<details>
<summary><strong>Day 11 — Malware Analysis Basics</strong></summary>

- Get one labeled, known sample from **MalwareBazaar** (free account/API key required). Handle it **only** inside a fully isolated, offline VM restored from a fresh snapshot — no shared folders, no bridged/NAT network.
- **Static analysis**: `sha256sum sample`, `strings -n 8 sample | less`, then load it into **PEStudio** to check imports, sections, and entropy (high entropy across sections is a packer indicator).
- **Dynamic analysis (optional, more advanced)**: detonate inside the isolated VM, monitor with **Process Monitor**/**Process Explorer**, check persistence with **Autoruns**, and capture any generated traffic with Wireshark on the isolated segment (it will go nowhere — that's fine, you're just observing behavior).
- Extract IOCs: file hash, any dropped file paths, mutex names, and any (sinkholed/non-routable) IPs or domains contacted.

> ⚠️ Only ever in a fully isolated, disposable VM. Restore your snapshot immediately after this day.

**Covers:** Static/dynamic malware analysis, IOC extraction
</details>

<details>
<summary><strong>Day 12 — Network Security Monitoring</strong></summary>

- Enable **promiscuous mode** on your isolated port group (in ESXi: Security policy → Promiscuous Mode → Accept) so traffic can be observed.
- Run **Suricata** or **Zeek** temporarily on the **Kali VM** (no dedicated monitoring VM needed) in IDS mode with the free **ET Open** ruleset, attached to the isolated segment.
- Re-run one of your earlier Atomic tests (Day 4 or 7) while capturing with **Wireshark** simultaneously.
- Correlate any Suricata/Zeek alert back to the exact packets in the Wireshark capture using the alert's timestamp and 5-tuple (src/dst IP and port, protocol).
- Delete large `.pcap` captures once you've pulled out the relevant packets/screenshots for your notebook.

**Covers:** Network traffic analysis, IDS, network artifacts
</details>

<details>
<summary><strong>Day 13 — Threat Intel Enrichment</strong></summary>

- Compile every IOC collected across the whole project into one spreadsheet: hash / IP / domain / which day it came from / associated technique.
- Look each one up: **VirusTotal** (hash/URL reputation), **AbuseIPDB** (IP reputation score), **AlienVault OTX** (pulse/context search).
- Write a short intel note distinguishing **tactical intelligence** (this specific hash/IP is/isn't known-bad) from **strategic intelligence** (the overall pattern — e.g. "LOLBin usage + registry persistence" — maps to common commodity malware behavior, not a targeted APT).

**Covers:** Threat intelligence fundamentals, OSINT
</details>

<details>
<summary><strong>Day 14 — Documentation & Playbook</strong></summary>

- Write the Day 7 incident up properly with this structure: Executive Summary, Timeline, Technical Details, IOC table, Impact, Root Cause, Recommendations.
- Produce **two versions**: a full technical report (log excerpts, artifact references, Volatility/KAPE output) and a 1-page executive summary (business impact and risk framed, no jargon).
- Build a **playbook** for this incident type: a decision tree — Alert trigger criteria → Triage steps → Containment steps → Eradication checklist → Recovery validation → Closure.
- Push everything to a GitHub repo with a clear structure, e.g.:
  ```
  /sigma-rules
  /soc-notebook
  /reports
  /playbooks
  ```

**Covers:** Incident reports, playbook creation, documentation
</details>

---

## 🐢 Low-Spec Mode

Got 8GB RAM or less, or very limited disk space? Run one VM at a time: attack the Windows VM, export logs, shut it down, then bring up whichever VM you need next to run Zircolite/hunt/analyze. Since there's no always-on SIEM server to keep running, this mode fits naturally with the architecture — slower, but every exercise stays doable.

---

## ⚠️ Note on Scope

This path doesn't cover cloud security monitoring (AWS/Azure/GCP logging), full SOAR automation, or centralized multi-host SIEM operation (correlating alerts live across many hosts) — those need paid cloud accounts, enterprise tooling, or more storage/compute than a 3-VM home lab. If you later have the disk space for a 4th VM, adding Wazuh or Security Onion back in as a **Phase 2 upgrade** is a natural next step once the fundamentals here are solid. A **Phase 2** using free-tier AWS/Azure logging may also follow as a future CyberAtaraxia Careers entry.

---

## 👤 About the Developer

- 🎓 Network Engineering student at **Polytech Dijon**, specializing in Cybersecurity
- 🔵 Blue Teamer | ICT Auditor | Network Administrator
- 🏢 Cybersecurity Intern at **Axem Belgium**
- 🌱 Currently pursuing **Blue Team L1 · ISO 27001/2022 · CCNA**

<p align="center">
  <a href="https://linkedin.com/in/selim-jerbi-b355a0202">
    <img src="https://img.shields.io/badge/LinkedIn-Connect%20with-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  &nbsp;
  <a href="mailto:selimjerbi@gmail.com">
    <img src="https://img.shields.io/badge/Gmail-Contact-D14836?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
</p>

<div align="center">
  <sub>CyberAtaraxia Careers — Open Source · Built with purpose by Selim JERBI</sub>
</div>
