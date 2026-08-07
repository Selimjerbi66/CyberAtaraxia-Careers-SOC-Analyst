# Playbook SOC — Credential Dumping (T1003.001) & chaîne associée

**Référence :** PB-SOC-CRED-001  
**Version :** 1.0  
**Date :** 07/08/2026  
**Parcours :** CyberAtaraxia Careers — SOC Analyst — **Jour final (Jour 14)**  
**Cas d’usage lab :** incident simulé Jour 7 (PowerShell / Mimikatz, ProcDump / LSASS, persistance Run)

---

## 1. Objectif

Guider l’analyste du **premier signal** (alerte Sigma, Sysmon ID 10, EDR) jusqu’à la **clôture**, pour un scénario de type :
- accès mémoire à `lsass.exe` ;
- usage de LOLBin (ex. ProcDump) ;
- éventuelle exécution PowerShell / Mimikatz ;
- éventuelle persistance registre (clés Run).

---

## 2. Déclencheurs (alertes)

| Signal | Source typique | Sévérité indicative |
|--------|----------------|---------------------|
| Suspicious LSASS Process Access | Sigma / Zircolite / SIEM | Haute |
| `TargetImage` = `lsass.exe` + droits élevés (`0x1FFFFF`) | Sysmon Event ID 10 | Haute |
| `procdump.exe` / `procdump64.exe` | Sysmon ID 1 ou 13 | Moyenne à haute selon contexte |
| CommandLine contenant Mimikatz / `DumpCreds` / `Invoke-AtomicTest` | Sysmon ID 1 | Haute |
| Création valeur sous `...\CurrentVersion\Run` | Sysmon ID 13 | Moyenne |

---

## 3. Arbre de décision

```text
Alerte Sigma / Sysmon (LSASS ou credential tools)
        │
        ▼
[1] Triage — validation de l'alerte
        │
        ├─ Faux positif probable (AV/EDR légitime, backup, admin documenté)
        │         │
        │         ▼
        │   Documenter + clôturer (faible)
        │
        └─ Vrai positif ou indéterminé
                  │
                  ▼
[2] Confinement
        │
        ├─ Poste critique / dump confirmé / persistance → isoler du réseau (VLAN / port group / lien)
        │
        └─ Risque faible + investigation locale possible → surveillance renforcée
                  │
                  ▼
[3] Collecte d'artefacts (ordre volatilité)
        │
        ├─ Mémoire (si machine encore allumée)
        ├─ Export journaux (Sysmon, Security)
        ├─ Disque / triage (Prefetch, registre, fichiers .dmp)
        └─ Notes horodatées (5-tuple si réseau)
                  │
                  ▼
[4] Analyse
        │
        ├─ Corréler ID 1 / 10 / 13
        ├─ Identifier Patient Zero (processus initial)
        ├─ Mémoire (Volatility) + disque (KAPE / registre)
        └─ Enrichir hash / IP / domaine (VT, AbuseIPDB, OTX)
                  │
                  ▼
[5] Éradication
        │
        ├─ Stop processus malveillants s'ils tournent encore
        ├─ Supprimer persistance (Run, tâches, services)
        ├─ Supprimer dumps et payloads résiduels
        └─ Vérifier absence des IOC fichiers / registre
                  │
                  ▼
[6] Récupération
        │
        ├─ Contrôles post-nettoyage (processus, Run, scheduled tasks)
        ├─ Rotation secrets si credentials exposés
        └─ Réactiver défenses (AV, politiques)
                  │
                  ▼
[7] Lessons learned + clôture ticket
```

---

## 4. Checklist — Triage

- [ ] Horodatage de l’alerte noté (UTC + heure locale)
- [ ] Machine / utilisateur / `SourceImage` / `TargetImage` / `GrantedAccess`
- [ ] Corrélation Sysmon ID 1 (création) et ID 10 (accès)
- [ ] Contexte : admin légitime, EDR, sauvegarde, lab Atomic ?
- [ ] Décision : FP documenté **ou** passage confinement / collecte

---

## 5. Checklist — Confinement

- [ ] Évaluer criticité métier du poste
- [ ] Isoler la VM / le poste du réseau de production (lab : port group isolé déjà en place)
- [ ] Préserver l’état (éviter reboot si dump mémoire encore nécessaire)
- [ ] Informer le canal IR / responsable lab

---

## 6. Checklist — Collecte

- [ ] Dump mémoire (outil validé, source officielle)
- [ ] Export Sysmon/Security (`wevtutil epl` …)
- [ ] Prefetch, Run keys, fichiers `Temp\*.dmp`
- [ ] Hash SHA256 des binaires suspects
- [ ] Journaliser la chaîne de custody (qui / quand / où)

---

## 7. Checklist — Éradication & récupération

- [ ] `Stop-Process` sur outils encore actifs (Mimikatz, ProcDump, etc.)
- [ ] Cleanup Atomic / suppression clé `Run`
- [ ] Suppression `lsass_dump.dmp` et leurres (`*.docm`)
- [ ] Re-vérifier processus, tâches planifiées, HKCU/HKLM Run
- [ ] Réactiver protection temps réel si désactivée pour investigation
- [ ] Rotation des mots de passe / secrets si dump confirmé **en production**

---

## 8. Critères de clôture

| Critère | OK |
|---------|----|
| Persistance retirée et vérifiée | ☐ |
| Pas de processus outil de dump actif | ☐ |
| Artefacts fichiers sensibles supprimés ou sécurisés | ☐ |
| Timeline et IOC documentés | ☐ |
| FP vs TP tranché et motivé | ☐ |
| Lessons learned enregistrées | ☐ |

---

## 9. Lien avec le lab CyberAtaraxia

Ce playbook s’appuie sur la simulation **Jour 7** et la détection **Jour 5** (Sigma LSASS), la forensique **Jours 8–9**, le phishing **Jour 10**, le malware **Jour 11**, le réseau **Jour 12** et la CTI **Jour 13**.

*Document — **jour final** du parcours SOC Analyst.*
