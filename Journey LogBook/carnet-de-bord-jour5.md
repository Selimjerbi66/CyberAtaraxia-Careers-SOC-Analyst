<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 5 — Création et tuning de règles de détection

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 04/08/2026
**Objectif du jour :** clôturer la vérification de T1003.001 laissée en attente au Jour 4, créer une règle Sigma permettant de détecter les accès suspects à `lsass.exe`, valider cette règle avec Zircolite sur les logs Sysmon générés par Atomic Red Team, puis effectuer un premier tuning en identifiant et en excluant une activité légitime de Microsoft Defender.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

Le Jour 4 s'était terminé avec un point encore incertain : le test `T1003.001` avait réussi à générer un dump mémoire de LSASS avec ProcDump, mais je n'avais pas encore vérifié directement si **Sysmon Event ID 10 (Process Access)** avait capturé l'accès à `lsass.exe`.

Le Jour 5 a donc commencé par cette vérification.

L'objectif était ensuite de transformer cette observation en **règle de détection Sigma**, de la tester avec Zircolite, puis de faire un premier exercice de **detection tuning** à partir d'une activité légitime observée dans les mêmes logs.

---

## 🔎 Vérification de T1003.001 — Sysmon Event ID 10

Sur Tiny10, j'ai recherché les événements Sysmon correspondant à `Event ID 10` :

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=10]]" -MaxEvents 20 | Format-List
```

Cette fois, la vérification attendue du Jour 4 a bien été réalisée.

Les logs contenaient des événements `Process Access` ciblant :

```text
C:\Windows\system32\lsass.exe
```

La présence d'Event ID 10 confirmait donc que ma télémétrie Sysmon était bien capable d'observer les accès à LSASS.

➡️ **T1003.001 : détection possible avec la télémétrie actuelle.**

Le point qui était resté jaune dans le carnet du Jour 4 pouvait donc être clôturé et devenir une technique exploitable pour la création d'une règle Sigma.

---

## 🧪 Création de la règle Sigma

Sur Kali, dans le dossier des règles Zircolite :

```bash
cd ~/Zircolite/rules
nano lsass_access.yml
```

J'ai créé la règle :

```yaml
title: Suspicious LSASS Process Access
id: 8c8f0b9d-1a2f-4b3c-9e5d-1234567890ab
status: experimental
description: Detects process access to lsass.exe consistent with credential dumping tools (e.g. ProcDump, Mimikatz)
references:
    - https://attack.mitre.org/techniques/T1003/001/
tags:
    - attack.credential_access
    - attack.t1003.001
logsource:
  category: process_access
  product: windows
detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
    GrantedAccess: '0x1FFFFF'
  condition: selection
falsepositives:
  - Legitimate antivirus/EDR memory scanning
  - Backup or monitoring tools
level: high
```

La logique est volontairement simple :

```text
TargetImage = lsass.exe
        AND
GrantedAccess = 0x1FFFFF
        ↓
      HIGH
```

L'objectif est de détecter un accès à la mémoire de LSASS présentant des droits d'accès particulièrement larges, comportement cohérent avec le scénario de credential dumping simulé avec ProcDump.

La règle est directement utilisable par Zircolite : aucune conversion manuelle supplémentaire n'est nécessaire.

---

## 🧪 Premier test avec Zircolite

J'ai utilisé le fichier EVTX exporté depuis Tiny10 :

```text
~/day5_test_lsass.evtx
```

et lancé Zircolite avec la règle nouvellement créée :

```bash
cd ~/Zircolite
source .venv/bin/activate

python3 zircolite.py \
    -e ~/day5_test_lsass.evtx \
    -o ~/resultat_day5.json \
    -r rules/lsass_access.yml
```

Zircolite a correctement chargé la règle :

```text
[+] Loading ruleset(s)
[>] Converting Native Sigma to Zircolite ruleset : rules/lsass_access.yml
[✓] Converted 1 rules
[+] 1 rules loaded
```

Le fichier EVTX a été automatiquement identifié comme :

```text
windows_evtx (evtx) - confidence: high
```

Zircolite a ensuite analysé :

```text
4,394 events
```

Résultat :

```text
HIGH    Suspicious LSASS Process Access    3    T1003.001
```

Résumé :

```text
🎯 Detections       3 HIGH
📏 Coverage         1/1 rules matched (100.0%)
🔍 Matched          3 events across 1 rules
```

La règle a donc produit **3 détections HIGH** sur les événements correspondant au scénario Atomic Red Team.

➡️ **T1003.001 : règle Sigma validée avec succès.**

---

## 🔬 Analyse des événements détectés

J'ai ensuite inspecté le résultat JSON produit par Zircolite :

```bash
python3 -m json.tool ~/resultat_day5.json | less
```

Un des événements montrait notamment :

```text
SourceImage:
C:\AtomicRedTeam\ExternalPayloads\procdump64.exe

