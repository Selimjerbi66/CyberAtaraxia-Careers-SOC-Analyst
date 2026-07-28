<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 1 — Fondations du lab

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 27/07/2026
**Objectif du jour :** monter l'infrastructure de base du home lab — hyperviseur, réseau isolé, 3 VMs provisionnées et patchées, snapshot de référence.

**Environnement du lab :**

| VM | Rôle | OS | Utilisateur | Mot de passe |
|---|---|---|---|---|
| Kali | Attaquant | Kali Linux | `kali` | `kali` |
| Ubuntu Server | Victime Linux | Ubuntu Server 22.04 | `victim` | `victim` |
| Tiny10 | Victime Windows | Tiny10 (voir note ci-dessous) | `victim` | `victim` |

> 💡 **Note — pourquoi Tiny10 et non un Windows 10 classique ?**
> **Tiny10** est une version allégée non-officielle de Windows 10 (créée par le développeur NTDEV), qui retire le bloatware, les services inutiles et une bonne partie de la télémétrie/apps préinstallées de Microsoft, pour un système beaucoup plus léger en RAM et en espace disque. C'est exactement le genre d'avantage recherché dans un lab à ressources limitées : boot plus rapide, empreinte disque réduite, moins de bruit inutile dans les logs.
> Pour ce lab, **ça ne pose aucun problème** : le noyau, le registre, l'Event Log et tous les composants dont on a besoin (Sysmon, auditd-équivalent Windows, etc.) restent identiques à un Windows 10 standard. Tiny10 se comporte **exactement comme un Windows 10 classique** pour tous les exercices du programme (Sysmon, ATT&CK, forensique, etc.) — seul le superflu a été retiré.

---

## ✅ Ce qui a été fait aujourd'hui
 
Aujourd'hui, j'ai posé les fondations du lab. L'idée du jour était simple : avoir un hyperviseur stable, un réseau totalement isolé du reste de mon réseau domestique, et trois VMs propres, patchées, prêtes à servir de terrain de jeu pour les deux semaines à venir.
 
### Vérification de l'hyperviseur
 
J'ai commencé par vérifier que mon hyperviseur : **[ESXi](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20vSphere%20Hypervisor&freeDownloads=true)** tournait correctement. Rien de particulier à signaler ici — c'était déjà en place et stable, donc je suis passé directement à la partie réseau.
 
### Création du réseau isolé
 
L'étape suivante, et probablement la plus importante de la journée, a été de créer un réseau totalement coupé du monde extérieur. J'ai créé un nouveau vSwitch / port group **sans aucun uplink physique attaché** — c'est-à-dire qu'aucune carte réseau physique de mon serveur ESXi n'est reliée à ce segment. Concrètement, ça veut dire que toute VM placée dessus ne peut communiquer qu'avec les autres VMs du même segment, jamais avec Internet ni avec mon réseau domestique.
 
C'est une décision volontaire et centrale du projet : je vais simuler des attaques (via Atomic Red Team) et manipuler des échantillons de malware plus tard dans le programme, donc avoir cette barrière physique/logique dès le premier jour est non négociable.
 
### Provisionnement des trois VMs
 
J'ai ensuite créé mes trois machines virtuelles, une par une, en les laissant temporairement sur un segment réseau qui a accès à Internet — le temps de les installer et de les patcher complètement. Une fois à jour, chaque VM sera basculée sur le segment isolé.
 
Les trois VMs du lab sont :
 
- **[Kali Linux](https://www.kali.org/get-kali/)** — ce sera ma machine attaquante. Compte : `kali` / mot de passe : `kali`.
- **[Ubuntu Server 22.04](https://ubuntu.com/download/server)** — une des deux victimes, côté Linux. Compte : `victim` / mot de passe : `victim`.
- **[Tiny10](https://lecrabeinfo.net/telecharger/tiny10-23h2/)** — la victime Windows. Compte : `victim` / mot de passe : `victim`.
 
J'ai créé les disques virtuels des trois VMs en **thin provisioning** plutôt qu'en épais, pour ne pas gaspiller d'espace disque dès le départ — un choix qui compte vu mes contraintes de stockage.
 
### Adressage et bascule vers le réseau isolé
 
Une fois les trois VMs patchées, je leur ai assigné des IP statiques, puis j'ai déplacé leurs cartes réseau virtuelles du segment temporaire vers le segment isolé.
 
J'ai vérifié la connectivité avec un simple `ping` entre les trois machines — tout passe. Puis j'ai vérifié dans l'autre sens, à savoir qu'aucune des trois VMs ne pouvait plus atteindre Internet (`ping 8.8.8.8` en échec sur les trois). C'était le test le plus important de la journée : confirmer que l'isolation fonctionne réellement, pas juste sur le papier.
 
### Snapshot et documentation
 
Pour terminer, j'ai pris un snapshot de chacune des trois VMs, avec un label clair : `clean-baseline-day1`. C'est mon filet de sécurité pour tout le reste du programme — si quelque chose tourne mal plus tard (notamment lors des simulations d'attaque ou de l'analyse de malware), je pourrai toujours revenir à cet état propre.
 
J'ai aussi dessiné le diagramme réseau du lab (IP, rôles de chaque VM) dans mon carnet de notes SOC, pour garder une trace visuelle de l'architecture.
 
---
 
## Comment ça s'est passé
 
Aucun problème rencontré aujourd'hui. La journée s'est déroulée sans accroc — l'installation et le patch des trois VMs ont pris le plus de temps, mais rien d'imprévu. Journée assez calme comparée à ce qui m'attend niveau détection et attaques dans les prochains jours.
 
---
 
## Ce que je retiens pour la suite
 
- Le snapshot `clean-baseline-day1` est ma référence propre : je ne dois jamais l'écraser, seulement en créer de nouveaux par-dessus si besoin.
- Tiny10 va se comporter comme un Windows 10 normal pour tout le reste du programme — pas besoin d'adapter les prochaines étapes à cause de ça.
- L'isolation réseau est confirmée fonctionnelle : je peux attaquer sans risque de sortir du lab.
---

## 🎯 Résultat

Aucun problème rencontré aujourd'hui — journée sans incident technique. Le lab de base est opérationnel : 3 VMs isolées, patchées, avec un snapshot de référence propre.

## ✅ Points validés

- [x] Hyperviseur stable
- [x] Réseau interne isolé créé (sans uplink)
- [x] 3 VMs provisionnées et patchées (Kali, Ubuntu Server, Tiny10)
- [x] IP statiques assignées, isolation vérifiée
- [x] Snapshot `clean-baseline-day1` pris sur les 3 VMs
- [x] Diagramme réseau documenté

---

## 📌 Notes pour la suite

- Le snapshot `clean-baseline-day1` reste la référence propre à ne jamais écraser — restauration possible à tout moment.
- Tiny10 se comporte comme un Windows 10 standard pour la suite du programme (Sysmon, Atomic Red Team, forensique) : aucune adaptation nécessaire.
