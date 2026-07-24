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
| 1 | Un **home SOC lab** fonctionnel (VMs victimes Windows/Linux + Kali attaquant + SIEM Wazuh/Security Onion), isolé de ton réseau réel | Tu peux monter un environnement monitoré à partir de zéro |
| 2 | Des **règles de détection Sigma** personnalisées, tunées et mappées sur des techniques MITRE ATT&CK | Tu peux passer de « aucune visibilité » à « détecté et mappé » |
| 3 | Un **incident entièrement documenté** — trié (triage), contenu (containment), investigué (dump mémoire, artefacts disque, timeline, IOCs) | Tu peux exécuter le cycle IR complet, pas juste en parler |
| 4 | Deux comptes-rendus d'incident (technique + exécutif) et un **playbook** réutilisable | Tu peux communiquer aussi bien avec des ingénieurs qu'avec du management |
| 5 | Un **repo GitHub** avec tes règles, ton carnet de notes, ton rapport et ton playbook | Une pièce concrète à montrer en entretien |

Ce n'est pas juste « j'ai appris quelques outils » — c'est un lab vivant et démontrable, plus une étude de cas soignée qui prouve que tu peux aller des logs bruts jusqu'à un rapport d'incident écrit, seul(e).

---

## 🧰 Matériel & Budget

| Élément | Nécessaire ? | Remarques |
|---|---|---|
| **PC / machine hôte** | Requis | 16 Go de RAM minimum (32 Go plus confortable), 250 Go+ de stockage libre. Fait tourner l'hyperviseur de ton choix — ESXi, VirtualBox ou Proxmox conviennent tous pour ce programme. Specs plus faibles ? Voir le *Mode Low-Spec* plus bas. |
| **Router + Switch** | Optionnel | Utile uniquement si tu veux segmenter physiquement le lab de ton LAN domestique. Pas requis — voir *Networking, Pas Besoin d'un Boîtier Firewall* ci-dessous. |
| **Tout le reste** | Gratuit | 100% logiciels open-source (voir ci-dessous). |

---

## 🔌 Réseau — Pas Besoin d'un Boîtier Firewall

Les premières versions de ce plan incluaient pfSense comme routeur/firewall virtuel pour le réalisme. **C'est optionnel et ça a été retiré du parcours principal** — ça ajoute une vraie charge de ressources et de configuration sur ESXi/l'hyperviseur pour zéro élément de syllabus réellement requis. Chaque jour ci-dessous fonctionne sans.

**Ce qu'il faut faire à la place :**
- Créer un **réseau interne isolé** unique pour ton lab — sur ESXi c'est un vSwitch/port group **sans uplink physique** (aucune carte réseau physique rattachée) ; sur VirtualBox utilise le mode "Internal Network" ; sur Proxmox utilise un bridge Linux sans interface physique rattachée.
- Mettre chaque VM (victime Windows, victime Ubuntu, attaquant Kali, SIEM) sur ce même réseau isolé avec des IPs privées statiques (ex : `10.10.10.10/24`, `.20`, `.30`, `.40`).
- Tu auras besoin d'un accès internet bref pendant la mise en place (mises à jour OS, installation des paquets Wazuh, téléchargement d'Atomic Red Team, des configs Sysmon, des outils Sigma). Deux façons simples de gérer ça sans VM firewall :
  - Rattacher temporairement une VM à un réseau NAT/bridged pour télécharger ce dont tu as besoin, puis remettre sa carte réseau virtuelle sur le réseau interne isolé avant de lancer la moindre simulation d'attaque.
  - Ou garder un **port group "staging"** séparé avec accès internet uniquement pour les téléchargements, et ne déplacer les VMs vers le segment isolé qu'une fois entièrement patchées et provisionnées.
- **Prends un snapshot propre de chaque VM immédiatement après cette mise en place initiale.** C'est ton filet de sécurité le plus important pour les deux semaines à venir — tu voudras revenir en arrière après le jour d'analyse de malware et après la simulation d'incident.
- Si plus tard tu veux quand même la sensation d'"infra réelle" d'un routeur/firewall virtuel dans le chemin du trafic, **OPNsense** est plus léger et souvent plus facile à installer que pfSense, et peut être ajouté comme amélioration optionnelle une fois le lab de base stable — jamais comme bloqueur du Jour 1.

