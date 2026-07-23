<div align="center">
  <img src="https://github.com/Selimjerbi66/CyberAtaraxia-Suite/blob/main/cyberataraxia_new_logo.png?raw=true" width="180" alt="CyberAtaraxia Logo"/>
  <h1>CyberAtaraxia Careers — SOC Analyst</h1>

  <p>
    Un parcours gratuit et pratique de 2 semaines pour passer de débutant en cybersécurité aux fondamentaux d'un SOC Analyst prêt pour l'emploi — un projet de la <strong>CyberAtaraxia Suite</strong> par <strong>Selim JERBI</strong>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Status-In%20Development-blue?style=for-the-badge" />
    <img src="https://img.shields.io/badge/License-Open%20Source-green?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Coût-100%25%20Outils%20Gratuits-success?style=for-the-badge" />
    <a href="https://github.com/Selimjerbi66">
      <img src="https://img.shields.io/badge/Fait%20par-Selim%20JERBI-blueviolet?style=for-the-badge" />
    </a>
  </p>
  <p>
    🇬🇧 <a href="README.md">English</a> &nbsp;|&nbsp; 🇫🇷 <a href="README_FR.md">Français</a>
  </p>
</div>

---

## 🧭 À Propos de ce Career Path

**CyberAtaraxia Careers** est une nouvelle ligne au sein de la CyberAtaraxia Suite : des **parcours de carrière** guidés et pratiques, plutôt que des outils isolés — chacun conçu pour amener un débutant de zéro à un ensemble de compétences démontrable et présentable en portfolio, en utilisant uniquement des ressources gratuites et open-source.

**SOC Analyst** est la première entrée de cette ligne. Ce n'est pas un cours qu'on lit — c'est un projet qu'on *exécute* : tu construis un petit SOC à la maison, tu attaques ton propre lab en toute sécurité, puis tu suis cette attaque à travers toutes les étapes qu'un vrai analyste SOC Tier 1/2 traverserait — détection, threat hunting, incident response, forensics, analyse de malware/phishing, et threat intelligence — pour finir sur un vrai rapport d'incident écrit.

Aucune expérience préalable en cybersécurité requise. Aucun outil payant. Aucun abonnement cloud.

---

## 🎯 Objectif Principal

Passer de débutant à quelqu'un ayant une **pratique concrète sur (presque) toute la chaîne blue team** — pas seulement lire sur le SIEM, le framework MITRE ATT&CK, la forensics et l'IR, mais réellement pratiquer chaque étape face à de vraies attaques (simulées et contenues en toute sécurité), de bout en bout :

> détecter quelque chose → le hunt (traquer) → y répondre → l'investiguer → le documenter

Tout est construit autour d'**une seule histoire continue** plutôt que des exercices déconnectés : tu construis un lab, tu attaques ta propre machine (via des simulations Atomic Red Team), et tu suis cette attaque à travers toutes les étapes qu'un vrai analyste SOC traverserait.

---

## 🏁 Ce Que Tu Repars Avec

| # | Livrable | Ce que ça prouve |
|---|---|---|
| 1 | Un **home SOC lab** fonctionnel (pfSense + VMs victimes Windows/Linux + Kali attaquant + SIEM Wazuh/Security Onion), isolé de ton réseau réel | Tu peux monter un environnement monitoré à partir de zéro |
| 2 | Des **règles de détection Sigma** personnalisées, tunées et mappées sur des techniques MITRE ATT&CK | Tu peux passer de « aucune visibilité » à « détecté et mappé » |
| 3 | Un **incident entièrement documenté** — trié (triage), contenu (containment), investigué (dump mémoire, artefacts disque, timeline, IOCs) | Tu peux exécuter le cycle IR complet, pas juste en parler |
| 4 | Deux comptes-rendus d'incident (technique + exécutif) et un **playbook** réutilisable | Tu peux communiquer aussi bien avec des ingénieurs qu'avec du management |
| 5 | Un **repo GitHub** avec tes règles, ton carnet de notes, ton rapport et ton playbook | Une pièce concrète à montrer en entretien |

