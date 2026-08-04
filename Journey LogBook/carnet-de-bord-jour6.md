<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 6 — Chasse aux menaces (Threat Hunting)

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 04/08/2026
**Objectif du jour :** changer de posture par rapport aux jours précédents — au lieu de partir d'une technique connue et de vérifier si elle est détectée (Jour 4/5), partir de deux hypothèses de chasse et aller vérifier moi-même dans les logs si elles se confirment. Transformer ensuite un résultat confirmé en règle Sigma réutilisable.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

### Génération de l'activité à chasser

Avant de pouvoir chasser quoi que ce soit, il fallait remettre le framework Atomic Red Team en état de marche — après avoir fermé PowerShell depuis le Jour 5, deux choses avaient sauté : la politique d'exécution (`Set-ExecutionPolicy -Scope Process` ne survit pas à la fermeture de session) et le module `Invoke-AtomicRedTeam` lui-même, plus chargé en mémoire. J'ai dû relancer :

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

Pour éviter de refaire ça à chaque session, j'ai ajouté ces deux lignes à mon profil PowerShell (`$PROFILE`) pour qu'elles se chargent automatiquement à l'avenir.

En cherchant un test Atomic pour T1105 (Ingress Tool Transfer, la catégorie qui couvre les LOLBins de téléchargement), j'ai eu la surprise suivante :

```
Found 0 atomic tests applicable to windows platform for Technique T1105
```

Aucun test disponible pour cette technique sur Windows dans la bibliothèque actuelle. J'ai donc simulé l'activité manuellement :

```powershell
certutil.exe -urlcache -split -f "https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/README.md" C:\temp\test_download.html
```

`certutil` est un utilitaire Windows légitime (gestion de certificats), mais détourné ici pour télécharger un fichier — un LOLBin classique (*Living Off the Land Binary*) utilisé par de vrais attaquants pour éviter d'avoir à apporter leurs propres outils sur la machine cible.

### Hypothèse 1 — LOLBins pour télécharger des fichiers

**Requête de chasse :**

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]" |
    Where-Object { $_.Message -match "certutil" -and ($_.Message -match "urlcache" -or $_.Message -match "-decode") } |
    Format-List TimeCreated, Message
```

Un seul événement trouvé, avec le détail complet attendu : `certutil.exe` avec la commande exacte `-urlcache -split -f` suivie de l'URL GitHub, lancé depuis une session PowerShell interactive de l'utilisateur `victim`.

**Verdict : confirmé, mais attendu** — c'était ma propre commande de test qui remonte. Pas une découverte en soi, mais une validation utile : la chasse fonctionne, le pattern LOLBin est bien visible dans Sysmon avec l'URL complète en clair.

### Hypothèse 2 — Activité PowerShell hors horaires

**Requête de chasse :**

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]" |
    Where-Object { $_.Message -match "powershell.exe" } |
    Select-Object TimeCreated, Message |
    Export-Csv -Path C:\temp\powershell_activity.csv -NoTypeInformation

Import-Csv C:\temp\powershell_activity.csv |
    Where-Object {
        $t = [datetime]$_.TimeCreated
        $t.Hour -lt 9 -or $t.Hour -ge 17
    } |
    Format-List
```

Trois événements remontés, tous vers 4h du matin, sur trois jours consécutifs différents (01/08, 02/08, 03/08). Premier réflexe : ce pattern régulier à quelques minutes d'écart, plusieurs jours de suite, ne collait pas avec mes propres horaires de travail sur le lab — un bon candidat à creuser plutôt qu'à ignorer.

En regardant le détail complet, les trois événements étaient **rigoureusement identiques** dans leur structure :
- `powershell.exe -ExecutionPolicy Restricted -Command Write-Host 'Final result: 1';`
- Exécuté par `NT AUTHORITY\SYSTEM` (pas un compte utilisateur)
- Processus parent : `CompatTelRunner.exe -m:appraiser.dll -f:DoScheduledTelemetryRun`

Ça correspond au **Microsoft Compatibility Appraiser**, une tâche planifiée Windows native qui collecte de la télémétrie de compatibilité système. Le payload (`Write-Host 'Final result: 1'`) est totalement anodin, le contexte SYSTEM et la régularité quotidienne confirment une tâche planifiée légitime plutôt qu'une activité suspecte.

**Verdict : écarté.** C'est le résultat le plus intéressant de la journée — pas parce que j'ai trouvé une attaque, mais parce que j'ai correctement identifié et justifié un faux positif avec des preuves concrètes (contexte SYSTEM, binaire Microsoft signé, payload trivial, pattern régulier) plutôt que de le laisser remonter comme une alerte non qualifiée.

