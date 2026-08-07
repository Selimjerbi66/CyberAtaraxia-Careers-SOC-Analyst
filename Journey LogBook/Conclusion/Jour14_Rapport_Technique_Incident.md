<div align="center">

# Rapport technique d'incident
### Simulation lab — Chaîne Atomic Red Team (Jour 7) & corrélation multi-jours
### **Jour final** du parcours SOC Analyst — CyberAtaraxia Careers

</div>

---

**Référence :** INC-LAB-2026-08-04-001  
**Date du rapport :** 07/08/2026  
**Machine concernée :** DESKTOP-33KBPU8 (Tiny10) — `10.10.10.10`  
**Analyste :** Lab SOC — CyberAtaraxia Careers  
**Classification :** Exercice de laboratoire (réseau isolé)

---

## 1. Résumé exécutif

Le 04/08/2026, une chaîne d'attaque simulée a été exécutée sur la machine Windows Tiny10 dans le cadre du parcours SOC. La séquence combinait un leurre documentaire (T1204.002), l'exécution de Mimikatz via PowerShell (T1059.001) et une persistance par clé de registre Run (T1547.001).

La télémétrie Sysmon a capturé la création de processus et les accès mémoire à `lsass.exe`. Les règles Sigma développées au Jour 5 (`Suspicious LSASS Process Access`) ont produit des détections de niveau **HIGH** sur les événements ProcDump (GrantedAccess `0x1FFFFF`). L'éradication de la clé de persistance a été confirmée ; un artefact résiduel du Jour 5 (`lsass_dump.dmp`) a été identifié lors de la phase de récupération.

L'enrichissement CTI du Jour 13 a confirmé que l'échantillon ELF analysé au Jour 11 (SHA256 `11f14d6…e8328`) est un malware **Mirai** largement détecté (ordre de grandeur ~38/62 sur VirusTotal). Aucun domaine ni IP publique C2 n'a été extrait de cet échantillon en analyse statique.

**Sévérité lab :** Élevée (credential access + persistance).  
**Impact business réel :** nul (environnement isolé, simulation contrôlée).

---

## 2. Périmètre (Scope)

| Élément | Détail |
|--------|--------|
| Système | Tiny10 (Windows 10 allégé), hostname `DESKTOP-33KBPU8` |
| Réseau | Segment isolé `10.10.10.0/24` (sans uplink Internet) |
| Fenêtre d'activité principale | 04/08/2026 (simulation Jour 7) ; artefacts liés aux Jours 4–5 |
| Sources de preuves | Sysmon/Operational, Atomic Red Team, KAPE, Autopsy, Volatility 3, Zircolite |
| Hors périmètre | Analyse dynamique de l'ELF ARM ; trafic Internet réel (isolation volontaire) |

---

## 3. Timeline

| Horodatage (approx.) | Événement | Source |
|----------------------|-----------|--------|
| Jour 4 (03/08) | Installation Atomic Red Team ; tests T1059.001, T1003.001, T1547.001 | Carnet J4 |
| 03/08/2026 ~12:01 UTC | Accès `procdump.exe` / `procdump64.exe` → `lsass.exe`, GrantedAccess `0x1fffff` | Sysmon ID 10, `resultat_day5` / `day7_incident.evtx` |
| Jour 5 | Confirmation Event ID 10 ; création et validation règles Sigma LSASS | Carnet J5, Zircolite |
| Jour 5 (non nettoyé) | Fichier `C:\Windows\Temp\lsass_dump.dmp` (~55 Mo) laissé sur disque | Découvert J7 |
| 04/08/2026 ~10:29 | Création leurre `Facture_Q3_2026.docm` (T1204.002 simulé) | Carnet J7 |
| 04/08/2026 | Relance T1059.001 (Mimikatz) + T1547.001 (clé Run) | Atomic + Sysmon |
| 04/08/2026 | Export `day7_incident.evtx` | wevtutil |
| 04/08/2026 | Cleanup clé Run Atomic ; vérifications processus / tâches / Run HKLM-HKCU | IR J7 |
| Jour 8 | Prefetch ProcDump confirmé ; Run vide sur image froide ; dump mémoire + VMDK | KAPE, PECmd, RegistryExplorer, DumpIt |
| Jour 9 | pslist/pstree/cmdline/netscan/malfind ; Patient Zero = PowerShell / Invoke-AtomicTest | Volatility 3 |
| Jour 10 | Phishing simulé (SPF/DKIM/DMARC fail, domaine leurre Microsoft) | Analyse `.eml` |
| Jour 11 | Analyse statique ELF ARM ; hash ; chaînes SSDP ; pas d'URL C2 | file, strings, readelf, objdump |
| Jour 12 | Suricata ET Open + Wireshark ; trafic ICMP lab ; pas d'alerte sur trafic bénin | Suricata, pcap `070826.pcapng` |
| Jour 13 | Inventaire IOC + enrichissement VT (Mirai) + note TI tactique/stratégique | Excel IOC, VT |

