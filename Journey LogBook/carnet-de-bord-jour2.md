<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 2 — Sources de logs & moteur de détection local (Zircolite)

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 28/07/2026
**Objectif du jour :** activer une télémétrie riche sur les VMs victimes (Sysmon/auditd), contrôler la volumétrie des logs, installer un moteur de détection local (Zircolite) et valider la chaîne complète avec un premier test.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

<details>
<summary><strong>1. Sysmon sur la VM Windows (Tiny10)</strong></summary>

- Téléchargé Sysmon (Sysinternals) + config communautaire SwiftOnSecurity (`sysmonconfig.xml`).
- Installé avec :
  ```
  sysmon64.exe -accepteula -i sysmonconfig.xml
  ```
- Vérifié que le service tourne (`sc query Sysmon64`) et que les événements arrivent (`wevtutil qe Microsoft-Windows-Sysmon/Operational /c:5 /f:text`).

**Résultat :** télémétrie riche activée (création de process, connexions réseau, modifications de registre, etc.) au lieu du logging Windows par défaut.

</details>

<details>
<summary><strong>2. auditd sur la VM Ubuntu Server</strong></summary>

- Installé et activé auditd :
  ```
  sudo apt install auditd audispd-plugins -y
  sudo systemctl enable --now auditd
  ```
- Ajouté les règles dans `/etc/audit/rules.d/soc-lab.rules` : surveillance de `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, exécutions (`execve`), modules kernel.
- Chargé et vérifié les règles : `sudo augenrules --load` / `sudo auditctl -l`.

**Résultat :** visibilité sur les fichiers sensibles et les exécutions côté Linux.

</details>

<details>
<summary><strong>3. Limitation de la taille des logs</strong></summary>

- **Windows (Tiny10)** : plafonné les canaux Security/System/Application à 50 Mo et Sysmon à 100 Mo (`wevtutil sl ... /ms:...`).
- **Ubuntu Server** : configuré la rotation dans `/etc/audit/auditd.conf` (`max_log_file = 50`, `num_logs = 5`, `max_log_file_action = ROTATE`).

**Résultat :** pas de risque de saturation disque pendant les 14 jours du lab.

</details>

<details>
<summary><strong>4. Installation de Zircolite (moteur de détection local, sur Kali)</strong></summary>

- Cloné le dépôt officiel : `git clone https://github.com/wagga40/Zircolite.git`
- Créé un environnement virtuel Python (`python3 -m venv .venv`) et installé les dépendances (`pip install -r requirements.txt`).

**Résultat :** moteur de détection Sigma opérationnel, sans serveur ni agent — voir incident de parcours ci-dessous.

</details>

<details>
<summary><strong>5. Premier test de détection</strong></summary>

- Export des logs Windows en EVTX :
  ```
  wevtutil epl Microsoft-Windows-Sysmon/Operational C:\temp\sysmon_export.evtx
  ```
- Transfert du fichier `.evtx` de Tiny10 vers Kali.
- Analyse avec Zircolite :
  ```
  python3 zircolite.py -e ~/sysmon_export.evtx -o ~/resultat_sysmon.json
  ```

**Résultat obtenu :**

| Métrique | Valeur |
|---|---|
| Règles Sigma chargées | 2171 |
| Événements traités | 252 (37 filtrés) |
| Détections | 1 (MEDIUM) |
| Règle matchée | *Sysmon Configuration Change* |
| Durée | 0.4s |

✅ La chaîne complète fonctionne : **Sysmon → export EVTX → Zircolite → alerte Sigma.**

La détection obtenue (changement de config Sysmon) est cohérente : elle correspond simplement à l'installation de Sysmon faite plus tôt dans la journée — pas une vraie anomalie, mais la preuve que le pipeline détecte bien des événements réels.

</details>

<details>
<summary><strong>6. Finalisation</strong></summary>

- VMs remises sur le segment réseau isolé (plus d'uplink Internet).
- Connectivité vérifiée : ping OK entre les 3 VMs, ping vers `8.8.8.8` en échec (isolation confirmée).
- Snapshot pris : `day2-logs-ready`.

</details>

---

## 🐛 Troubleshooting

### Erreur : conflits de permissions/répertoire avec Zircolite

**Contexte :** Zircolite avait été installé une première fois en tant qu'utilisateur **root**.

**Symptôme :** confusion entre les chemins `~/` et `/root/` — les commandes cherchaient/écrivaient les fichiers au mauvais endroit selon l'utilisateur actif, provoquant des erreurs de permission et des chemins introuvables.

**Cause racine :** un `git clone` fait en root crée les fichiers avec un propriétaire et un chemin home différents (`/root/Zircolite`) de ceux de l'utilisateur normal (`/home/kali/Zircolite`). Toute commande lancée ensuite en utilisateur normal ne retrouve donc pas les mêmes fichiers, d'où la confusion de répertoire.

**Résolution :**
1. Suppression complète du dossier Zircolite installé en root.
2. `exit` de la session root pour revenir à l'utilisateur normal (`kali`).
3. Réinstallation propre avec `git clone` en tant qu'utilisateur normal (pas de `sudo`).

**Résultat :** plus aucune erreur de permission ensuite ; tous les tests (étape 5) se sont déroulés sans problème.

**Leçon retenue :** ne jamais installer d'outils utilisateur (Zircolite, environnements Python, etc.) en root sauf nécessité explicite — toujours travailler avec un utilisateur normal pour éviter les conflits de propriété de fichiers et de chemins home.

---

## ✅ Points validés aujourd'hui

- [x] Télémétrie riche Windows (Sysmon sur Tiny10)
- [x] Télémétrie Linux (auditd sur Ubuntu Server)
- [x] Contrôle de la volumétrie des logs
- [x] Moteur de détection local fonctionnel (Zircolite)
- [x] Premier test de détection réussi (1 alerte MEDIUM)
- [x] Retour sur réseau isolé + snapshot `day2-logs-ready`

---

## 📌 Notes pour la suite (Jour 3+)

- Le pipeline **log → export → Zircolite → alerte** est validé, réutilisable pour chaque test Atomic Red Team des prochains jours.
- Toujours travailler avec l'utilisateur normal (`kali`) sur la VM Kali (pas root) pour Zircolite et les futurs outils.
- Penser à supprimer les exports `.evtx` bruts après analyse pour économiser l'espace disque.
