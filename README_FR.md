<div align="center">
  <img src="https://github.com/Selimjerbi66/CyberAtaraxia-Suite/blob/main/cyberataraxia_new_logo.png?raw=true" width="180" alt="CyberAtaraxia Logo"/>
  <h1>CyberAtaraxia Careers — SOC Analyst</h1>

  <p>
    Un programme gratuit et pratique de home lab sur 2 semaines pour passer de débutant en cybersécurité à des fondamentaux de SOC Analyst prêts pour l'emploi — fait partie de la <strong>CyberAtaraxia Suite</strong> par <strong>Selim JERBI</strong>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Statut-En%20D%C3%A9veloppement-blue?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Licence-Open%20Source-green?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Co%C3%BBt-100%25%20Outils%20Gratuits-success?style=for-the-badge" />
    <a href="https://github.com/Selimjerbi66">
      <img src="https://img.shields.io/badge/Fait%20par-Selim%20JERBI-blueviolet?style=for-the-badge" />
    </a>
  </p>
  <p>
    🇬🇧 <a href="README.md">English</a> &nbsp;|&nbsp; 🇫🇷 <a href="README_FR.md">Français</a>
  </p>
</div>

---

## 🧭 À propos de ce parcours

**CyberAtaraxia Careers** est une nouvelle ligne au sein de la CyberAtaraxia Suite : des **parcours de carrière** guidés et pratiques plutôt que de simples outils isolés — chacun conçu pour amener un débutant de zéro à une compétence démontrable, prête pour un portfolio, en utilisant uniquement des ressources gratuites et open-source.

**SOC Analyst** est la première entrée de cette ligne. Ce n'est pas un cours qu'on lit — c'est un projet qu'on *exécute* : tu montes un petit home lab, tu attaques ta propre machine (via des simulations Atomic Red Team), puis tu suis cette attaque à travers toutes les étapes qu'un vrai analyste SOC Tier 1/2 traverserait — détection, chasse aux menaces, réponse à incident, forensique, analyse de malware/phishing, et threat intelligence — pour finir par un vrai rapport d'incident écrit.

Aucune expérience en cybersécurité requise. Aucun outil payant. Aucun abonnement cloud. **Aucun serveur SIEM central requis** — ce parcours utilise des outils de détection légers, basés sur des fichiers, ce qui permet de tenir tout le lab dans seulement 3 VMs.

---

## 🎯 Objectif principal

T'amener de débutant à quelqu'un ayant une **pratique concrète sur presque toute la chaîne blue team** — pas juste lire sur la détection, MITRE ATT&CK, la forensique et l'IR, mais vraiment faire chacune de ces étapes face à de vraies attaques (simulées en toute sécurité), de bout en bout :

> détecter quelque chose → chasser dessus → y répondre → l'investiguer → l'écrire

Tout est construit autour d'**une seule histoire continue** plutôt que des exercices déconnectés : tu montes un lab, tu attaques ta propre machine (via Atomic Red Team), et tu suis cette attaque à travers chaque étape qu'un vrai analyste SOC traverserait.

---

## 🏁 Ce que tu obtiendras à la fin

| # | Livrable | Ce que ça prouve |
|---|---|---|
| 1 | Un **home lab fonctionnel** (Windows + Linux + attaquant Kali) — seulement 3 VMs, isolé de ton vrai réseau | Tu sais monter un environnement surveillé à partir de rien, sans avoir besoin d'infrastructure lourde |
| 2 | Des **règles de détection Sigma** personnalisées, ajustées et mappées à des techniques MITRE ATT&CK, exécutées avec un moteur local léger | Tu sais passer de "aucune visibilité" à "détecté et mappé" sans SIEM |
| 3 | Un incident entièrement **documenté** — trié, contenu, investigué (dump mémoire, artefacts disque, timeline, IOCs) | Tu sais exécuter le cycle complet de réponse à incident, pas juste en parler |
| 4 | Deux comptes-rendus d'incident (technique + exécutif) et un **playbook** réutilisable | Tu sais communiquer aussi bien aux ingénieurs qu'au management |
| 5 | Un **dépôt GitHub** avec tes règles, ton carnet de notes, ton rapport et ton playbook | Un artefact concret à montrer en entretien |