---

## 4. Détails techniques

### 4.1 Chaîne d'attaque (Jour 7)

| Ordre | Technique | Action | Preuve |
|------|-----------|--------|--------|
| 1 | T1204.002 | Leurure `Facture_Q3_2026.docm` | Fichier créé dans Downloads |
| 2 | T1059.001 | `Invoke-AtomicTest` → Mimikatz / dump credentials | Sysmon ID 1 (CommandLine) |
| 3 | T1547.001 | Clé `HKCU\...\Run\Atomic Red Team` | Sysmon ID 13 ; cleanup Atomic |
| (lié J4/J5) | T1003.001 | ProcDump → dump LSASS | Prefetch, Sysmon ID 10, fichier `.dmp` |

### 4.2 Détection (Sigma / Zircolite)

Règle : **Suspicious LSASS Process Access**  
Condition : `TargetImage` se termine par `\lsass.exe` **et** `GrantedAccess = 0x1FFFFF`.

Extraits représentatifs (Sysmon Event ID 10) :

- **UtcTime :** `2026-08-03 12:01:03.630`  
- **SourceImage :** `C:\AtomicRedTeam\ExternalPayloads\procdump.exe`  
- **TargetImage :** `C:\Windows\system32\lsass.exe`  
- **SourceUser :** `DESKTOP-33KBPU8\victim`  
- **Computer :** `DESKTOP-33KBPU8`

Variante tunée : exclusion de `SourceImage` se terminant par `\MsMpEng.exe` (même nombre de hits ProcDump sur le jeu de test).

Règle annexe (Jour 6) : **Suspicious Certutil Usage** — détection de `certutil.exe` avec `urlcache` ou `-decode` (T1105), validée à 1 hit MEDIUM sur le dataset de chasse.

### 4.3 Forensique disque et mémoire

- **Prefetch :** `PROCDUMP.EXE-*.pf`, `PROCDUMP64.EXE-*.pf` (KAPE / PECmd).
- **Amcache :** ProcDump non vu (délai de mise à jour) — divergence documentée.
- **Registre (image froide) :** clé Run Atomic absente après cleanup.
- **Volatility 3 :** pas d'injection significative (`malfind`) ; pas de connexion réseau suspecte persistante (`netscan`) ; reconstruction de chaîne via corrélation multi-sources.
- **Patient Zero :** `powershell.exe` / `Invoke-AtomicTest`.

### 4.4 Malware ELF (Jour 11) — enrichissement CTI

- Format : ELF 32-bit ARM, interpréteur `ld-uClibc.so.0`, stripped.
- VT (synthèse carnet J13) : **38/62** détections, famille **Mirai**, backdoor Linux / DDoS.
- Pas de C2 domaine/URL dans les chaînes ; discovery SSDP locale.

### 4.5 Réseau (Jour 12)

- Suricata ET Open (~52 220 règles), interface `eth0`, HOME_NET `10.10.10.0/24`.
- Capture Wireshark : trafic ICMP inter-VM ; **pas d'alerte** sur ping bénin (comportement IDS attendu).
- Alerte récurrente non liée au test : hostname Kali dans requête DHCP.

---

## 5. Tableau des IOC (synthèse incident & lab)

