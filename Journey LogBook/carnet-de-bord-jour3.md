<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 3 — Requêtes manuelles & revue de logs

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 31/07/2026
**Objectif du jour :** apprendre à interroger directement les logs exportés, sans dashboard, sur Windows (`Get-WinEvent`) et Ubuntu (`ausearch`), puis construire des routines de revue réutilisables pour la suite du programme.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

L'idée du jour était simple sur le papier mais importante dans la pratique : après avoir mis en place la télémétrie (Jour 2), je devais apprendre à l'interroger moi-même, à la main, sans dashboard type Wazuh. Dans un lab sans SIEM centralisé, c'est ce genre de réflexe manuel qui remplace la barre de recherche d'un vrai SIEM.

### Connexions échouées et réussies (Tiny10)

J'ai ouvert PowerShell sur Tiny10 — première petite étape, découvrir qu'il fallait bien être dans PowerShell et non dans l'invite de commandes classique (`cmd`), sinon `Get-WinEvent` n'est tout simplement pas reconnu. Une fois dans le bon environnement, j'ai lancé mes premières requêtes :

```powershell
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]"
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]"
```

Résultat : seulement 2 échecs de connexion (Event ID 4625) sur toute la période observée, ce qui colle bien avec ce qu'on attend d'un système propre. En creusant le détail avec `Format-List`, j'ai vu que :
- un échec venait de `msedge.exe` en logon interactif — probablement une tentative d'authentification interne d'Edge, rien d'alarmant ;
- l'autre échec concernait une tentative de connexion réseau sur le compte **Guest**, désactivé — un point que j'ai noté comme "à surveiller" même si isolé, puisque c'est exactement le genre de compte générique qu'on désactive pour éviter ce type de tentative.

Côté connexions réussies (4624), énormément d'événements sont remontés — cohérent avec toute l'activité de la VM depuis le Jour 1 (connexions système, services, etc.).

### Création de processus — Sysmon Event ID 1

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]" | Select-Object TimeCreated, Message | Format-List
```

Ici, tout ce qui est remonté correspond exactement aux actions que j'avais faites moi-même pendant l'installation de Sysmon au Jour 2 : `wevtutil.exe`, `sc.exe`, `Sysmon64.exe`, lancés depuis `cmd.exe`. C'est un bon exemple concret de ce à quoi ressemble une chaîne parent/enfant normale.

### Connexions réseau — Sysmon Event ID 3

Même principe avec l'Event ID 3, pour observer les connexions sortantes. Rien de particulier à signaler, l'essentiel du jour était de valider que la requête fonctionne et de comprendre ce que je regarde.

### Équivalent auditd sur Ubuntu Server

Sur Ubuntu, j'ai utilisé les deux clés définies au Jour 2 :

```bash
sudo ausearch -k identity
sudo ausearch -k exec
```

Pour `identity`, tout ce qui remonte, ce sont en fait mes propres actions de configuration d'auditd (des `add_rule`/`remove_rule`) faites au Jour 2 — pas de vrais accès suspects à `/etc/shadow` ou `/etc/sudoers`, juste le bruit de ma propre mise en place.

Pour `exec`, énormément d'exécutions remontent, mais toutes liées à des scripts système standards déclenchés à la connexion (`update-motd`, `lesspipe`, `dircolors`...) ainsi qu'à mes propres commandes `sudo ausearch`. Rien d'anormal, c'est le bruit habituel d'une session shell qui démarre.

### Construction des 3 routines de revue

C'était le vrai objectif de la journée : ne pas se contenter de lancer des commandes une fois, mais les transformer en un **pense-bête réutilisable**. J'ai créé un fichier séparé, `routines-revue-logs-jour3.md`, avec 3 routines documentées :

1. **Connexions échouées par source** — avec la commande `Get-WinEvent` correspondante, et une note sur ce qui est "normal" (0-2 échecs isolés) vs "suspect" (rafale d'échecs, compte/heure inhabituels).
2. **Création de processus (parent/enfant)** — même principe côté Sysmon Event ID 1, avec un exemple concret de ce qui doit alerter (ex. `winword.exe` qui lance `powershell.exe`).
3. **Connexions réseau sortantes** — Sysmon Event ID 3, avec la distinction entre connexions connues et processus qui ne devraient jamais faire de réseau.

L'idée : dans quelques jours, pendant le hunting (Jour 6) ou la gestion d'un incident (Jour 7+), je n'aurai pas à réinventer ces requêtes — je pourrai les copier-coller directement depuis ce fichier.

### Export des échantillons en CSV

Pour garder une trace concrète et exploitable, j'ai exporté un échantillon de chaque requête :

```powershell
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]" | Select-Object TimeCreated, Id, Message | Export-Csv -Path C:\temp\failed_logons_sample.csv -NoTypeInformation

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=1]]" | Select-Object TimeCreated, Message | Export-Csv -Path C:\temp\process_creation_sample.csv -NoTypeInformation
```

Et côté Ubuntu, avec le format CSV natif d'`ausearch` :

```bash
sudo ausearch -k identity --format csv > ~/identity_sample.csv
sudo ausearch -k exec --format csv > ~/exec_sample.csv
```

J'ai vérifié le contenu des 4 fichiers obtenus (`failed_logons_sample.csv`, `process_creation_sample.csv`, `identity_sample.csv`, `exec_sample.csv`) : tous cohérents avec ce qui avait été observé en direct dans le terminal.

### Nettoyage

Une fois les CSV validés, j'ai supprimé les exports `.evtx` bruts du Jour 2 (`sysmon_export.evtx`, `security_export.evtx`) sur Tiny10 pour libérer de l'espace disque — les CSV suffisent amplement pour garder une trace exploitable.

### Transfert du fichier de routines vers Kali (FileZilla)

Pour centraliser mon travail, j'ai transféré le fichier `routines-revue-logs-jour3.md` depuis mon PC hôte vers ma VM Kali, via **FileZilla** en SFTP (le service SSH ayant déjà été activé sur Kali plus tôt dans le projet).

J'utilise la version portable de FileZilla Client, téléchargeable ici : [Download FileZilla Client](https://filezilla-project.org/download.php?show_all=1)

Connexion : `sftp://kali@192.168.77.10`, dossier distant `/home/kali/Downloads`. Le transfert s'est fait sans problème — capture d'écran à l'appui ci-dessous :