TargetImage:
C:\Windows\system32\lsass.exe

GrantedAccess:
0x1fffff
```

La recherche ciblée :

```bash
python3 -m json.tool ~/resultat_day5.json | grep -E "SourceImage|TargetImage|GrantedAccess"
```

a permis de confirmer que les trois événements détectés correspondaient au scénario attendu :

```text
procdump.exe     → lsass.exe → 0x1fffff
procdump64.exe   → lsass.exe → 0x1fffff
procdump64.exe   → lsass.exe → 0x1fffff
```

La règle ne s'est donc pas déclenchée sur un événement arbitraire : les trois correspondances sont bien liées à l'activité ProcDump générée par Atomic Red Team.

---

## 🧩 Identification d'une activité légitime — Microsoft Defender

Une étape importante du Jour 5 consistait à identifier une activité légitime pouvant générer un comportement similaire et donc potentiellement créer du bruit dans une détection réelle.

J'ai recherché les Event ID 10 liés à LSASS :

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=10]]" -MaxEvents 50 |
    Where-Object { $_.Message -match "lsass" } |
    Format-List
```

Cette recherche a permis d'identifier une activité légitime de Microsoft Defender.

Le processus :

```text
MsMpEng.exe
```

était observé en train d'accéder à :

```text
C:\Windows\system32\lsass.exe
```

Plusieurs valeurs de `GrantedAccess` ont été observées pour cette activité légitime, notamment :

```text
0x1000
0x1410
0x1418
```

Il s'agit d'un exemple concret d'activité de sécurité légitime pouvant apparaître dans les mêmes sources de télémétrie qu'une activité malveillante.

### Point important

Ces événements Defender ne correspondaient pas à la sélection exacte de ma règle initiale, qui recherchait :

```text
GrantedAccess = 0x1FFFFF
```

Par conséquent, **les événements Defender observés n'étaient pas responsables des 3 alertes HIGH** générées par Zircolite.

Cela constitue néanmoins un excellent candidat pour documenter le principe de tuning : si une activité Defender correspondant à la sélection apparaissait dans un autre jeu de logs, elle pourrait être explicitement exclue.

---

## 🛠️ Tuning de la règle

J'ai créé une deuxième règle afin de conserver la règle originale comme référence et de documenter séparément la version tunée :

```bash
cd ~/Zircolite/rules
nano lsass_access_tuned.yml
```

La nouvelle règle ajoute une exclusion pour Microsoft Defender :

```yaml
title: Suspicious LSASS Process Access - Tuned
id: 3b7f4c21-6d8a-4e91-b2f5-9c7a1e6d4f30
status: experimental
description: Detects suspicious process access to lsass.exe with high access rights while excluding Microsoft Defender.
references:
    - https://attack.mitre.org/techniques/T1003/001/
tags:
    - attack.credential_access
    - attack.t1003.001
logsource:
    category: process_access
    product: windows
detection:
    selection:
        TargetImage|endswith: '\lsass.exe'
        GrantedAccess: '0x1FFFFF'
    filter_defender:
        SourceImage|endswith: '\MsMpEng.exe'
    condition: selection and not filter_defender
falsepositives:
    - Legitimate security software
    - Administrative or monitoring tools
level: high
```

La logique est désormais :

```text
TargetImage = lsass.exe
        AND
GrantedAccess = 0x1FFFFF
        AND
SourceImage ≠ MsMpEng.exe
        ↓
      HIGH
```

L'objectif du tuning est donc de conserver la détection de l'activité suspecte tout en documentant explicitement le processus de sécurité légitime.

---

## 🧪 Validation de la règle tunée

J'ai ensuite relancé Zircolite avec la règle tunée :

```bash
python3 zircolite.py \
    -e ~/day5_test_lsass.evtx \
    -o ~/resultat_day5_tuned.json \
    -r rules/lsass_access_tuned.yml
```

Zircolite a confirmé que la règle était valide :

```text
[✓] Converted 1 rules
[+] 1 rules loaded
```

Le même fichier de 4 394 événements a été analysé.

Résultat :

```text
HIGH    Suspicious LSASS Process Access - Tuned    3    T1003.001
```

Résumé :

```text
🎯 Detections       3 HIGH
📏 Coverage         1/1 rules matched (100.0%)
🔍 Matched          3 events across 1 rules
```