| IOC | Type | Jour | Technique | Commentaire |
|-----|------|------|-----------|-------------|
| `11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328` | SHA256 | 11 | Mirai / IoT | 38/62 VT, famille Mirai |
| `procdump64.exe` (chemin Atomic ExternalPayloads) | Fichier / LOLBin | 4–7 | T1003.001 | Accès LSASS `0x1FFFFF` |
| `C:\Windows\Temp\lsass_dump.dmp` | Fichier | 5 (résidu J7) | T1003.001 | ~55 Mo, nettoyage IR |
| `HKCU\...\Run\Atomic Red Team` | Registre | 4–7 | T1547.001 | Éradiqué (cleanup + RegistryExplorer) |
| CommandLine Mimikatz / Invoke-AtomicTest | Processus | 4–7 | T1059.001 | Sysmon ID 1 |
| `Facture_Q3_2026.docm` | Fichier leurre | 7 | T1204.002 | Simulation phishing PJ |
| `microsoft-secure-login[.]com` | Domaine | 10 | T1566.002 | Simulation lab uniquement |
| `185[.]222[.]111[.]15` | IP | 10 | T1566.002 | Simulation lab uniquement |
| SSDP `M-SEARCH` / `255.255.255.255:1900` | Signature réseau | 11 | T1046 | Discovery IoT |
| `127.0.0.1` | IP | 11 | — | Loopback, non actionnable |

Référentiel détaillé : `Jour13_IOC_Threat_Intelligence.xlsx`.

---

## 6. Impact

| Domaine | Impact lab | Impact si production |
|---------|------------|----------------------|
| Confidentialité | Dump LSASS / hashes NTLM exposés sur la VM | Compromission d'identifiants, mouvement latéral |
| Intégrité | Clé Run de persistance (supprimée) | Exécution au logon |
| Disponibilité | Non impactée | Mirai : risque DDoS / botnet IoT |
| Métier | Nul (réseau isolé, simulation) | Selon exposition réelle des assets |

---

## 7. Cause racine

**Cause racine (scénario Jour 7) :** exécution contrôlée d'Atomic Red Team par l'opérateur du lab (PowerShell administrateur), avec enchaînement volontaire credential access + persistance.

**Facteurs contributifs :**
- Défense temps réel désactivée pour permettre les tests (elle-même journalisée — T1562.001).
- Artefact `lsass_dump.dmp` non supprimé après le Jour 5 (lacune de hygiene post-test).

**Cause racine (échantillon J11) :** déploiement d'un binaire type **Mirai** (ELF ARM), orienté compromission d'équipements et activité réseau/discovery, sans C2 explicite dans les chaînes statiques analysées.

---

## 8. Recommandations

1. **Détection :** conserver Sysmon ID 1, 10, 13 ; règles Sigma LSASS (version tunée) et certutil LOLBin ; Suricata ET Open sur le segment de lab.
2. **IR :** checklist d'éradication systématique après chaque test (processus, Run keys, `.dmp`, leurres, exports).
3. **Forensique :** toujours corréler Prefetch, Amcache, registre et mémoire ; une source unique est insuffisante.
4. **CTI :** enrichir tout nouveau hash sur VT / MalwareBazaar / OTX ; ne pas traiter les IOC de simulation phishing comme des IOC de production.
5. **Réseau :** maintenir l'isolation ; mode promiscuous uniquement sur le port group de lab ; purger les gros `.pcap` après extraction des preuves.

---

## 9. Références d'artefacts (lab)

| Artefact | Référence |
|----------|-----------|
| Hash malware | `11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328` |
| Hostname | `DESKTOP-33KBPU8` |
| Dump mémoire J8/J9 | `DESKTOP-33KBPU8-20260804-110954.raw` |
| Prefetch | `PROCDUMP.EXE-*.pf`, `PROCDUMP64.EXE-*.pf` |
| Capture J12 | `070826.pcapng` |
| Règles Sigma | `lsass_access.yml`, `lsass_access_tuned.yml`, `certutil_lolbin.yml` |
| Tableur CTI | `Jour13_IOC_Threat_Intelligence.xlsx` |

---

*Document produit dans le cadre du **jour final** du parcours CyberAtaraxia Careers — SOC Analyst. Environnement de laboratoire isolé — aucune impact sur un système de production.*
EOF
