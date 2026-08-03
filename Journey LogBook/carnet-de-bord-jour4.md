<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 4 — Mapping MITRE ATT&CK

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 03/08/2026
**Objectif du jour :** installer Atomic Red Team sur Tiny10, exécuter 3 techniques MITRE ATT&CK réelles (T1059.001, T1003.001, T1547.001), vérifier ce que ma télémétrie actuelle capture pour chacune, puis construire un layer ATT&CK Navigator pour visualiser mes angles morts.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

L'objectif du jour était de passer de la théorie à la pratique : jusqu'ici j'avais de la télémétrie (Jour 2) et je savais l'interroger (Jour 3), mais je n'avais jamais généré de vraie activité malveillante pour voir ce que ça donne concrètement dans les logs. Aujourd'hui, j'ai simulé trois techniques ATT&CK réelles sur Tiny10, une par une, en vérifiant après chacune ce que Sysmon avait capturé.

### Installation d'Atomic Red Team

Sur Tiny10, en PowerShell administrateur :

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force
```

L'installation a demandé l'ajout du fournisseur NuGet (accepté avec `Y`), puis s'est terminée sans problème. `Get-Command -Module invoke-atomicredteam` a confirmé la présence de toutes les fonctions attendues, notamment `Invoke-AtomicTest`.

### Test 1 — T1059.001 (Command and Scripting Interpreter: PowerShell)

En consultant les détails du test (`-ShowDetails`), j'ai découvert que le Test #1 de cette technique dans la bibliothèque actuelle s'appelle **"Mimikatz"** — il télécharge et exécute Mimikatz pour dumper les identifiants, plutôt qu'une simple exécution PowerShell basique. La bibliothèque Atomic évolue, donc le contenu exact des tests numérotés change avec le temps ; ce n'était pas grave en soi, juste une découverte à noter.

**Premier essai — échec :**
```
Exception calling "Start" with "0" argument(s): "Access is denied"
```

**J'ai désactivé la protection en temps réel de Microsoft Defender**, puis relancé le test — cette fois, succès complet : Mimikatz s'est téléchargé, exécuté, et a dumpé les hashs NTLM des sessions actives (mon compte `victim`, plus plusieurs comptes système).

**Vérification avec Sysmon (Event ID 1) :**

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]" -MaxEvents 5 | Format-List
```

Résultat excellent : Sysmon a capturé **la totalité de la chaîne d'attaque** — `cmd.exe` lançant `powershell.exe`, avec la ligne de commande complète et lisible, incluant l'URL exacte du script Mimikatz téléchargé et l'appel `Invoke-Mimikatz -DumpCreds`. Rien n'est passé inaperçu au niveau de la création de processus.

➡️ **T1059.001 : détection complète (vert)** — Sysmon voit tout, y compris la ligne de commande malveillante en clair.

### Test 2 — T1003.001 (OS Credential Dumping: LSASS Memory via ProcDump)

**Premier essai — échec :**
```
Exception calling "Start" with "0" argument(s): "Access is denied"
```

**Deuxième essai — échec différent :**
```
The system cannot find the path specified.
```

Ce deuxième message m'a mis sur la piste d'une dépendance manquante. J'ai vérifié et récupéré les prérequis :

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1 -GetPrereqs
Invoke-AtomicTest T1003.001 -TestNumbers 1 -CheckPrereqs
```

`-GetPrereqs` a téléchargé **ProcDump** (outil Sysinternals légitime) dans `C:\AtomicRedTeam\ExternalPayloads\`. `-CheckPrereqs` a confirmé que tout était en place.

**Troisième essai — succès :**
```
ProcDump v12.01 - Sysinternals process dump utility
Dump 1 complete: 55 MB written in 0.7 seconds
```

Le dump mémoire de `lsass.exe` a été créé avec succès dans `C:\Windows\Temp\lsass_dump.dmp` (55 Mo).

**Vérification — point en attente :** je n'ai pas encore lancé la requête `Get-WinEvent` sur l'Event ID 10 (Process Access) pour ce test précisément. J'ai en revanche une confirmation *indirecte* via l'Event ID 13 : Sysmon a loggé l'exécution de `procdump.exe` et `procdump64.exe` avec sa propre règle intégrée **"Alert,Sysinternals Tool Used"**, au moment de l'acceptation de l'EULA de ProcDump. C'est un signal utile, mais ce n'est pas la vérification centrale prévue par le plan — à faire en premier au prochain démarrage du lab :

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=10]]" -MaxEvents 10 | Format-List
```