Ce n'est pas juste "j'ai appris quelques outils" — c'est un lab vivant et démontrable plus une étude de cas aboutie prouvant que tu sais aller de logs bruts à un rapport d'incident écrit, en autonomie.

---

## 🧰 Matériel & budget

| Élément | Nécessaire ? | Notes |
|---|---|---|
| **PC / machine hôte** | Requis | 8–16 Go de RAM suffisent (16 Go confortable), **environ 80–100 Go de stockage libre suffisent** puisqu'il n'y a pas de VM serveur SIEM. Fait tourner l'hyperviseur de ton choix — ESXi, VirtualBox ou Proxmox conviennent tous. Configuration plus modeste ? Voir le *Mode Low-Spec* plus bas. |
| **Routeur + Switch** | Optionnel | Utile uniquement si tu veux physiquement segmenter le lab de ton réseau domestique. Pas nécessaire — voir *Réseau, pas besoin de pare-feu dédié* plus bas. |
| **Tout le reste** | Gratuit | 100% logiciels open-source (voir plus bas). |

---

## 🔌 Réseau — pas besoin de pare-feu dédié

Les premières versions de ce plan incluaient pfSense comme routeur/pare-feu virtuel pour plus de réalisme. **C'est optionnel et retiré du parcours principal** — ça ajoute une vraie consommation de ressources ESXi/hyperviseur et une charge de configuration pour zéro couverture obligatoire du programme. Chaque journée ci-dessous fonctionne sans.

**Ce qu'il faut faire à la place :**
- Crée un **réseau interne isolé** pour ton lab — sur ESXi c'est un vSwitch/port group **sans uplink physique** (aucune carte réseau physique attachée) ; sur VirtualBox utilise le mode "Réseau interne" ; sur Proxmox utilise un bridge Linux sans interface physique attachée.
- Place chaque VM (victime Windows, victime Ubuntu, attaquant Kali) sur ce même réseau isolé avec des IP privées statiques (ex. `10.10.10.10/24`, `.20`, `.30`).
- Tu auras besoin d'un accès internet bref pendant l'installation (mises à jour OS, installation de Sysmon/Zircolite, téléchargement d'Atomic Red Team, des règles Sigma). Deux façons simples de gérer ça sans VM pare-feu :
  - Attache temporairement une VM à un réseau NAT/pont pour télécharger ce dont tu as besoin, puis remets sa carte réseau virtuelle sur le segment interne isolé avant de lancer une simulation d'attaque.
  - Ou garde un **port group "staging" séparé** avec accès internet uniquement pour les téléchargements, et ne déplace les VMs sur le segment isolé qu'une fois entièrement patchées et provisionnées.
- **Prends un snapshot propre de chaque VM immédiatement après cette installation initiale.** C'est ton filet de sécurité le plus important pour les deux semaines à venir — tu voudras revenir en arrière après la journée d'analyse de malware et après la simulation d'incident. Pour économiser de l'espace disque, ne garde qu'**un seul snapshot actif par VM** à la fois (consolide/supprime les anciens une fois qu'une journée est documentée), et provisionne tes disques virtuels en **thin provisioning** plutôt qu'en épais.
- Si tu veux plus tard la sensation "infrastructure réelle" d'un routeur/pare-feu virtuel dans le chemin du trafic, **OPNsense** est une installation plus légère et souvent plus simple que pfSense, et peut être ajoutée comme amélioration bonus une fois le lab de base stable — jamais un blocage du Jour 1.

---

## 🛠️ Outils utilisés (tous gratuits, aucune pile serveur/agent nécessaire)