Zircolite a également affiché la requête générée à partir de la règle :

```text
SELECT * FROM logs
WHERE
(
    TargetImage LIKE '%\\lsass.exe'
    AND GrantedAccess='0x1FFFFF'
)
AND
(
    NOT SourceImage LIKE '%\\MsMpEng.exe'
)
```

Cette sortie permet de vérifier concrètement que la condition d'exclusion est bien prise en compte par le moteur.

---

## 🔬 Vérification des événements après tuning

J'ai inspecté les événements détectés par la règle tunée :

```bash
python3 -m json.tool ~/resultat_day5_tuned.json | grep -E "SourceImage|TargetImage|GrantedAccess"
```

Les trois correspondances étaient :

```text
GrantedAccess: 0x1fffff
SourceImage: C:\AtomicRedTeam\ExternalPayloads\procdump.exe
TargetImage: C:\Windows\system32\lsass.exe

GrantedAccess: 0x1fffff
SourceImage: C:\AtomicRedTeam\ExternalPayloads\procdump64.exe
TargetImage: C:\Windows\system32\lsass.exe

GrantedAccess: 0x1fffff
SourceImage: C:\AtomicRedTeam\ExternalPayloads\procdump64.exe
TargetImage: C:\Windows\system32\lsass.exe
```

Aucun événement `MsMpEng.exe` n'apparaissait dans les résultats de détection.

J'ai également vérifié explicitement :

```bash
python3 -m json.tool ~/resultat_day5_tuned.json | grep -i "MsMpEng"
```

Aucun événement Defender n'était retourné par cette recherche.

---

## ⚖️ Comparaison avant / après tuning

Pour vérifier que le tuning n'avait pas cassé la détection, j'ai relancé la règle originale :

```bash
python3 zircolite.py \
    -e ~/day5_test_lsass.evtx \
    -o ~/resultat_day5_original.json \
    -r rules/lsass_access.yml
```

La règle originale a produit :

```text
3 HIGH
1/1 rules matched
3 events across 1 rules
```

La règle tunée a produit exactement le même nombre de détections :

```text
3 HIGH
1/1 rules matched
3 events across 1 rules
```

### Comparaison

|                     | Règle originale | Règle tunée |
| ------------------- | --------------: | ----------: |
| Événements analysés |           4 394 |       4 394 |
| Règles chargées     |               1 |           1 |
| Détections          |          3 HIGH |      3 HIGH |
| Couverture          |           100 % |       100 % |
| Technique           |       T1003.001 |   T1003.001 |
| ProcDump détecté    |             Oui |         Oui |
| Exclusion Defender  |             Non |         Oui |

➡️ **Le tuning n'a pas cassé la détection du scénario ProcDump.**

### Limite à documenter

Il faut cependant rester précis sur ce que démontre ce test.

Les événements Microsoft Defender observés dans les logs initiaux utilisaient `GrantedAccess` `0x1000`, `0x1410` et `0x1418`, alors que la règle recherche `0x1FFFFF`.

Par conséquent, **le dataset utilisé ne permet pas de démontrer une réduction effective du nombre d'alertes**, puisque les événements Defender ne correspondaient déjà pas à la sélection initiale.

Le test démontre plutôt que :

1. une activité légitime d'accès à LSASS a été identifiée ;
2. une exclusion explicite de `MsMpEng.exe` a été ajoutée à la règle ;
3. Zircolite applique effectivement cette exclusion ;
4. la détection ProcDump reste opérationnelle.

C'est une distinction importante dans une démarche de detection engineering : je ne considère pas une réduction du bruit comme démontrée simplement parce qu'une exclusion a été ajoutée.

---

## 💡 Ce que j'ai appris

### 1. Une règle de détection ne doit pas seulement être écrite

Le passage important aujourd'hui était :

```text
Log brut
   ↓
Observation du comportement
   ↓
Hypothèse de détection
   ↓
Règle Sigma
   ↓
Test avec une attaque simulée
   ↓
Validation
   ↓
Recherche de faux positifs
   ↓
Tuning
   ↓
Revalidation
```

C'est la première fois dans le lab que je suis réellement cette chaîne de bout en bout.

### 2. Un événement Sysmon seul ne constitue pas forcément une alerte

Les Event ID 10 de Microsoft Defender montrent qu'un accès à LSASS n'est pas automatiquement malveillant.

Le contexte est essentiel :

```text
SourceImage
TargetImage
GrantedAccess
```

et, dans un environnement réel, d'autres éléments devraient également être pris en compte.

### 3. Le tuning doit être justifié

Ajouter :

