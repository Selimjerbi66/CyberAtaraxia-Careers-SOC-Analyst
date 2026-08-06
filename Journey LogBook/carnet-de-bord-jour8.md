<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 8 — Collecte de preuves & forensique Windows

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 04-06/08/2026
**Objectif du jour :** appliquer le cycle complet de collecte de preuves forensiques sur Tiny10 — capture mémoire, image disque, extraction d'artefacts (KAPE), et analyse approfondie (Autopsy, PECmd, AmcacheParser, RegistryExplorer) — pour confirmer, depuis des sources indépendantes, ce que les jours précédents (notamment l'incident du Jour 7) avaient déjà montré en direct.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`), machine hôte (poste d'analyse) — voir carnet Jour 1 pour le détail de l'architecture.

**Nom du case Autopsy :** `Tiny10`, examinateur : Selim.

---

## ✅ Ce qui a été fait aujourd'hui

### Capture mémoire — DumpIt

Sur Tiny10, j'ai téléchargé DumpIt et capturé la mémoire vive avant toute autre manipulation — principe de base en forensique : le plus volatile en premier.

```powershell
C:\temp\DumpIt.exe
```

Résultat : fichier `.raw` de 5 Go, cohérent avec la RAM allouée à la VM.

**Point de vigilance noté :** j'avais téléchargé DumpIt depuis un site tiers (toolwar.com) plutôt que la source officielle Comae/Magnet Forensics — acceptable dans ce lab isolé et jetable, mais à corriger pour de futures captures (toujours privilégier la source officielle de l'éditeur pour un outil qui touche à la mémoire système).

### Image disque — Export VMDK direct

Plutôt que FTK Imager, j'ai exporté directement le disque virtuel depuis ESXi :

1. Arrêt propre de Tiny10 (`shutdown /s /t 0`) pour garantir un état disque cohérent.
2. Téléchargement des deux fichiers VMDK (`tinyVictim.vmdk` descripteur + `-flat.vmdk`) depuis le Datastore Browser ESXi.
3. Hash SHA-256 noté pour la chaîne de custody.

### KAPE — collecte ciblée d'artefacts

Lancé directement sur Tiny10 :

```powershell
.\kape.exe --tsource C: --target KapeTriage --tdest C:\out
```

Extraction réussie de Prefetch, Event Logs, ruches de registre, MFT, USN Journal, Amcache et ShimCache. Transféré vers Kali pour une première analyse rapide (`find` sur les fichiers extraits), après correction d'un souci de permissions Linux (`chown`/`chmod`, les fichiers avaient conservé des permissions restrictives après extraction).

**Première confirmation obtenue via KAPE :** recherche directe sur les fichiers extraits →

```
/kape_day8/C/Windows/prefetch/PROCDUMP.EXE-37165721.pf
/kape_day8/C/Windows/prefetch/PROCDUMP64.EXE-36874EF3.pf
```

Preuve que ProcDump a bien été exécuté, indépendamment de toute commande PowerShell en direct.

### Chaîne d'outils Eric Zimmerman — PECmd, AmcacheParser, RegistryExplorer

Plutôt que KAPE seul (qui nécessite une inscription bloquante sur le site officiel), j'ai basculé sur les outils individuels d'Eric Zimmerman, téléchargés depuis la source officielle (`download.ericzimmermanstools.com`).

**PECmd (analyse Prefetch) :** lancé sur Tiny10 après plusieurs galères d'installation (mono sur Kali qui échouait sur les dépendances .NET modernes, puis installation réussie du SDK .NET 10 sur Tiny10 directement). Confirmation détaillée du nombre d'exécutions et des horodatages de `PROCDUMP.EXE` / `PROCDUMP64.EXE`, en cohérence avec la découverte KAPE.

**AmcacheParser :** installé sur la machine hôte cette fois (après une tentative avortée de tout faire depuis Tiny10 — annulée et reprise proprement). Extraction d'`Amcache.hve` depuis Autopsy, transfert vers la machine hôte, puis parsing.

**Résultat notable — divergence Prefetch / Amcache :**

| Source | ProcDump détecté ? |
|---|---|
| Prefetch (KAPE + PECmd) | ✅ Oui |
| Amcache (AmcacheParser) | ❌ Non |

Explication retenue : Amcache se met à jour de façon périodique (cycle `Microsoft Compatibility Appraiser`), contrairement à Prefetch qui est immédiat. Le cycle de flush n'était probablement pas encore repassé au moment de la capture. **Leçon clé du jour : ne jamais se fier à une seule source forensique** — Amcache montre bien `DumpIt.exe` et `filezilla.exe` (exécutés plus tard/plus souvent), mais pas ProcDump, sans que ça remette en cause sa réelle exécution, confirmée par ailleurs.

**RegistryExplorer :** extraction de `NTUSER.DAT` de l'utilisateur `victim` depuis Autopsy, chargement dans RegistryExplorer, navigation jusqu'à `Software\Microsoft\Windows\CurrentVersion\Run`.

**Résultat : 0 valeur.** Confirmation finale et la plus solide de toute la chaîne — l'éradication de la clé `Atomic Red Team` du Jour 7 a bien fonctionné, vue cette fois depuis une image disque froide et figée, la méthode la plus fiable en forensique.

### Analyse Autopsy — exploration complète du case

Une fois l'ingestion terminée (ingestion complète : Recent Activity, Hash Lookup, Extension Mismatch, etc.), exploration des artefacts déjà extraits automatiquement par Autopsy, puis export en Excel pour consultation hors ligne.

**Ce que l'export a permis de corroborer, à partir de sources indépendantes des logs Sysmon :**

- **Web Search History** : recherches "dumpit download" (04/08 15:40:18) et "filezilla download" (04/08 15:41:39) — corrèlent exactement avec les téléchargements effectués ce jour-là.
- **Recent Documents / LNK files** : confirme toute la manipulation du dossier `Downloads\Sysmon\` (config Sysmon, `lsass-monitoring.xml`, `sysmonconfig-export.xml`) des Jours 2 et 5 — bonne validation croisée indépendante d'un travail déjà ancien dans le lab.
- **Shell Bags** : navigation confirmée vers `C:\temp`, `C:\Windows`, `Downloads\FileZilla_3.70.6_win64`.
- **USB Devices Attached** : rien d'anormal — uniquement la souris virtuelle VMware, cohérent avec un environnement virtualisé.
- **Extension Mismatch Detected** : a révélé la présence de fichiers `.nupkg` sous `/AtomicRedTeam/atomics/T1218.008/...` — une technique **jamais testée** explicitement dans le parcours. Ça confirme que le téléchargement en masse d'Atomic Red Team au Jour 4 (`-getAtomics`) a récupéré les dépendances de *toutes* les techniques disponibles dans la bibliothèque, pas seulement celles réellement exécutées. **Point méthodologique important : présence de fichiers sur disque ne veut pas dire technique exécutée** — distinction à toujours faire en investigation.
- **Interesting Items** ("Remote Monitoring Management Programs") : Autopsy a flaggé `mstsc.exe` (RDP) et `quickassist.exe` comme "programmes de gestion à distance" — ce sont de simples binaires Windows natifs présents par défaut sur toute installation, pas une preuve d'usage d'outil d'accès distant malveillant. Faux positif attendu, documenté comme tel.
- **Encryption Suspected** : 3 fichiers à haute entropie détectés (`iconcache_256.db`, `WebCacheV01.dat-slack`, `mpenginedb.db` — base de scan Defender). Tous des fichiers système/cache légitimes à haute entropie native, pas une preuve d'exfiltration ou de chiffrement malveillant.

---

## Comment ça s'est passé

Journée riche en rebondissements techniques, mais tous surmontés — le fil conducteur a surtout été des questions de **quelle machine fait quoi**.

### Confusion machine hôte / Tiny10 pour les outils d'analyse

Erreur récurrente aujourd'hui : avoir installé un outil (AmcacheParser) sur une machine, puis avoir le fichier `.hve` à analyser sur l'autre. Réglé en établissant clairement une règle pour la suite : **la machine hôte est le poste d'analyse** (Autopsy, PECmd, AmcacheParser, RegistryExplorer tous centralisés là), **Tiny10 reste la machine victime pure**. Les rares cas où un outil restait sur Tiny10 (comme PECmd, installé avant cette clarification) sont documentés comme une exception assumée.

### Mono vs .NET moderne sur Kali

Tentative de faire tourner PECmd sur Kali via `mono` — a échoué à deux reprises (`Cannot open assembly`, puis `TypeLoadException` sur des dépendances `netstandard`). Abandonné proprement (nettoyage complet des outils Zimmerman et de `mono-runtime` sur Kali) au profit d'une installation native sur Tiny10 avec le SDK .NET, plus simple et sans couche de compatibilité supplémentaire.

**Leçon retenue :** pour les outils .NET modernes (Eric Zimmerman, notamment les builds `net9`), privilégier une machine Windows native plutôt que mono sur Linux — le temps perdu à déboguer la compatibilité dépasse largement le bénéfice de centraliser sur Kali.

### Chemins et dossiers inexistants sur la machine hôte

Plusieurs erreurs `DirectoryNotFoundException` en tentant de télécharger vers `C:\temp` sur la machine hôte — ce dossier, créé de longue date sur Tiny10, n'existait simplement pas encore sur la machine hôte. Résolu avec un `New-Item -ItemType Directory -Force` systématique avant tout téléchargement sur une nouvelle machine.

---

## ✅ Points validés aujourd'hui

- [x] Capture mémoire (DumpIt, 5 Go)
- [x] Image disque complète exportée (VMDK, hash noté)
- [x] Artefacts KAPE extraits et transférés
- [x] Prefetch confirmé via deux méthodes indépendantes (KAPE+PECmd, Autopsy)
- [x] Amcache analysé — divergence avec Prefetch documentée et expliquée
- [x] Clés Run confirmées vides via RegistryExplorer (éradication Jour 7 validée depuis image froide)
- [x] Exploration complète Autopsy (Web History, Recent Documents, Shell Bags, USB, Extension Mismatch, Interesting Items, Encryption Suspected)
- [x] Corrélation indépendante de l'activité des Jours 2-8 via artefacts navigateur/OS
- [x] Faux positifs Autopsy identifiés et documentés (mstsc.exe, quickassist.exe, fichiers haute entropie)
- [x] Découverte méthodologique : présence de fichiers T1218.008 sur disque sans exécution associée

---

## 📌 Notes pour la suite

- Nettoyer le dump mémoire brut, le VMDK téléchargé et les dossiers KAPE une fois ce carnet finalisé — ces fichiers sont volumineux et n'ont plus besoin de rester sur le lab.
- Toujours privilégier une source officielle de téléchargement pour les outils touchant à la mémoire/au système (leçon DumpIt/toolwar.com).
- Règle de répartition des rôles à conserver pour la suite du programme : machine hôte = poste d'analyse, Tiny10 = machine victime pure.
- Le principe "présence de fichiers ≠ exécution" (découvert via les fichiers T1218.008) et "une seule source forensique ne suffit jamais" (découvert via la divergence Prefetch/Amcache) sont deux points forts à réutiliser dans le rapport final du Jour 14.