Ce n'est pas juste « j'ai appris quelques outils » — c'est un lab vivant et démontrable, plus une étude de cas soignée qui prouve que tu peux aller des logs bruts jusqu'à un rapport d'incident écrit, seul(e).

---

## 🧰 Matériel & Budget

| Élément | Nécessaire ? | Remarques |
|---|---|---|
| **PC** | Requis | 16 Go de RAM minimum (32 Go plus confortable), 250 Go+ de SSD libre. Fait tourner l'hyperviseur + toutes les VMs. Specs plus faibles ? Voir le *Mode Low-Spec* plus bas. |
| **Router + Switch** | Optionnel | Pour segmenter physiquement le lab de ton LAN domestique (VLAN/sous-réseau). Tu peux t'en passer et tout faire avec des switches virtuels — coût zéro, risque zéro. |
| **Tout le reste** | Gratuit | 100% logiciels open-source (voir ci-dessous). |

---

## 🛠️ Outils Utilisés (tous gratuits)

- **Hyperviseur :** VirtualBox (simple) ou Proxmox (plus "infra réelle")
- **Router/Firewall :** pfSense
- **SIEM :** Wazuh (plus léger) ou Security Onion (plus lourd, inclut Zeek/Suricata/mapping ATT&CK)
- **Visibilité endpoint :** Sysmon (Windows), auditd (Linux)
- **Règles de détection :** Sigma
- **Mapping ATT&CK :** MITRE ATT&CK Navigator
- **Simulation d'attaque :** Atomic Red Team
- **Forensics :** Autopsy, KAPE, Volatility 3
- **Analyse réseau :** Wireshark, Zeek, Suricata
- **Analyse de malware :** PEStudio, strings/exiftool, VM sandbox isolée
- **Threat intel/OSINT :** VirusTotal, AbuseIPDB, AlienVault OTX
- **Gestion de cas :** TheHive (optionnel) ou un log markdown structuré
- **Documentation :** Obsidian / carnet SOC markdown

---

## 🗺️ Architecture de Base du Lab

```
[VM Attaquante : Kali]  --->  [Routeur/Firewall Virtuel : pfSense]  --->  [VMs Victimes : Windows 10/11, Ubuntu]
                                        │
                          [VM SIEM : Wazuh ou Security Onion]  <── collecte logs/agents de partout
```

Tous les réseaux sont virtuels/isolés (mode host-only ou internal network) — étape critique avant que quoi que ce soit de malveillant ne touche une VM.

---

## 📅 Le Parcours en 14 Jours

<details>
<summary><strong>Jour 1 — Fondations du Lab</strong></summary>

Installer l'hyperviseur, mettre en place les réseaux internes isolés, déployer pfSense, les VMs victimes Windows + Ubuntu, la VM attaquante Kali.
**Couvre :** bases de l'architecture SOC, fondamentaux réseau
</details>

<details>
<summary><strong>Jour 2 — Mise en Place du SIEM</strong></summary>

Déployer Wazuh/Security Onion, installer les agents/Sysmon/auditd sur les victimes, vérifier le flux de logs normalisés.
**Couvre :** fondamentaux SIEM, sources de logs, normalisation des données
</details>

<details>
<summary><strong>Jour 3 — Requêtes & Dashboards</strong></summary>

Écrire des requêtes de base (KQL/langage de requête Wazuh), construire 2-3 dashboards, sauvegarder des recherches récurrentes.
**Couvre :** requêtes SIEM de base, dashboards & visualisation
</details>

<details>
<summary><strong>Jour 4 — Mapping MITRE ATT&CK</strong></summary>

Lancer 5 à 10 tests Atomic Red Team, mapper les logs résultants sur ATT&CK Navigator, repérer les blind spots.
**Couvre :** framework ATT&CK, mapping de techniques, identification des blind spots
</details>

<details>
<summary><strong>Jour 5 — Création de Règles de Détection</strong></summary>

Écrire et importer 3 à 5 règles Sigma pour les techniques non détectées, retester, tuner un faux positif.
**Couvre :** détection engineering, baselining, tuning
</details>