---

## 🛠️ Outils Utilisés (tous gratuits)

- **Hyperviseur :** ESXi, VirtualBox ou Proxmox — celui que tu arrives à faire tourner de manière fiable
- **SIEM :** Wazuh (plus léger, recommandé si c'est ton premier SIEM) ou Security Onion (plus lourd, inclut Zeek/Suricata/mapping ATT&CK intégré)
- **Visibilité endpoint :** Sysmon (Windows) avec une config communautaire (ex : celle de SwiftOnSecurity ou `sysmon-modular` d'Olaf Hartong), auditd (Linux)
- **Règles de détection :** Sigma (+ `sigma-cli`/pySigma pour la conversion)
- **Mapping ATT&CK :** MITRE ATT&CK Navigator (web, gratuit)
- **Simulation d'attaque :** Atomic Red Team (module PowerShell `Invoke-AtomicRedTeam`)
- **Forensics :** KAPE, Autopsy, Volatility 3, Magnet RAM Capture ou DumpIt (acquisition mémoire), FTK Imager Lite (imagerie disque)
- **Analyse réseau :** Wireshark, Zeek, Suricata (avec le ruleset gratuit ET Open)
- **Analyse de malware :** PEStudio, `strings`/`sha256sum`, Process Monitor/Process Explorer/Autoruns, VM sandbox isolée
- **Threat intel/OSINT :** VirusTotal, AbuseIPDB, AlienVault OTX
- **Gestion de cas :** TheHive (optionnel) ou un log markdown structuré
- **Documentation :** Obsidian / carnet SOC markdown

---

## 🗺️ Architecture de Base du Lab

```
[VM Attaquante : Kali]  ─┐
[VM Victime : Windows]   ─┼──  vSwitch interne isolé / port group (sans uplink physique)
[VM Victime : Ubuntu]    ─┤
[VM SIEM : Wazuh]        ─┘
```

Aucun routeur virtuel requis pour ce parcours — toutes les VMs sont sur un seul segment réseau isolé et plat. IPs statiques, pas d'internet une fois le provisionnement terminé.

---

## 📅 Le Parcours en 14 Jours

<details>
<summary><strong>Jour 1 — Fondations du Lab</strong></summary>

- Installer/confirmer que ton hyperviseur est stable (ESXi/VirtualBox/Proxmox).
- Créer le réseau interne isolé (vSwitch/port group sans uplink, ou mode "Internal Network" sur VirtualBox).
- Provisionner trois VMs sur un segment temporairement connecté à internet : **Windows 10/11**, **Ubuntu Server 22.04**, **Kali Linux**. Les patcher entièrement.
- Assigner des IPs statiques une fois provisionnées (ex : `10.10.10.10` Windows, `.20` Ubuntu, `.30` Kali), puis déplacer les trois cartes réseau virtuelles vers le segment isolé.
- Vérifier la connectivité : `ping` entre les trois VMs ; confirmer qu'aucune VM ne peut atteindre le vrai internet depuis le segment isolé.
- **Snapshot chaque VM maintenant** — nomme-le clairement, ex : `clean-baseline-day1`.
- Dessiner ton diagramme réseau (IPs, rôles des VMs) dans ton carnet SOC.

**Couvre :** bases de l'architecture SOC, fondamentaux réseau
</details>

<details>
<summary><strong>Jour 2 — Mise en Place du SIEM</strong></summary>

- Déployer Wazuh : soit l'OVA officielle Wazuh (si ton hyperviseur supporte l'import OVA), soit une installation manuelle tout-en-un sur la VM Ubuntu (script quickstart `wazuh-install.sh` — nécessite le segment temporairement connecté à internet).
- Installer l'agent Wazuh sur la VM Windows et sur une (seconde) VM Ubuntu victime ; les enregistrer auprès du manager Wazuh.
- Installer **Sysmon** sur la VM Windows avec une config communautaire (`sysmon64.exe -i sysmonconfig.xml`) — c'est ce qui te donne une télémétrie riche (process/réseau/registre) au lieu du logging Windows par défaut.
- Configurer des règles **auditd** sur Linux pour surveiller des fichiers et syscalls clés, ex :
  ```
  -w /etc/passwd -p wa -k identity
  -w /etc/shadow -p wa -k identity
  -a always,exit -F arch=b64 -S execve -k exec
  ```
- Vérifier dans le dashboard Wazuh : **Agents → les deux affichent "Active"** ; l'onglet **Discover** montre des champs structurés (ex : `data.win.eventdata.image`), pas du texte brut non parsé.
- Remettre les deux VMs victimes sur le segment isolé une fois les agents confirmés en train de reporter.

**Couvre :** fondamentaux SIEM, sources de logs, normalisation des données
</details>

<details>
<summary><strong>Jour 3 — Requêtes & Dashboards</strong></summary>

- Apprendre la syntaxe de la barre de requête Wazuh (style Lucene) avec ces requêtes de départ :
  - Logons échoués : `data.win.system.eventID:4625`
  - Logons réussis : `data.win.system.eventID:4624`
  - Création de process (Sysmon Event ID 1) : `data.win.system.eventID:1`
  - Connexion réseau (Sysmon Event ID 3) : `data.win.system.eventID:3`
- Construire **3 dashboards** dans la section Visualize de Wazuh : (1) logons échoués dans le temps par source, (2) top créations de process par image parent/enfant, (3) timeline des connexions réseau sortantes.
- Sauvegarder chaque recherche avec un nom clair (`auth-failures`, `proc-creation-all`, `netconn-all`) pour pouvoir les réutiliser pendant le hunting plus tard.

**Couvre :** requêtes SIEM de base, dashboards & visualisation
</details>

<details>
<summary><strong>Jour 4 — Mapping MITRE ATT&CK</strong></summary>

- Installer le module PowerShell `Invoke-AtomicRedTeam` sur la VM Windows victime.
- Lancer ces tests précis un par un, en vérifiant Wazuh après chacun :
  - `Invoke-AtomicTest T1059.001 -TestNumbers 1` (exécution PowerShell)
  - `Invoke-AtomicTest T1003.001 -TestNumbers 1` (simulation de credential dumping LSASS)
  - `Invoke-AtomicTest T1547.001 -TestNumbers 1` (persistence via Registry Run Key)
- Pour chaque test, retrouver le/les event(s) correspondant(s) dans Wazuh (Sysmon Event ID 1 pour la création de process, Event ID 10 pour l'accès process sur le test LSASS, Event ID 13 pour la modification de valeur de registre sur le test de persistence).
- Ouvrir le **MITRE ATT&CK Navigator** (mitre-attack.github.io/attack-navigator), créer un nouveau layer, et colorer chaque technique testée : vert = détectée, rouge = aucune visibilité.

**Couvre :** framework ATT&CK, mapping de techniques, identification des blind spots
</details>

<details>
<summary><strong>Jour 5 — Création de Règles de Détection</strong></summary>

- Écrire une règle Sigma pour chaque technique rouge (non détectée) du Jour 4. Exemple — détecter un accès LSASS cohérent avec du credential dumping :
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
- La convertir au format de règle Wazuh (soit manuellement dans `/var/ossec/etc/rules/local_rules.xml`, soit via `sigma-cli` avec un backend Wazuh/OpenSearch si disponible).
- Relancer le test Atomic correspondant du Jour 4 et confirmer que la nouvelle alerte se déclenche au niveau attendu.
- **Tuner une règle** : trouver une action légitime déclenchant un faux positif (ex : un antivirus accédant aussi à LSASS) et ajouter une condition d'exclusion.

**Couvre :** détection engineering, baselining, tuning
</details>

<details>
<summary><strong>Jour 6 — Threat Hunting</strong></summary>

- Hypothèse 1 : *« Un attaquant utilise des LOLBins pour télécharger des fichiers. »* Requête de hunt : chercher `certutil.exe` avec `urlcache` ou `-decode` dans la ligne de commande.
- Hypothèse 2 : *« Il y a de l'activité PowerShell en dehors des horaires normaux. »* Requête de hunt : events de création de process pour `powershell.exe`, filtrés en dehors de 09h00–17h00.
- Utiliser un template de log de hunt structuré pour chacune : **Hypothèse → Sources de données utilisées → Requête lancée → Découvertes → Verdict (confirmé/écarté)**.
- Transformer au moins une découverte confirmée en nouvelle règle de détection Sigma pour qu'elle soit captée automatiquement la prochaine fois.

**Couvre :** hunting basé sur hypothèses, exécution de hunt, hunting basé sur IOC
</details>

<details>
<summary><strong>Jour 7 — Simulation d'Incident & IR</strong></summary>

- Lancer une séquence chaînée pour simuler une vraie intrusion : `T1204.002` (l'utilisateur ouvre un document malveillant, simulé manuellement) → `T1059.001` (exécution PowerShell) → `T1547.001` (persistence via Registry Run Key).
- Pratiquer le cycle IR complet dessus :
  - **Détection** : une alerte SIEM se déclenche grâce à tes règles du Jour 5.
  - **Triage** : assigner une sévérité via une matrice simple (criticité de l'asset × impact de la technique).
  - **Containment** : isoler la VM — détacher sa vNIC ou déplacer son port group vers un segment "quarantaine" sans connectivité.
  - **Eradication** : supprimer la clé de registre de persistence, tuer le process malveillant.
  - **Recovery** : restaurer depuis ton snapshot du Jour 1, ou vérifier manuellement que le système est propre.
  - **Lessons Learned** : noter ce qui était facile/difficile à détecter et pourquoi.
- Tout logger avec un template de ticket simple : ID d'incident, timestamp, source de détection, sévérité, actions menées, qui/quand.

**Couvre :** cycle IR complet, triage d'alertes, containment
</details>

<details>
<summary><strong>Jour 8 — Collecte de Preuves & Forensics Windows</strong></summary>

- **Avant** de nettoyer l'incident du Jour 7 : capturer la mémoire en premier (les données volatiles d'abord) avec **Magnet RAM Capture** ou **DumpIt**.
- Puis capturer une image disque avec **FTK Imager Lite**, ou exporter directement le disque virtuel de la VM puisqu'on est en lab.
- Lancer **KAPE** pour un balayage complet des artefacts en une commande :
  ```
  kape.exe --tsource C: --target KapeTriage --tdest C:\out
  ```
  Ça extrait Prefetch, Event Logs, ruches de registre, MFT, USN Journal, Amcache et ShimCache en une seule passe.
- Charger les artefacts collectés dans **Autopsy** : créer un nouveau cas, ajouter l'image disque/les fichiers logiques, et explorer les modules Registry Explorer et timeline.
- Vérifier spécifiquement : les ruches `SYSTEM`/`SOFTWARE` pour les clés Run, `Amcache.hve` pour les preuves d'exécution, et les fichiers `.pf` Prefetch pour les timestamps d'exécution.

**Couvre :** collecte de preuves, chain of custody, forensics Windows
</details>

<details>
<summary><strong>Jour 9 — Forensics Mémoire & Analyse de Timeline</strong></summary>

- Analyser le dump mémoire du Jour 8 avec **Volatility 3** :
  - `vol -f memdump.raw windows.pslist` et `windows.pstree` — liste des process/parenté
  - `windows.netscan` — connexions réseau au moment de la capture
  - `windows.malfind` — détecter une éventuelle injection de code
  - `windows.cmdline` — lignes de commande des process en cours
- Recouper la chaîne de process `powershell.exe` du Jour 7 dans ces résultats.
- Construire une timeline simple en combinant les CSV exportés de KAPE avec les résultats horodatés de Volatility dans un tableur, triés chronologiquement.
- Identifier l'événement **patient zéro** (la première action malveillante) et le scope complet de ce qu'il a touché.

**Couvre :** analyse mémoire, analyse de timeline
</details>

<details>
<summary><strong>Jour 10 — Analyse de Phishing</strong></summary>

- Construire toi-même un email de phishing test réaliste — soit un fichier `.eml` brut fabriqué manuellement, soit via un sandbox mail gratuit (ex : Mailtrap/Mailhog). Désaligner volontairement le `From`/`Return-Path` et faire échouer le SPF.
- Lire les headers manuellement : regarder le header `Authentication-Results` pour `spf=fail`/`dkim=fail`, et comparer l'expéditeur affiché avec le serveur d'origine réel dans les headers `Received`.
- Defanger avant de documenter quoi que ce soit : `http` → `hxxp`, `.` → `[.]`.
- Vérifier la réputation de l'URL/du domaine (fictif/test) via le scan URL de VirusTotal ou urlscan.io — sans jamais la visiter directement dans un navigateur.

**Couvre :** analyse de headers email, détection de contenu malveillant, reconnaissance d'ingénierie sociale
</details>

<details>
<summary><strong>Jour 11 — Bases de l'Analyse de Malware</strong></summary>

- Récupérer un échantillon connu et labellisé depuis **MalwareBazaar** (compte/clé API gratuits requis). Le manipuler **uniquement** dans une VM entièrement isolée et offline, restaurée depuis un snapshot propre — pas de dossiers partagés, pas de réseau bridged/NAT.
- **Analyse statique** : `sha256sum sample`, `strings -n 8 sample | less`, puis le charger dans **PEStudio** pour vérifier les imports, les sections et l'entropie (une entropie élevée sur toutes les sections est un indicateur de packer).
- **Analyse dynamique (optionnel, plus avancé)** : détoner dans la VM isolée, surveiller avec **Process Monitor**/**Process Explorer**, vérifier la persistence avec **Autoruns**, et capturer tout trafic généré avec Wireshark sur le segment isolé (il n'ira nulle part — c'est normal, tu observes juste le comportement).
- Extraire les IOCs : hash du fichier, chemins de fichiers déposés, noms de mutex, et toute IP/domaine contacté (sinkholé/non routable).

> ⚠️ Uniquement dans une VM entièrement isolée et jetable. Restaure ton snapshot immédiatement après ce jour.

**Couvre :** analyse de malware statique/dynamique, extraction d'IOCs
</details>

<details>
<summary><strong>Jour 12 — Network Security Monitoring</strong></summary>

- Activer le **mode promiscuous** sur ton port group isolé (sur ESXi : Security policy → Promiscuous Mode → Accept) pour qu'une VM de monitoring puisse voir tout le trafic du lab.
- Déployer **Suricata** ou **Zeek** sur une VM de monitoring dédiée rattachée à ce segment, en mode IDS avec le ruleset gratuit **ET Open**.
- Relancer un de tes tests Atomic précédents (Jour 4 ou 7) tout en capturant simultanément avec **Wireshark**.
- Corréler toute alerte Suricata/Zeek avec les paquets exacts dans la capture Wireshark en utilisant le timestamp et le 5-tuple de l'alerte (IP/port src/dst, protocole).

**Couvre :** analyse de trafic réseau, IDS, artefacts réseau
</details>

<details>
<summary><strong>Jour 13 — Enrichissement Threat Intel</strong></summary>

- Compiler tous les IOCs collectés sur l'ensemble du projet dans un seul tableur : hash / IP / domaine / jour d'origine / technique associée.
- Vérifier chacun : **VirusTotal** (réputation hash/URL), **AbuseIPDB** (score de réputation IP), **AlienVault OTX** (recherche de pulse/contexte).
- Écrire une courte note intel distinguant l'**intelligence tactique** (ce hash/IP précis est connu-malveillant ou non) de l'**intelligence stratégique** (le pattern global — ex : « usage de LOLBins + persistence via registre » — correspond à un comportement de malware commodity, pas à un APT ciblé).

**Couvre :** fondamentaux de la threat intelligence, OSINT
</details>

<details>
<summary><strong>Jour 14 — Documentation & Playbook</strong></summary>

- Rédiger proprement le rapport de l'incident du Jour 7 avec cette structure : Résumé Exécutif, Timeline, Détails Techniques, Tableau des IOCs, Impact, Cause Racine, Recommandations.
- Produire **deux versions** : un rapport technique complet (extraits de logs, références aux artefacts, sorties Volatility/KAPE) et un résumé exécutif d'1 page (impact business et risque, sans jargon).
- Construire un **playbook** pour ce type d'incident : un arbre de décision — Critères de déclenchement d'alerte → Étapes de triage → Étapes de containment → Checklist d'eradication → Validation de recovery → Clôture.
- Pousser le tout sur un repo GitHub avec une structure claire, ex :
  ```
  /sigma-rules
  /soc-notebook
  /reports
  /playbooks
  ```

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