- **Hyperviseur :** ESXi, VirtualBox ou Proxmox — celui que tu arrives à faire tourner de manière fiable
- **Visibilité endpoint :** Sysmon (Windows) avec une config communautaire (ex. celle de SwiftOnSecurity ou `sysmon-modular` d'Olaf Hartong), auditd (Linux) — les logs restent locaux, exportés à la demande plutôt que streamés vers un SIEM
- **Moteur de détection (sans serveur) :** **Zircolite** — applique des règles Sigma directement sur des logs EVTX/JSON exportés depuis ta propre machine, aucun manager, aucun agent, aucun cluster d'indexation. **Chainsaw** ou **Hayabusa** sont des alternatives interchangeables si tu veux comparer les outils
- **Règles de détection :** Sigma (règles utilisées nativement par Zircolite — aucune conversion de format nécessaire)
- **Mapping ATT&CK :** MITRE ATT&CK Navigator (basé web, gratuit)
- **Simulation d'attaque :** Atomic Red Team (module PowerShell `Invoke-AtomicRedTeam`)
- **Forensique :** KAPE, Autopsy, Volatility 3, Magnet RAM Capture ou DumpIt (acquisition mémoire), FTK Imager Lite (imagerie disque)
- **Analyse réseau :** Wireshark, Zeek, Suricata (avec le ruleset gratuit ET Open) — lancés temporairement sur la VM Kali quand nécessaire, aucune VM de monitoring dédiée requise
- **Analyse de malware :** PEStudio, `strings`/`sha256sum`, Process Monitor/Process Explorer/Autoruns, une VM sandbox isolée
- **Threat intel/OSINT :** VirusTotal, AbuseIPDB, AlienVault OTX
- **Gestion de cas :** Un journal markdown structuré (TheHive retiré — encore un serveur dont tu n'as pas besoin pour ce parcours)
- **Documentation :** Obsidian / markdown simple, "carnet de notes SOC"

> 💾 **Pourquoi c'est important pour le stockage :** abandonner la pile manager/agent SIEM retire le plus gros consommateur de disque du plan original (indexation continue de logs + une VM serveur dédiée). Zircolite lui-même est un binaire unique de moins de 50 Mo, et tu ne gardes que le petit fichier JSON de sortie — pas les logs bruts exportés — une fois les résultats d'une journée passés en revue.

---

## 🗺️ Architecture centrale du lab

```
[VM Attaquant : Kali]  ─┐
[VM Victime : Windows]  ─┼──  vSwitch / port group interne isolé (aucun uplink physique)
[VM Victime : Ubuntu]   ─┘
```

Seulement 3 VMs, un seul segment réseau isolé et à plat, IP statiques, aucun internet une fois le provisionnement terminé. Aucune VM SIEM/serveur. La détection se fait **à la demande** : exporte les logs de Windows/Ubuntu après une simulation, lance Zircolite dessus (depuis n'importe laquelle des 3 VMs, ou ta machine hôte si tu l'y installes), passe en revue la sortie JSON, puis supprime l'export brut.

---

## 📅 Le parcours en 14 jours

<details>
<summary><strong>Jour 1 — Fondations du lab</strong></summary>

- Installe/confirme que ton hyperviseur est stable (ESXi/VirtualBox/Proxmox).
- Crée le réseau interne isolé (vSwitch/port group sans uplink, ou mode "Réseau interne" dans VirtualBox).
- Provisionne trois VMs sur un segment temporairement connecté à internet : **Windows 10/11**, **Ubuntu Server 22.04**, **Kali Linux**. Patch entièrement chacune. Crée leurs disques virtuels en **thin provisioning**.
- Assigne des IP statiques une fois provisionnées (ex. `10.10.10.10` Windows, `.20` Ubuntu, `.30` Kali), puis déplace les cartes réseau virtuelles des trois VMs sur le segment isolé.
- Vérifie la connectivité : `ping` entre les trois VMs ; confirme qu'aucune VM ne peut atteindre le vrai internet depuis le segment isolé.
- **Prends un snapshot de chaque VM maintenant** — étiquette-le clairement, ex. `clean-baseline-day1`. C'est ton unique snapshot actif par VM pour l'instant.
- Dessine ton diagramme réseau (IP, rôles des VMs) dans ton carnet de notes SOC.

**Couvre :** bases de l'architecture SOC, fondamentaux réseau
</details>

<details>
<summary><strong>Jour 2 — Sources de logs & mise en place du moteur de détection local</strong></summary>

- Installe **Sysmon** sur la VM Windows avec une config communautaire (`sysmon64.exe -i sysmonconfig.xml`) — c'est ce qui te donne une télémétrie riche (processus/réseau/registre) au lieu du logging Windows par défaut.
- Configure des règles **auditd** sur la VM Ubuntu pour surveiller des fichiers et syscalls clés, ex. :
  ```
  -w /etc/passwd -p wa -k identity
  -w /etc/shadow -p wa -k identity
  -a always,exit -F arch=b64 -S execve -k exec
  ```
- Limite la croissance des logs pour ne pas saturer ton disque : plafonne la taille des canaux d'Event Log Windows (`wevtutil sl <log> /ms:<octets>`, ex. 50–100 Mo par canal au lieu de la valeur par défaut) et configure la rotation auditd dans `/etc/audit/auditd.conf` (`max_log_file`, `num_logs`).
- Installe **Zircolite** (binaire unique, à télécharger depuis les releases GitHub) — aucune autre installation nécessaire, et aucun agent à déployer sur les VMs victimes.
- Fais un premier test : exporte les canaux Security/Sysmon de Windows en EVTX (`wevtutil epl Microsoft-Windows-Sysmon/Operational sysmon_export.evtx`), lance Zircolite dessus avec un ruleset Sigma par défaut, et confirme que tu obtiens une sortie JSON avec des correspondances.
- Remets les deux VMs victimes sur le segment isolé une fois la télémétrie confirmée dans les logs locaux.

**Couvre :** fondamentaux équivalents SIEM, sources de logs, normalisation des données — sans serveur SIEM
</details>

<details>
<summary><strong>Jour 3 — Requêtes manuelles & revue de logs</strong></summary>

- Apprends à interroger directement les logs exportés au lieu d'un dashboard. Sur Windows, utilise `Get-WinEvent` avec des filtres XPath, ex. :
  - Connexions échouées : `Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]"`
  - Connexions réussies : `Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]"`
  - Création de processus (Sysmon Event ID 1) : `Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]"`
  - Connexion réseau (Sysmon Event ID 3) : même log, `EventID=3`
- Sur Ubuntu, utilise `ausearch -k identity` / `ausearch -k exec` pour récupérer les événements auditd équivalents.
- Au lieu de dashboards façon Wazuh, construis **3 routines de revue sauvegardées** dans ton carnet de notes SOC : (1) connexions échouées par source, (2) créations de processus par image parent/enfant, (3) timeline des connexions sortantes — chacune étant juste une commande PowerShell/`ausearch` sauvegardée plus une note sur ce à quoi ressemble la "normalité".
- Exporte un petit échantillon de chacune en CSV (`Export-Csv`) pour avoir quelque chose de concret à réutiliser pendant la chasse plus tard, puis supprime les exports EVTX bruts pour économiser de l'espace.

**Couvre :** analyse de logs de base, équivalents de requêtes manuelles, documentation de baseline
</details>

<details>
<summary><strong>Jour 4 — Mapping MITRE ATT&CK</strong></summary>

- Installe le module PowerShell `Invoke-AtomicRedTeam` sur la victime Windows.
- Lance ces tests spécifiques un par un, en exportant le canal de log concerné et en vérifiant avec Zircolite (ou une requête manuelle `Get-WinEvent`) après chacun :
  - `Invoke-AtomicTest T1059.001 -TestNumbers 1` (exécution PowerShell)
  - `Invoke-AtomicTest T1003.001 -TestNumbers 1` (simulation de dump d'identifiants LSASS)
  - `Invoke-AtomicTest T1547.001 -TestNumbers 1` (persistance par clé de registre Run)
- Pour chaque test, trouve le(s) événement(s) correspondant(s) : Sysmon Event ID 1 pour la création de processus, Event ID 10 pour l'accès processus sur le test LSASS, Event ID 13 pour la modification de valeur de registre sur le test de persistance.
- Ouvre le **MITRE ATT&CK Navigator** (mitre-attack.github.io/attack-navigator), crée un nouveau layer, et colore chaque technique testée : vert = détecté, rouge = aucune visibilité.

**Couvre :** framework ATT&CK, mapping de techniques, identification des angles morts
</details>

<details>
<summary><strong>Jour 5 — Création de règles de détection</strong></summary>

- Écris une règle Sigma pour chaque technique rouge (non détectée) du Jour 4. Exemple — détecter un accès LSASS cohérent avec un dump d'identifiants :
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
- Dépose la règle directement dans ton dossier de règles Zircolite — **aucune conversion nécessaire**, Zircolite consomme Sigma nativement.
- Relance le test Atomic correspondant du Jour 4, exporte le log concerné, lance Zircolite, et confirme que la nouvelle correspondance apparaît au niveau attendu.
- **Ajuste une règle** : trouve une action légitime déclenchant un faux positif (ex. un antivirus accédant aussi à LSASS) et ajoute une condition d'exclusion à la règle Sigma.

**Couvre :** ingénierie de détection, baselining, ajustement
</details>

<details>
<summary><strong>Jour 6 — Chasse aux menaces (Threat Hunting)</strong></summary>

- Hypothèse 1 : *"Un attaquant utilise des LOLBins pour télécharger des fichiers."* Requête de chasse : exporte les logs de création de processus, `grep`/`Select-String` pour `certutil.exe` avec `urlcache` ou `-decode` dans la ligne de commande.
- Hypothèse 2 : *"Il y a de l'activité PowerShell en dehors des heures de travail normales."* Requête de chasse : exporte les événements de création de processus `powershell.exe`, filtre en dehors de 09h00–17h00 dans ton tableur/CSV.
- Utilise un modèle de journal de chasse structuré pour chacune : **Hypothèse → Sources de données utilisées → Requête exécutée → Résultats → Verdict (confirmé/écarté)**.
- Transforme au moins un résultat confirmé en nouvelle règle Sigma déposée dans ton ruleset Zircolite, pour qu'elle soit détectée automatiquement la prochaine fois que tu exportes et scannes des logs.

**Couvre :** chasse dirigée par hypothèses, exécution de chasse, chasse basée sur des IOC
</details>

<details>
<summary><strong>Jour 7 — Simulation d'incident & réponse à incident</strong></summary>

- Lance une séquence chaînée pour simuler une vraie intrusion : `T1204.002` (l'utilisateur ouvre un document malveillant, simulé manuellement) → `T1059.001` (exécution PowerShell) → `T1547.001` (persistance par clé de registre Run).
- Pratique le cycle complet de réponse à incident face à cette attaque :
  - **Détection** : exporte le canal de log concerné, lance Zircolite avec tes règles du Jour 5, confirme la correspondance.
  - **Triage** : assigne une sévérité avec une matrice simple (criticité de l'actif × impact de la technique).
  - **Confinement** : isole la VM — détache sa carte réseau virtuelle ou déplace son port group vers un segment "quarantaine" sans connectivité.
  - **Éradication** : supprime la clé de registre de persistance, tue le processus malveillant.
  - **Récupération** : restaure depuis le snapshot du Jour 1, ou vérifie manuellement que le système est propre.
  - **Retour d'expérience** : note ce qui était facile/difficile à détecter et pourquoi.
- Journalise tout avec un modèle de ticket simple : ID d'incident, horodatage, source de détection, sévérité, actions prises, qui/quand.

**Couvre :** cycle complet de réponse à incident, triage d'alertes, confinement
</details>

<details>
<summary><strong>Jour 8 — Collecte de preuves & forensique Windows</strong></summary>

- **Avant** de nettoyer l'incident du Jour 7 : capture d'abord la mémoire (données volatiles en premier) avec **Magnet RAM Capture** ou **DumpIt**.
- Puis capture une image disque avec **FTK Imager Lite**, ou exporte directement le disque virtuel de la VM puisque c'est un lab.
- Lance **KAPE** pour un balayage complet des artefacts en une commande :
  ```
  kape.exe --tsource C: --target KapeTriage --tdest C:\out
  ```
  Ça récupère Prefetch, Event Logs, ruches de registre, MFT, USN Journal, Amcache et ShimCache en un seul passage.
- Charge les artefacts collectés dans **Autopsy** : crée un nouveau cas, ajoute l'image disque/les fichiers logiques, et explore les modules Registry Explorer et timeline.
- Vérifie spécifiquement : les ruches `SYSTEM`/`SOFTWARE` pour les clés Run, `Amcache.hve` pour les preuves d'exécution, et les fichiers `.pf` Prefetch pour les horodatages d'exécution.
- Une fois que tu as extrait ce dont tu as besoin, déplace le dump mémoire brut et l'image disque hors du disque de la VM (disque externe ou suppression après avoir copié les résultats dans ton carnet) — ces fichiers sont volumineux et n'ont pas besoin de rester dans le lab durablement.

**Couvre :** collecte de preuves, chaîne de custody, forensique Windows
</details>

<details>
<summary><strong>Jour 9 — Forensique mémoire & analyse de timeline</strong></summary>

- Analyse le dump mémoire du Jour 8 avec **Volatility 3** :
  - `vol -f memdump.raw windows.pslist` et `windows.pstree` — liste et filiation des processus
  - `windows.netscan` — connexions réseau au moment de la capture
  - `windows.malfind` — détection d'injection de code possible
  - `windows.cmdline` — lignes de commande des processus en cours
- Croise la chaîne de processus `powershell.exe` du Jour 7 dans ces sorties.
- Construis une timeline simple en combinant les CSV exportés de KAPE avec la sortie horodatée de Volatility dans un tableur, triée chronologiquement.
- Identifie l'événement **patient zéro** (la première action malveillante) et l'étendue complète de ce qu'elle a touché.

**Couvre :** analyse mémoire, analyse de timeline
</details>

<details>
<summary><strong>Jour 10 — Analyse de phishing</strong></summary>

- Construis toi-même un email de phishing de test réaliste — soit un fichier `.eml` brut construit manuellement, soit via un sandbox mail gratuit (ex. Mailtrap/Mailhog). Fais délibérément un décalage entre `From`/`Return-Path` et fais échouer le SPF.
- Lis les en-têtes manuellement : regarde l'en-tête `Authentication-Results` pour `spf=fail`/`dkim=fail`, et compare l'expéditeur affiché avec le serveur d'origine réel dans les en-têtes `Received`.
- Defang avant de tout documenter : `http` → `hxxp`, `.` → `[.]`.
- Vérifie la réputation de l'URL/domaine (fictif/de test) avec le scan URL de VirusTotal ou urlscan.io — sans le visiter directement dans un navigateur.

**Couvre :** analyse d'en-têtes email, détection de contenu malveillant, reconnaissance de l'ingénierie sociale
</details>

<details>
<summary><strong>Jour 11 — Bases de l'analyse de malware</strong></summary>

- Récupère un échantillon connu et labellisé depuis **MalwareBazaar** (compte/clé API gratuit requis). Manipule-le **uniquement** dans une VM entièrement isolée, hors ligne, restaurée depuis un snapshot propre — aucun dossier partagé, aucun réseau pont/NAT.
- **Analyse statique** : `sha256sum sample`, `strings -n 8 sample | less`, puis charge-le dans **PEStudio** pour vérifier les imports, sections et l'entropie (une entropie élevée sur toutes les sections est un indicateur de packer).
- **Analyse dynamique (optionnelle, plus avancée)** : détone dans la VM isolée, surveille avec **Process Monitor**/**Process Explorer**, vérifie la persistance avec **Autoruns**, et capture tout trafic généré avec Wireshark sur le segment isolé (il n'ira nulle part — c'est normal, tu observes juste le comportement).
- Extrais les IOC : hash du fichier, chemins de fichiers déposés, noms de mutex, et toute IP/domaine contacté (sinkholé/non-routable).

> ⚠️ Toujours dans une VM entièrement isolée et jetable. Restaure ton snapshot immédiatement après cette journée.

**Couvre :** analyse de malware statique/dynamique, extraction d'IOC
</details>

<details>
<summary><strong>Jour 12 — Surveillance de sécurité réseau</strong></summary>

- Active le **mode promiscuous** sur ton port group isolé (sur ESXi : Politique de sécurité → Mode Promiscuous → Accepter) pour que le trafic puisse être observé.
- Lance **Suricata** ou **Zeek** temporairement sur la **VM Kali** (aucune VM de monitoring dédiée nécessaire) en mode IDS avec le ruleset gratuit **ET Open**, attaché au segment isolé.
- Relance l'un de tes tests Atomic précédents (Jour 4 ou 7) tout en capturant simultanément avec **Wireshark**.
- Corrèle toute alerte Suricata/Zeek avec les paquets exacts dans la capture Wireshark en utilisant l'horodatage de l'alerte et le 5-tuple (IP/port source/destination, protocole).
- Supprime les grosses captures `.pcap` une fois que tu as extrait les paquets/captures d'écran pertinents pour ton carnet.

**Couvre :** analyse de trafic réseau, IDS, artefacts réseau
</details>

<details>
<summary><strong>Jour 13 — Enrichissement en threat intelligence</strong></summary>

- Compile tous les IOC collectés sur l'ensemble du projet dans un tableur unique : hash / IP / domaine / journée d'origine / technique associée.
- Vérifie chacun : **VirusTotal** (réputation hash/URL), **AbuseIPDB** (score de réputation IP), **AlienVault OTX** (recherche de pulse/contexte).
- Rédige une courte note de renseignement distinguant le **renseignement tactique** (ce hash/cette IP spécifique est/n'est pas connu comme malveillant) du **renseignement stratégique** (le schéma global — ex. "usage de LOLBin + persistance registre" — correspond à un comportement de malware commodity courant, pas à une APT ciblée).

**Couvre :** fondamentaux de la threat intelligence, OSINT
</details>

<details>
<summary><strong>Jour 14 — Documentation & playbook</strong></summary>

- Rédige proprement l'incident du Jour 7 avec cette structure : Résumé exécutif, Timeline, Détails techniques, Tableau des IOC, Impact, Cause racine, Recommandations.
- Produis **deux versions** : un rapport technique complet (extraits de logs, références d'artefacts, sorties Volatility/KAPE) et un résumé exécutif d'une page (impact business et risque, sans jargon).
- Construis un **playbook** pour ce type d'incident : un arbre de décision — Critères de déclenchement d'alerte → Étapes de triage → Étapes de confinement → Checklist d'éradication → Validation de récupération → Clôture.
- Pousse tout sur un dépôt GitHub avec une structure claire, ex. :
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

Tu as 8 Go de RAM ou moins, ou très peu d'espace disque ? Fais tourner une VM à la fois : attaque la VM Windows, exporte les logs, éteins-la, puis démarre la VM dont tu as besoin ensuite pour lancer Zircolite/chasser/analyser. Comme il n'y a pas de serveur SIEM toujours allumé à faire tourner, ce mode s'intègre naturellement à l'architecture — plus lent, mais chaque exercice reste faisable.

---

## ⚠️ Note sur le périmètre

Ce parcours ne couvre pas la surveillance de sécurité cloud (logging AWS/Azure/GCP), l'automatisation SOAR complète, ni l'exploitation centralisée d'un SIEM multi-hôtes (corréler des alertes en direct sur de nombreux hôtes) — cela nécessite des comptes cloud payants, des outils entreprise, ou plus de stockage/calcul qu'un home lab à 3 VMs. Si tu as plus tard l'espace disque pour une 4ème VM, réintégrer Wazuh ou Security Onion comme **amélioration Phase 2** est une suite naturelle une fois les fondamentaux ici bien maîtrisés. Une **Phase 2** utilisant le logging gratuit AWS/Azure pourrait aussi suivre comme future entrée CyberAtaraxia Careers.

---

## 👤 À propos du développeur

- 🎓 Étudiant en Ingénierie Réseau à **Polytech Dijon**, spécialisation Cybersécurité
- 🔵 Blue Teamer | Auditeur ICT | Administrateur réseau
- 🏢 Stagiaire Cybersécurité chez **Axem Belgium**
- 🌱 Poursuit actuellement **Blue Team L1 · ISO 27001/2022 · CCNA**

<p align="center">
  <a href="https://linkedin.com/in/selim-jerbi-b355a0202">
    <img src="https://img.shields.io/badge/LinkedIn-Connect%20with-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  &nbsp;
  <a href="mailto:selimjerbi66@gmail.com">
    <img src="https://img.shields.io/badge/Gmail-Contact-D14836?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
</p>

<div align="center">
  <sub>CyberAtaraxia Careers — Open Source · Construit avec sens par Selim JERBI</sub>
</div>
