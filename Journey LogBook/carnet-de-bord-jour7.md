<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 7 — Simulation d'incident & réponse à incident

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 04/08/2026
**Objectif du jour :** simuler une vraie chaîne d'intrusion (T1204.002 → T1059.001 → T1547.001) et dérouler le cycle IR complet dessus — détection, triage, confinement, éradication, récupération, retour d'expérience — en s'appuyant sur les règles Sigma accumulées depuis le Jour 5.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

### Préparatifs

Comme au Jour 6, la politique d'exécution PowerShell et le module Atomic Red Team ne persistent pas entre les sessions. J'ai dû les recharger manuellement :

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

Ce point recommence à revenir régulièrement malgré la modification apportée au `$PROFILE` au Jour 6 — à vérifier si l'ajout a bien été sauvegardé.

### Simulation de la chaîne d'attaque

**Étape 1 — T1204.002 (ouverture d'un document malveillant, simulée) :**

```powershell
New-Item -Path "C:\Users\victim\Downloads\Facture_Q3_2026.docm" -ItemType File -Force
```

Fichier leurre créé à 10:29 AM, pour donner un point d'ancrage narratif au scénario : un utilisateur reçoit un email de phishing avec cette pièce jointe, l'ouvre, active les macros.

**Étape 2 — T1059.001 (Mimikatz) :**

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 1
```

Comme au Jour 4, Mimikatz s'est téléchargé et exécuté sans accroc cette fois (pas d'erreur "Access is denied" — la protection Defender était déjà gérée depuis les jours précédents). Dump complet des identifiants de session obtenu (hash NTLM du compte `victim`, plus les comptes système).

**Étape 3 — T1547.001 (persistance registre) :**

```powershell
Invoke-AtomicTest T1547.001 -TestNumbers 1
```

Exécution réussie, clé `Run\Atomic Red Team` ajoutée dans `HKCU`.

### Export du log

```powershell
wevtutil epl Microsoft-Windows-Sysmon/Operational C:\temp\day7_incident.evtx
```

**Point à noter honnêtement :** je suis passé directement de l'export à l'éradication, sans documenter formellement l'étape de détection via Zircolite ni l'étape de confinement réseau (isolation ESXi) prévues dans le plan. Le pipeline de détection (Sysmon → export → Zircolite → alerte) est déjà validé depuis les Jours 5-6, donc je considère la mécanique acquise, mais je n'ai pas de preuve JSON pour *cet* incident précis — à compléter avant de considérer le ticket réellement clos.

### Éradication

**Tentative d'arrêt des processus malveillants :**

```powershell
Get-Process | Where-Object { $_.ProcessName -match "mimikatz|procdump" } | Stop-Process -Force
```

Aucune sortie — donc rien à tuer. En y réfléchissant, c'est cohérent : Mimikatz s'était déjà auto-terminé via sa propre commande `exit` à la fin du test, avant même d'arriver à cette étape. Bonne découverte méthodologique : l'éradication ne peut pas toujours compter sur un processus encore actif à tuer, surtout pour des outils à durée de vie courte.

**Suppression de la clé de persistance :**

```powershell
Invoke-AtomicTest T1547.001 -TestNumbers 1 -Cleanup
```

Confirmé propre ensuite :
```powershell
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" | Select-Object "Atomic Red Team"
```
→ aucune sortie, la clé a bien disparu.

### Récupération — Option B (vérification manuelle)

**Choix : vérification manuelle plutôt que restauration du snapshot.** Justification : plus représentatif d'un vrai poste de travail qu'on ne peut généralement pas réimager instantanément sans impact métier. Cette approche est plus lente et dépend entièrement de la qualité de ma checklist — contrairement à une restauration de snapshot qui élimine ce risque par construction, mais j'ai choisi de m'entraîner sur le scénario le plus réaliste.

**Vérification 1 — Processus actifs :**
```powershell
Get-Process | Select-Object Name, Id, Path, StartTime | Sort-Object StartTime -Descending | Format-Table -AutoSize
```
Liste complète passée en revue — que des processus Windows/système légitimes (svchost, explorer, msedge, Sysmon64, MsMpEng, etc.), rien de suspect.

Confirmation ciblée :
```powershell
Get-Process | Where-Object { $_.ProcessName -match "mimikatz|procdump" }
```
→ vide, confirmé propre.

**Vérification 2 — Tâches planifiées :**
```powershell
Get-ScheduledTask | Where-Object { $_.TaskPath -notlike "\Microsoft\*" } | Select-Object TaskName, TaskPath, State
```
→ vide, aucune tâche planifiée suspecte.

**Vérification 3 — Clés Run :**
```powershell
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```
→ vide, propre.

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```
→ seule entrée présente : `SecurityHealth` (native Windows, légitime), rien d'ajouté par les tests.

**Vérification 4 — Fichiers résiduels — et découverte inattendue :**

```powershell
Get-ChildItem "C:\Windows\Temp\lsass_dump.dmp" -ErrorAction SilentlyContinue
```

Résultat surprenant : le fichier **existe toujours**, daté du **8/3/2026** (55 880 134 octets, soit ~53 Mo). Ce n'est pas un artefact d'aujourd'hui — c'est le dump LSASS généré par le test ProcDump du **Jour 5**, jamais supprimé depuis. Il était resté sur le disque pendant plus d'une journée sans que je m'en rende compte, jusqu'à ce que cette checklist de vérification du Jour 7 le révèle.

```powershell
Get-ChildItem "C:\Users\victim\Downloads\Facture_Q3_2026.docm" -ErrorAction SilentlyContinue
```
Le fichier leurre d'aujourd'hui est bien présent aussi (attendu, à nettoyer en clôture d'incident).

---

## Comment ça s'est passé

### Découverte principale : un oubli d'éradication du Jour 5 a survécu jusqu'au Jour 7

C'est le vrai enseignement de la journée, plus que la chaîne d'attaque elle-même. Le Jour 5 s'était terminé avec un test ProcDump réussi et une règle Sigma validée, mais **le fichier de dump lui-même n'avait jamais été supprimé**. Il a fallu la checklist de récupération du Jour 7 — pensée pour un incident totalement différent — pour le remarquer par hasard.

**Ce que ça m'apprend :** dans un lab (ou un vrai poste jamais réimagé), les artefacts de tests/incidents précédents s'accumulent silencieusement si l'éradication n'est pas systématiquement vérifiée après *chaque* test, pas seulement à la fin d'un scénario d'incident complet. Un vrai analyste ferait face au même risque sur un poste de production ancien : des IOC oubliés de mois précédents peuvent traîner sans que personne ne les remarque, jusqu'à ce qu'une chasse ou une vérification les révèle.

### Étapes non documentées : détection Zircolite et confinement réseau

Honnêtement, je suis allé trop vite de l'export du log à l'éradication, sans capturer la détection Zircolite formelle ni réaliser le confinement réseau (isolation du port group ESXi) prévus dans le plan. Le ticket d'incident reste donc **ouvert** tant que ces deux points ne sont pas complétés.

---

## ✅ Points validés aujourd'hui

- [x] Chaîne d'attaque simulée (T1204.002 → T1059.001 → T1547.001)
- [x] Export du log Sysmon couvrant l'incident
- [x] Éradication de la clé de persistance (confirmée absente)
- [x] Vérification manuelle complète (processus, tâches planifiées, clés Run)
- [x] Matrice de triage appliquée — sévérité Élevée justifiée
- [x] Découverte et documentation d'un artefact résiduel du Jour 5 non nettoyé
- [ ] Détection Zircolite formelle sur `day7_incident.evtx` — **à faire**
- [ ] Confinement réseau (isolation ESXi) — **à faire/documenter**
- [ ] Suppression de `lsass_dump.dmp` (résidu Jour 5) et `Facture_Q3_2026.docm` (artefact du jour)

---

## 📌 Notes pour la suite

- **Priorité immédiate :** lancer Zircolite sur `day7_incident.evtx` avec le ruleset complet pour clore proprement le ticket d'incident.
- Ajouter une étape systématique de nettoyage des artefacts (fichiers `.dmp`, `.docm`, exports `.evtx`) à la fin de **chaque** journée du lab, pas seulement en fin de scénario d'incident — pour éviter que des résidus anciens ne s'accumulent silencieusement.
- Revérifier que l'ajout au `$PROFILE` du Jour 6 a bien été sauvegardé, puisque le rechargement manuel du module Atomic a encore été nécessaire aujourd'hui.
- Pour le prochain incident, penser à `Get-Date` à chaque transition de phase (attaque → détection → confinement → éradication → récupération) pour avoir une chronologie précise dans le ticket, plutôt que des horaires approximatifs reconstruits après coup.
