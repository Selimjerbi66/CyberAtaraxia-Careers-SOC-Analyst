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

L'objectif du jour était de sortir de l'aveuglement total : par défaut, ni Windows ni Linux ne loggent grand-chose d'exploitable pour de la détection. Aujourd'hui, je devais mettre en place une vraie source de télémétrie sur mes deux VMs victimes, contrôler que ces logs ne vont pas exploser en taille, puis installer un moteur capable de transformer ces logs bruts en alertes concrètes — sans passer par un SIEM lourd type Wazuh, puisque je n'ai pas la place pour une 4e VM.
 
### Sysmon sur la VM Windows (Tiny10)
 
J'ai commencé par la VM Windows. J'ai téléchargé Sysmon depuis Sysinternals, ainsi qu'une configuration communautaire toute faite (celle de SwiftOnSecurity), que j'ai enregistrée sous `sysmonconfig.xml`. Sans cette config, Sysmon logge un peu n'importe quoi ou pas grand-chose d'utile — la config SwiftOnSecurity est un bon point de départ standard dans l'industrie.
 
J'ai installé Sysmon avec :
 
```
sysmon64.exe -accepteula -i sysmonconfig.xml
```
 
Puis j'ai vérifié deux choses : que le service tournait bien (`sc query Sysmon64`), et que des événements arrivaient réellement dans le journal (`wevtutil qe Microsoft-Windows-Sysmon/Operational /c:5 /f:text`). Les deux vérifications étaient bonnes — Sysmon me donne maintenant une télémétrie beaucoup plus riche que le logging Windows par défaut : création de processus, connexions réseau, modifications de registre, etc.
 
### auditd sur la VM Ubuntu Server
 
Côté Linux, l'équivalent de Sysmon, c'est auditd. Je l'ai installé et activé :
 
```
sudo apt install auditd audispd-plugins -y
sudo systemctl enable --now auditd
```
 
Ensuite j'ai écrit un fichier de règles dans `/etc/audit/rules.d/soc-lab.rules`, pour surveiller ce qui compte vraiment pour un lab SOC : les fichiers liés à l'identité (`/etc/passwd`, `/etc/shadow`, `/etc/sudoers`), toutes les exécutions de commandes (via `execve`), et le chargement de modules kernel. J'ai chargé ces règles avec `sudo augenrules --load` et vérifié qu'elles étaient bien actives avec `sudo auditctl -l`.
 
### Limiter la croissance des logs
 
Avant d'aller plus loin, je me suis occupé d'un point pratique mais important : sans limite, les logs peuvent remplir un disque en quelques jours, et je n'ai pas beaucoup de marge de stockage. Sur Windows, j'ai plafonné la taille des canaux Security/System/Application à 50 Mo chacun, et le canal Sysmon (plus verbeux) à 100 Mo, avec `wevtutil sl <canal> /ms:<octets>`. Sur Ubuntu, j'ai configuré la rotation dans `/etc/audit/auditd.conf` (`max_log_file = 50`, `num_logs = 5`, `max_log_file_action = ROTATE`) pour que les vieux logs soient automatiquement écrasés au lieu de s'accumuler.
 
### Installer Zircolite sur Kali
 
C'est ici que j'ai remplacé l'idée d'un SIEM centralisé par une approche plus légère. J'ai installé **Zircolite** sur ma VM Kali : un outil qui applique des règles Sigma directement sur des fichiers de logs exportés (EVTX pour Windows, JSON pour auditd), sans avoir besoin d'un serveur ni d'un agent installé sur les victimes.
 
L'installation s'est faite via git :
 
```
git clone https://github.com/wagga40/Zircolite.git
cd Zircolite
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```
 
C'est à cette étape que j'ai eu mon seul vrai accroc de la journée (détaillé plus bas dans la section Troubleshooting) : j'avais fait cette installation en root la première fois, ce qui a créé des soucis. Une fois corrigé, tout s'est enchaîné normalement.
 
### Premier test de détection
 
Une fois Zircolite opérationnel, je suis passé au test qui devait valider toute la chaîne : est-ce que je suis capable d'aller d'un événement réel sur Windows jusqu'à une alerte concrète ?
 
J'ai d'abord exporté le canal Sysmon de Windows en EVTX :
 
```
wevtutil epl Microsoft-Windows-Sysmon/Operational C:\temp\sysmon_export.evtx
```
 
J'ai transféré ce fichier de Tiny10 vers Kali, puis lancé Zircolite dessus :
 
```
python3 zircolite.py -e ~/sysmon_export.evtx -o ~/resultat_sysmon.json
```
 
Le résultat a été concluant : Zircolite a chargé 2171 règles Sigma par défaut, traité 252 événements (37 filtrés), et m'a sorti **1 détection de niveau MEDIUM** : une règle appelée *Sysmon Configuration Change*. Le tout en 0.4 seconde.
 
Cette alerte n'a rien d'inquiétant en soi — elle correspond simplement au fait que je venais d'installer/configurer Sysmon un peu plus tôt dans la journée. Mais c'est exactement ce que je voulais voir : la preuve que la chaîne complète fonctionne de bout en bout, **Sysmon → export EVTX → Zircolite → alerte Sigma**, sans avoir eu besoin d'un SIEM centralisé.
 
### Finalisation
 
Pour terminer la journée, j'ai remis les trois VMs sur le segment réseau isolé, revérifié que le ping fonctionnait bien entre elles et que rien ne pouvait sortir vers Internet, puis j'ai pris un snapshot nommé `day2-logs-ready`.
 
---
 
## Comment ça s'est passé
 
La journée s'est globalement bien passée, avec un seul vrai problème rencontré, au moment de l'installation de Zircolite.
 
### Le problème : installation de Zircolite en root
 
J'avais installé Zircolite une première fois en étant connecté en tant que root sur Kali. Ça a très vite créé des erreurs de permissions et des confusions de répertoire — en gros, mes commandes tantôt cherchaient les fichiers dans `~/` (mon home normal, `/home/kali`), tantôt dans `/root/`, selon l'utilisateur avec lequel j'étais connecté au moment de la commande. Résultat : des fichiers introuvables, des erreurs de permission, une vraie confusion.
 
En creusant, la cause est logique : un `git clone` lancé en root crée tous les fichiers dans `/root/Zircolite`, avec le propriétaire `root`. Si je reviens ensuite à mon utilisateur normal (`kali`) pour lancer les commandes, je n'ai évidemment plus accès à ces mêmes fichiers de la même façon, et surtout mon `~/` ne pointe plus vers le même dossier.
 
Pour corriger ça, j'ai :
1. Supprimé complètement le dossier Zircolite installé en root.
2. Fait un `exit` pour sortir de la session root et revenir à mon utilisateur normal.
3. Réinstallé Zircolite proprement avec `git clone`, cette fois en tant qu'utilisateur normal, sans `sudo`.
Après ça, plus aucune erreur — toute la partie test (export, transfert, analyse) s'est déroulée sans accroc.
 
**Ce que je retiens :** ne plus jamais installer d'outils "utilisateur" comme Zircolite en root. Je dois rester sur mon compte normal pour ce genre d'installation, sauf si l'outil l'exige vraiment — sinon je m'expose à exactement ce genre de confusion de chemins.

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