### Transformer l'Hypothèse 1 confirmée en règle Sigma

Sur Kali, j'ai créé la règle :

```bash
nano ~/Zircolite/rules/certutil_lolbin.yml
```

```yaml
title: Suspicious Certutil Usage (LOLBin Download/Decode)
id: 5e9a3c47-2b1d-4f8e-8a6c-7d3b9f1e4c62
status: experimental
description: Detects certutil.exe being used to download or decode files, a common LOLBin technique for payload delivery
references:
    - https://attack.mitre.org/techniques/T1105/
    - https://lolbas-project.github.io/lolbas/Binaries/Certutil/
tags:
    - attack.command_and_control
    - attack.t1105
logsource:
  category: process_creation
  product: windows
detection:
  selection:
    Image|endswith: '\certutil.exe'
  keywords:
    CommandLine|contains:
      - 'urlcache'
      - '-decode'
  condition: selection and keywords
falsepositives:
  - Legitimate certificate management operations (rare in most environments)
level: medium
```

### Validation avec Zircolite

Première tentative :

```bash
python3 zircolite.py -e ~/day6_hunt.evtx -o ~/resultat_day6.json -r rules/certutil_lolbin.yml
```

```
ModuleNotFoundError: No module named 'orjson'
```

J'avais oublié d'activer l'environnement virtuel Python avant de lancer la commande. Une fois corrigé :

```bash
source .venv/bin/activate
python3 zircolite.py -e ~/day6_hunt.evtx -o ~/resultat_day6.json -r rules/certutil_lolbin.yml
```

Résultat : **1 détection MEDIUM**, "Suspicious Certutil Usage (LOLBin Download/Decode)", mappée sur **T1105**, avec une couverture de 100% (1/1 règle chargée, 1 événement matché sur 10 277 événements analysés). Zircolite a même affiché la couverture ATT&CK correspondante (*Command and Control*, 1 technique, 1 hit).

➡️ La règle fonctionne exactement comme prévu : elle capture la commande de test certutil, et rien d'autre dans les 10 277 événements du fichier.

---

## Comment ça s'est passé

Deux petits accrocs aujourd'hui, tous les deux rapidement résolus.

### Perte du module Atomic Red Team entre les sessions

Comme au Jour 5, la politique d'exécution PowerShell (`-Scope Process`) ne persiste pas entre les sessions, et le module Atomic doit être rechargé à chaque nouvelle ouverture de PowerShell. Cette fois, j'ai ajouté les deux commandes à mon profil PowerShell pour ne plus avoir à y penser.

### Environnement virtuel Python non activé

Erreur classique : lancer `zircolite.py` sans avoir activé `.venv` au préalable, provoquant une `ModuleNotFoundError` sur `orjson` (une dépendance installée uniquement dans l'environnement virtuel). Résolu simplement avec `source .venv/bin/activate` avant de relancer la commande.

**Ce que je retiens :** systématiquement vérifier `(.venv)` dans le prompt avant de lancer une commande Zircolite — c'est le même genre de vigilance que pour la politique d'exécution PowerShell côté Windows.

---

## ✅ Points validés aujourd'hui

- [x] Activité LOLBin (`certutil -urlcache`) générée et capturée par Sysmon
- [x] Hypothèse 1 testée et confirmée (résultat attendu, validation du pipeline de chasse)
- [x] Activité PowerShell hors horaires identifiée (3 occurrences)
- [x] Hypothèse 2 testée et correctement écartée (Microsoft Compatibility Appraiser, faux positif justifié)
- [x] Journal de chasse structuré rempli (`hunt-log-jour6.md`) : hypothèse → sources → requête → résultats → verdict
- [x] Règle Sigma `certutil_lolbin.yml` créée et déposée dans le ruleset Zircolite
- [x] Règle validée : 1 détection MEDIUM, T1105, 100% de couverture

---

## 📌 Notes pour la suite

- Le réflexe acquis aujourd'hui — ne pas s'arrêter à "l'horaire est suspect" mais creuser jusqu'au processus parent et au contexte utilisateur — sera directement réutile pendant la simulation d'incident du Jour 7.
- Ajouter `Set-ExecutionPolicy` et `Import-Module Invoke-AtomicRedTeam` au profil PowerShell a réglé le problème de façon durable — plus besoin d'y penser aux prochains jours utilisant Atomic Red Team.
- Toujours vérifier `(.venv)` actif avant de lancer Zircolite sur Kali.
- La règle `certutil_lolbin.yml` s'ajoute à `lsass_access.yml` / `lsass_access_tuned.yml` du Jour 5 dans le ruleset personnel — le dossier `rules/` du projet commence à devenir un vrai petit corpus de détection sur mesure.