➡️ **T1003.001 : à confirmer** — signal indirect positif (détection de l'outil ProcDump lui-même), mais la vérification directe de l'accès à LSASS (Event ID 10) reste à faire.

### Test 3 — T1547.001 (Boot or Logon Autostart Execution: Registry Run Keys)

Ce test ajoute une entrée dans la clé de registre `HKCU\...\Run` pour simuler un mécanisme de persistance classique.

```powershell
Invoke-AtomicTest T1547.001 -TestNumbers 1
```
```
The operation completed successfully.
Exit code: 0
```

**Vérification avec Sysmon (Event ID 13) :**

Résultat parfait : Sysmon a capturé exactement l'événement attendu — `reg.exe` créant la valeur `Atomic Red Team` sous `HKCU\...\Run`, avec le détail complet (`C:\Path\AtomicRedTeam.exe`). Fait intéressant : la config Sysmon communautaire utilisée (SwiftOnSecurity) tague déjà cet événement avec sa propre règle nommée **"T1060,RunKey"** — la détection de ce type de persistance est donc un cas prévu et documenté nativement dans la config, pas une découverte de ma part.

➡️ **T1547.001 : détection complète (vert)**.

### Découverte annexe — la désactivation de Defender laisse une trace

En parcourant les résultats de l'Event ID 13, j'ai remarqué quelque chose d'important que je n'avais pas anticipé : le moment où j'ai désactivé la protection en temps réel de Defender (avant le Test 1) a lui-même été capturé par Sysmon, avec sa propre règle **"T1089,Tamper-Defender"** — `MsMpEng.exe` modifiant `HKLM\...\Real-Time Protection\DisableRealtimeMonitoring` à `1`.

C'est un excellent exemple concret pour mon carnet : même l'action de désactiver l'antivirus (une étape "méthodologique" de mon côté, pas une attaque) est elle-même une technique ATT&CK documentée (*Impair Defenses: Disable or Modify Tools*) et elle a été détectée par ma télémétrie actuelle. Bon rappel que dans un vrai environnement, ce genre d'action attirerait immédiatement l'attention d'un analyste SOC.

### Construction du layer ATT&CK Navigator

Sur [mitre-attack.github.io/attack-navigator](https://mitre-attack.github.io/attack-navigator/), j'ai créé un nouveau layer Enterprise et coloré les techniques testées :

| Technique | Couleur | Justification |
|---|---|---|
| T1059.001 | 🟢 Vert | Sysmon ID 1 — ligne de commande complète capturée |
| T1003.001 | 🟡 À confirmer | Signal indirect (outil détecté), vérification ID 10 en attente |
| T1547.001 | 🟢 Vert | Sysmon ID 13 — événement + règle native "T1060,RunKey" |
| T1562.001 (bonus, découverte annexe) | 🟢 Vert | Désactivation Defender détectée via règle "T1089,Tamper-Defender" |

---

## Comment ça s'est passé

Deux points de troubleshooting aujourd'hui, tous les deux résolus.

### Erreur "Access is denied" sur les deux premiers tests

T1059.001 et T1003.001 ont chacun échoué une première fois avec exactement la même erreur (`Exception calling "Start"... Access is denied`). Pour T1059.001, le test est passé après avoir désactivé la protection en temps réel de Defender. Je ne suis pas certain à 100% que ce soit la seule cause : PowerShell affichait un chemin `system32` qui ne garantit pas en soi une session réellement élevée en administrateur, donc il est possible que les deux facteurs (élévation + Defender) aient joué un rôle. **Point à clarifier dans un prochain test** : vérifier explicitement l'élévation avec `([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)` avant de conclure que c'est uniquement Defender qui bloquait.

### Dépendance manquante pour T1003.001

Après avoir contourné le premier blocage, une deuxième erreur est apparue (`The system cannot find the path specified`) : le test attendait `procdump.exe` à un emplacement précis, qui n'existait pas encore. La commande `-GetPrereqs` a réglé ça en une étape en téléchargeant l'outil automatiquement. **Leçon retenue :** systématiquement lancer `-CheckPrereqs` (ou directement `-GetPrereqs`) avant un nouveau test Atomic, plutôt que de découvrir la dépendance manquante après un échec.

### Rappel à ne pas oublier

La protection en temps réel de Microsoft Defender a été désactivée pour ces tests — **à réactiver avant de fermer la session de travail**, pour ne pas laisser Tiny10 sans protection active plus longtemps que nécessaire, même en lab isolé.

---

## ✅ Points validés aujourd'hui

- [x] Atomic Red Team installé et fonctionnel sur Tiny10
- [x] T1059.001 exécuté et détecté (Sysmon ID 1, ligne de commande complète)
- [x] T1003.001 exécuté avec succès (dump LSASS via ProcDump) — vérification Event ID 10 encore à faire
- [x] T1547.001 exécuté et détecté (Sysmon ID 13, règle native "T1060,RunKey")
- [x] Découverte annexe : la désactivation de Defender elle-même est détectée (T1089, Tamper-Defender)
- [x] Layer MITRE ATT&CK Navigator créé avec les techniques colorées

---

## 📌 Notes pour la suite (Jour 5+)

- **Priorité immédiate :** lancer la requête Event ID 10 pour confirmer/infirmer la détection de l'accès à LSASS sur T1003.001, avant de figer la couleur dans le layer ATT&CK.
- Si l'Event ID 10 ne remonte rien, ce sera l'angle mort à traiter en priorité au Jour 5 avec une règle Sigma dédiée (`TargetImage` se terminant par `\lsass.exe`).
- Réactiver Microsoft Defender (protection en temps réel) sur Tiny10.
- Toujours lancer `-CheckPrereqs` avant un nouveau test Atomic pour éviter les échecs de dépendance en cours de route.
- Nettoyer les artefacts laissés par les tests (`C:\Windows\Temp\lsass_dump.dmp`, clé de registre `Run\Atomic Red Team` via la commande de cleanup fournie par le test).