<details>
<summary><strong>Jour 6 — Threat Hunting</strong></summary>

Formuler 2 hypothèses, hunt de manière proactive via des requêtes SIEM, transformer une découverte en nouvelle règle.
**Couvre :** hunting basé sur hypothèses, exécution de hunt, hunting basé sur IOC
</details>

<details>
<summary><strong>Jour 7 — Simulation d'Incident & IR</strong></summary>

Lancer une attack chain multi-étapes, la trier (triage) comme un incident en direct, la contenir (containment), documenter chaque étape du cycle IR.
**Couvre :** cycle IR complet, triage d'alertes, containment
</details>

<details>
<summary><strong>Jour 8 — Collecte de Preuves & Forensics Windows</strong></summary>

Collecter un dump mémoire puis une image disque, extraire des artefacts avec KAPE, analyser avec Autopsy.
**Couvre :** collecte de preuves, chain of custody, forensics Windows
</details>

<details>
<summary><strong>Jour 9 — Forensics Mémoire & Analyse de Timeline</strong></summary>

Analyser le dump mémoire avec Volatility 3, construire une timeline multi-sources, identifier le patient zéro.
**Couvre :** analyse mémoire, analyse de timeline
</details>

<details>
<summary><strong>Jour 10 — Analyse de Phishing</strong></summary>

Fabriquer un échantillon de phishing réaliste, analyser les headers (SPF/DKIM/DMARC), vérifier la réputation d'une URL en toute sécurité.
**Couvre :** analyse de headers email, détection de contenu malveillant, reconnaissance d'ingénierie sociale
</details>

<details>
<summary><strong>Jour 11 — Bases de l'Analyse de Malware</strong></summary>

Analyse statique (hash, strings, structure PE) et détonation dynamique contenue en option ; extraction d'IOCs.

> ⚠️ Uniquement dans une VM entièrement isolée et jetable (disposable) — sans dossiers partagés, sans pont internet.

**Couvre :** analyse de malware statique/dynamique, extraction d'IOCs
</details>

<details>
<summary><strong>Jour 12 — Network Security Monitoring</strong></summary>

Capturer du trafic avec Wireshark/Zeek/Suricata, corréler une alerte IDS avec le PCAP.
**Couvre :** analyse de trafic réseau, IDS, artefacts réseau
</details>

<details>
<summary><strong>Jour 13 — Enrichissement Threat Intel</strong></summary>

Passer les IOCs collectés dans VirusTotal/AbuseIPDB/OTX, écrire une note tactique vs. stratégique.
**Couvre :** fondamentaux de la threat intelligence, OSINT
</details>

<details>
<summary><strong>Jour 14 — Documentation & Playbook</strong></summary>

Rédiger les rapports d'incident technique + exécutif, construire un playbook réutilisable, publier le tout sur GitHub.
**Couvre :** rapports d'incident, création de playbook, documentation
</details>

---

## 🐢 Mode Low-Spec

8 Go de RAM ou moins ? Lance une VM à la fois : attaque la VM Windows, exporte les logs, éteins-la, puis démarre la VM SIEM pour ingérer et analyser. Plus lent, mais chaque exercice reste faisable.

---

## ⚠️ Note sur le Périmètre

Ce parcours ne couvre pas le cloud security monitoring (logging AWS/Azure/GCP) ni le SOAR complet — cela nécessite des comptes cloud payants ou des outils enterprise. Une **Phase 2** utilisant le logging free-tier AWS/Azure pourrait suivre comme future entrée CyberAtaraxia Careers.

---

## 👤 À Propos du Développeur

- 🎓 Étudiant en Ingénierie Réseau à **Polytech Dijon**, spécialisation Cybersécurité
- 🔵 Blue Teamer | Auditeur SI | Administrateur Réseau
- 🏢 Stagiaire Cybersécurité chez **Axem Belgium**
- 🌱 Actuellement en préparation **Blue Team L1 · ISO 27001/2022 · CCNA**

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
  <sub>CyberAtaraxia Careers — Open Source · Créé avec ambition par Selim JERBI</sub>
</div>