```yaml
SourceImage|endswith: '\MsMpEng.exe'
```

n'est pas suffisant en soi.

Il faut pouvoir expliquer :

* pourquoi le processus est légitime ;
* pourquoi il peut générer le comportement ;
* quel bruit il peut produire ;
* pourquoi son exclusion ne supprime pas le comportement réellement recherché.

### 4. La comparaison avant/après est essentielle

Le tuning n'est pas terminé au moment où la règle est modifiée.

Il faut vérifier que :

```text
avant tuning
→ détection fonctionnelle

après tuning
→ détection toujours fonctionnelle
```

Dans mon cas, la détection ProcDump est restée à **3 HIGH** après modification.

---

## 🗂️ Fichiers créés aujourd'hui

### Règles Sigma

```text
~/Zircolite/rules/lsass_access.yml
~/Zircolite/rules/lsass_access_tuned.yml
```

### Résultats Zircolite

```text
~/resultat_day5.json
~/resultat_day5_original.json
~/resultat_day5_tuned.json
```

Le fichier EVTX utilisé pour les tests était :

```text
~/day5_test_lsass.evtx
```

---

## 📊 Résultat technique du Jour 5

```text
Technique ATT&CK :
T1003.001 — OS Credential Dumping: LSASS Memory

Télémétrie :
Sysmon Event ID 10

Simulation :
Atomic Red Team / ProcDump

Moteur :
Zircolite v3.7.6

Logs analysés :
4,394 événements

Règle initiale :
1/1 matched — 100 %

Détections initiales :
3 HIGH

Règle tunée :
1/1 matched — 100 %

Détections après tuning :
3 HIGH
```

---

## ✅ Points validés aujourd'hui

* [x] Vérification Event ID 10 pour T1003.001 effectuée
* [x] Accès à `lsass.exe` observé dans Sysmon
* [x] Règle Sigma `lsass_access.yml` créée
* [x] Règle chargée et convertie correctement par Zircolite
* [x] EVTX de 4 394 événements analysé
* [x] 3 détections HIGH obtenues
* [x] Détections confirmées comme provenant de ProcDump
* [x] Mapping MITRE T1003.001 confirmé
* [x] Activité légitime de Microsoft Defender (`MsMpEng.exe`) identifiée
* [x] Faux positif potentiel documenté
* [x] Règle `lsass_access_tuned.yml` créée
* [x] Exclusion `MsMpEng.exe` ajoutée
* [x] Règle tunée correctement interprétée par Zircolite
* [x] 3 détections HIGH conservées après tuning
* [x] Comparaison règle originale / règle tunée effectuée
* [x] Aucun impact négatif observé sur la détection ProcDump

---

## 📌 Notes pour la suite

* Conserver `lsass_access.yml` comme version initiale et `lsass_access_tuned.yml` comme version tunée.
* Garder les résultats JSON comme preuves de validation pour le portfolio.
* Dans une version future de la règle, envisager une approche plus robuste que la seule valeur `GrantedAccess = 0x1FFFFF`, afin d'éviter de dépendre d'une unique valeur de droits d'accès.
* Tester éventuellement le comportement de la règle sur un dataset contenant un accès LSASS légitime qui correspond réellement à la sélection, afin de mesurer une réduction effective des faux positifs.
* Le prochain objectif du parcours est de poursuivre la chaîne SOC : partir des détections obtenues pour passer progressivement à la chasse, l'investigation et la réponse à incident.

---

## 🏁 Bilan du Jour 5

Le Jour 4 m'avait permis d'identifier un angle mort potentiel dans ma télémétrie : je savais que ProcDump avait réussi à dumper LSASS, mais je n'avais pas encore confirmé que l'accès au processus était visible dans Sysmon.

Aujourd'hui, j'ai fermé cette boucle.

Je suis passé de :

```text
ProcDump réussit
       ↓
"Est-ce que Sysmon le voit ?"
```

à :

```text
ProcDump
   ↓
Sysmon Event ID 10
   ↓
TargetImage = lsass.exe
   ↓
Sigma
   ↓
Zircolite
   ↓
3 HIGH
   ↓
T1003.001
```

Puis j'ai ajouté une première étape de detection engineering plus avancée :

```text
Détection
   ↓
Recherche de bruit légitime
   ↓
Microsoft Defender identifié
   ↓
Exclusion documentée
   ↓
Re-test
   ↓
Détection toujours fonctionnelle
```

➡️ **Jour 5 : terminé.**

Le lab dispose désormais de sa première règle Sigma personnalisée, testée contre une technique ATT&CK simulée et accompagnée d'une première démarche de tuning.
