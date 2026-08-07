<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 14 — Documentation & playbook (**jour final**)

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 07/08/2026  

**Objectif du jour :** clôturer le parcours en produisant la documentation d’incident professionnelle (rapport technique + résumé exécutif), un playbook opérationnel pour le scénario de credential dumping / chaîne associée, et en centralisant les livrables (IOC, règles Sigma, carnets) dans une structure de portfolio type dépôt GitHub.

**Environnement du lab :** Kali (`10.10.10.30`), Tiny10 (`10.10.10.10`), Ubuntu Server (`10.10.10.20`) — voir carnet Jour 1 pour le détail de l’architecture.

**Statut du parcours :** **jour final** — synthèse des Jours 1 à 13.

---

## ✅ Ce qui a été fait aujourd’hui

### Assemblage de la documentation

Les carnets des Jours 1 à 13, le tableur d’IOC du Jour 13, les résultats Zircolite (dont détections LSASS et certutil) et les règles Sigma ont servi de source unique de vérité pour rédiger les livrables de clôture.

Aucun nouvel incident n’a été joué : le travail porte sur la **formalisation** de l’incident simulé du **Jour 7** et de la chaîne de preuves construite aux Jours 4–9, complétée par le phishing (J10), le malware ELF/Mirai (J11), la supervision réseau (J12) et la CTI (J13).

---

### Rapport technique d’incident

Rédaction d’un rapport structuré couvrant :

- résumé exécutif ;
- périmètre ;
- timeline multi-jours ;
- détails techniques (Atomic, Sysmon, Sigma/Zircolite, KAPE, Volatility, CTI Mirai, Suricata) ;
- tableau des IOC ;
- impact ;
- cause racine ;
- recommandations ;
- références d’artefacts.

Fichier : `Jour14_Rapport_Technique_Incident.md`

Points saillants repris du lab :

- Chaîne **T1204.002 → T1059.001 → T1547.001**, avec lien fort vers **T1003.001** (ProcDump / LSASS).
- Détections **HIGH** Zircolite sur accès `lsass.exe` avec `GrantedAccess = 0x1FFFFF` (source `procdump.exe` / `procdump64.exe`, hôte `DESKTOP-33KBPU8`).
- **Patient Zero** (Jour 9) : PowerShell / `Invoke-AtomicTest`.
- Artefact résiduel `C:\Windows\Temp\lsass_dump.dmp` (Jour 5) découvert à la récupération du Jour 7.
- Hash ELF **connu Mirai** (enrichissement Jour 13 / VirusTotal).

---

### Résumé exécutif (une page)

Production d’une version **direction / RSSI** : impact lab vs risque théorique en production, messages clés sans jargon technique excessif, statut de clôture pédagogique.

Fichier : `Jour14_Resume_Executif_Incident.md`

---

### Playbook SOC

Construction d’un playbook **Credential Dumping (T1003.001)** et chaîne associée, avec :

- déclencheurs d’alerte ;
- arbre de décision (triage → confinement → collecte → analyse → éradication → récupération → lessons learned) ;
- checklists par phase ;
- critères de clôture ;
- ancrage explicite sur les exercices du parcours.

Fichier : `playbooks/Credential_Dumping_Playbook.md`

---

### Registre IOC et règles de détection

- Tableur CTI : `Jour13_IOC_Threat_Intelligence.xlsx` (inventaire, note tactique/stratégique, légende).
- Règles Sigma du lab : accès LSASS (brute + tunée Defender), usage suspect de `certutil` (LOLBin / T1105), documentées dans les carnets J5–J6 et les JSON Zircolite.

---

### Structure type portfolio / dépôt

Préparation de l’organisation logique des livrables pour publication GitHub :

```text
SOC-Lab/
├── README.md
├── sigma-rules/
├── reports/
├── playbooks/
├── iocs/
├── notebooks/          # carnets Jour 1 à 14
└── artifacts/
```

Le détail README et l’arbre complet peuvent être poussés tels quels dans le dépôt du parcours.

---

## Comment ça s’est passé

Le Jour 14 est une journée de **synthèse**. La qualité des carnets antérieurs a limité le besoin de rejouer des collectes : timeline, IOC, sorties d’outils et enseignements étaient déjà traçables.

Le fil conducteur retenu pour le rapport d’incident est celui du **Jour 7**, parce qu’il matérialise le cycle IR complet sur une chaîne ATT&CK, tout en s’appuyant sur la détection Sigma (Jour 5), la forensique (Jours 8–9) et l’enrichissement CTI (Jour 13).

L’absence d’IP/domaine C2 publics dans le binaire Mirai du Jour 11 a été traitée comme une **information d’enquête** (limites de l’analyse statique), et non comme un manque de documentation.

---

## ✅ Points validés aujourd’hui

- [x] Rapport technique d’incident rédigé (structure demandée)
- [x] Résumé exécutif d’une page produit (sans jargon)
- [x] Playbook credential dumping / chaîne associée rédigé
- [x] IOC centralisés (référence tableur Jour 13)
- [x] Liens avec Sigma, Sysmon, KAPE, Volatility, Suricata et CTI documentés
- [x] Mention explicite de **jour final** du parcours
- [x] Base d’arborescence portfolio / GitHub définie

---

## 📌 Notes pour la suite (après le parcours)

- Publier le dépôt avec carnets, règles Sigma, playbook, rapports et tableur IOC.
- Ne pas versionner de gros binaires (`.raw`, `.vmdk`, `.pcap` complets) : conserver hashes, extraits et captures documentées.
- Réutiliser le playbook comme modèle pour d’autres familles d’alertes (phishing, LOLBin téléchargement, IDS).
- En production, tout hash nouveau doit repasser par enrichissement VT / bases CTI avant classification définitive.

---

## 🏁 Bilan du parcours

Du **Jour 1** (lab isolé) au **Jour 14** (**jour final**), la chaîne SOC a été parcourue de bout en bout :

**télémétrie → détection (Sigma) → hunting → simulation IR → forensique disque/mémoire → phishing → malware → IDS/réseau → threat intelligence → documentation & playbook.**

Le livrable de ce jour final formalise l’incident de référence et les modes opératoires réutilisables pour un poste d’analyste SOC.
EOF