![Transfert FileZilla du fichier de routines vers Kali](https://raw.githubusercontent.com/Selimjerbi66/CyberAtaraxia-Careers-SOC-Analyst/main/Screenshots/filezilla-transfert-jour3.png)

---

## Comment ça s'est passé

Journée sans incident technique majeur. Le seul petit accroc a été de ma part au tout début : j'ai essayé de lancer `Get-WinEvent` dans une invite de commandes classique (`cmd.exe`) au lieu de PowerShell, ce qui donne logiquement une erreur "n'est pas reconnu en tant que commande interne ou externe" — `Get-WinEvent` est une cmdlet PowerShell, pas une commande `cmd`. Une fois basculé dans PowerShell (`PS C:\...>` au lieu de `C:\...>`), tout a fonctionné normalement.

À part ça, toutes les requêtes ont fonctionné du premier coup, les exports CSV se sont bien passés, et le transfert FileZilla vers Kali n'a posé aucun problème.

---

## ✅ Points validés aujourd'hui

- [x] Requêtes manuelles Windows fonctionnelles (`Get-WinEvent` : 4625, 4624, Sysmon ID 1, Sysmon ID 3)
- [x] Requêtes manuelles Ubuntu fonctionnelles (`ausearch -k identity`, `ausearch -k exec`)
- [x] 3 routines de revue documentées dans `routines-revue-logs-jour3.md`
- [x] 4 échantillons CSV exportés et vérifiés
- [x] Exports `.evtx` bruts du Jour 2 supprimés
- [x] Fichier de routines transféré sur Kali via FileZilla (SFTP)

---

## 📌 Notes pour la suite

- Les 3 routines de revue seront réutilisées telles quelles pendant le mapping ATT&CK (Jour 4), le hunting (Jour 6) et la gestion d'incident (Jour 7+).
- Point à surveiller mentalement : la tentative de connexion sur le compte Guest désactivé — pas une alerte en soi, mais bon réflexe de la noter.
- Continuer à nettoyer les exports bruts après chaque analyse pour économiser l'espace disque.
