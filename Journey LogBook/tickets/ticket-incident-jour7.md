## Ticket d'incident #INC-2026-007

**Horodatage de détection :** Non confirmé formellement — export du log Sysmon (`day7_incident.evtx`) effectué, mais aucune analyse Zircolite n'a été documentée pour cet incident précis avant l'ouverture du ticket. **Action à compléter avant clôture définitive.**

**Source de détection :** Sysmon (Event ID 1 process creation, Event ID 13 registry) — export réalisé, analyse Zircolite en attente.

**Actif concerné :** Tiny10 (victime Windows, `victim`/`victim`)

**Sévérité :** **Élevée**

*Justification (matrice de triage) :*

| Criticité de l'actif → \ Impact technique ↓ | Faible | Moyen | Élevé |
|---|---|---|---|
| **Faible** (poste de test/lab) | Bas | Bas | Moyen |
| **Moyen** (poste utilisateur standard) | Bas | Moyen | **Élevé** ← |
| **Élevé** (serveur/admin) | Moyen | Élevé | Critique |

Tiny10 = poste utilisateur standard (criticité moyenne dans le scénario fictif) × dumping d'identifiants via Mimikatz (impact élevé, risque de mouvement latéral) → **case Élevée**.

---

**Résumé :** Chaîne d'attaque simulée en 3 étapes — T1204.002 (ouverture d'un document malveillant simulée manuellement) → T1059.001 (exécution Mimikatz via PowerShell, dump des hashs NTLM de la session active) → T1547.001 (persistance via clé de registre `HKCU\...\Run`).

**Chronologie :**

| Heure (approximative) | Événement |
|---|---|
| ~10:29 AM | Création du fichier leurre `Facture_Q3_2026.docm` (simulation T1204.002) |
| ~10:29–10:35 AM | Exécution du test T1059.001 (Mimikatz) — dump des identifiants de session |
| ~10:35 AM | Exécution du test T1547.001 — ajout de la clé de registre `Run\Atomic Red Team` |
| Non horodaté précisément | Export du canal Sysmon vers `day7_incident.evtx` |
| Non horodaté précisément | Éradication : tentative de kill process (aucun processus actif trouvé), cleanup de la clé de registre |
| Non horodaté précisément | Vérification de récupération (Option B — vérification manuelle) |

*Note méthodologique : les horodatages précis de chaque phase (`Get-Date`) n'ont pas été systématiquement capturés pendant l'exécution — seule l'heure de création du fichier `.docm` (10:29 AM) est certaine. À corriger pour le prochain incident : logger `Get-Date` à chaque transition de phase.*

**Actions prises :**

1. Simulation de la chaîne d'attaque (T1204.002 → T1059.001 → T1547.001) sur Tiny10.
2. Export du canal Sysmon Operational en EVTX.
3. Tentative d'arrêt des processus `mimikatz`/`procdump` — **aucun processus actif trouvé** (l'outil s'était auto-terminé via sa propre commande `exit` avant cette étape).
4. Suppression de la clé de persistance via `Invoke-AtomicTest T1547.001 -Cleanup` — confirmée absente lors de la vérification (`HKCU:\...\Run` vide).
5. Vérification manuelle complète (Option B) :
   - `Get-Process` complet passé en revue — aucun processus suspect actif.
   - `Get-ScheduledTask` (hors `\Microsoft\*`) — aucune tâche planifiée suspecte.
   - `HKCU:\...\Run` — vide, propre.
   - `HKLM:\...\Run` — seule entrée légitime `SecurityHealth` présente (native Windows).
6. **Découverte annexe pendant la vérification :** artefact résiduel `C:\Windows\Temp\lsass_dump.dmp` (55 880 134 octets), daté du **8/3/2026**, donc laissé par le test ProcDump du **Jour 5** — non lié à l'incident du jour, mais jamais nettoyé depuis. **Action corrective ajoutée :** suppression de ce fichier.
7. Fichier leurre `Facture_Q3_2026.docm` toujours présent au moment de la vérification — à supprimer en clôture d'incident (artefact du jour, distinct du point 6).

**Actions restantes avant clôture :**
- [ ] Lancer Zircolite sur `day7_incident.evtx` avec le ruleset complet (`lsass_access.yml`, `lsass_access_tuned.yml`, `certutil_lolbin.yml`) pour confirmer formellement la détection.
- [ ] Supprimer `C:\Windows\Temp\lsass_dump.dmp` (résidu Jour 5).
- [ ] Supprimer `C:\Users\victim\Downloads\Facture_Q3_2026.docm` (artefact du jour).
- [ ] Confirmer explicitement l'étape de confinement réseau (isolation ESXi), non documentée dans cette session.

**Qui / Quand :** Selim JERBI, 04/08/2026

**Statut :** **En cours — clôture conditionnée aux actions restantes ci-dessus**
