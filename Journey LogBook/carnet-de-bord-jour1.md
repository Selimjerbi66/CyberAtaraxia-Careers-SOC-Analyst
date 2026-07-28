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

<details>
<summary><strong>1. Hyperviseur</strong></summary>

- Installation/vérification de la stabilité de l'hyperviseur (ESXi).
- Confirmé que l'environnement est prêt à recevoir les VMs.

</details>

<details>
<summary><strong>2. Réseau interne isolé</strong></summary>

- Création d'un vSwitch/port group **sans uplink physique** (aucune carte réseau physique attachée).
- Ce segment servira de réseau isolé pour l'ensemble du lab, sans accès direct à Internet une fois le provisionnement terminé.

</details>

<details>
<summary><strong>3. Provisionnement des 3 VMs</strong></summary>

- **Kali Linux** — VM attaquant.
- **Ubuntu Server 22.04** — VM victime Linux (`victim`/`victim`).
- **Tiny10** — VM victime Windows (`victim`/`victim`).
- Chaque VM entièrement patchée sur un segment temporairement connecté à Internet.
- Disques virtuels créés en **thin provisioning** pour économiser l'espace disque dès le départ.

</details>

<details>
<summary><strong>4. Adressage IP & isolation</strong></summary>

- IP statiques assignées à chaque VM.
- Cartes réseau virtuelles déplacées sur le segment isolé une fois le provisionnement terminé.
- Connectivité vérifiée : `ping` entre les 3 VMs → OK.
- Confirmé qu'aucune VM ne peut atteindre Internet depuis le segment isolé.

</details>

<details>
<summary><strong>5. Snapshot & documentation</strong></summary>

- Snapshot pris sur chaque VM, étiqueté clairement : `clean-baseline-day1`.
- Diagramme réseau (IP, rôles des VMs) consigné dans le carnet de notes SOC.

</details>

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
