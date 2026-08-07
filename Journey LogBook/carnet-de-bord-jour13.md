<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 13 — Enrichissement en Threat Intelligence

Partie du parcours CyberAtaraxia Careers — SOC Analyst
</div>

---

## 🧭 Contexte

**Date :** 07/08/2026

**Objectif du jour :** centraliser tous les IOC collectés depuis le début du laboratoire dans un référentiel unique, enrichir ces indicateurs à l'aide de plateformes de Threat Intelligence (VirusTotal, AbuseIPDB et AlienVault OTX), puis produire une courte note distinguant renseignement tactique et renseignement stratégique.

**Environnement du lab :** Kali (`10.10.10.30`), Tiny10 (`10.10.10.10`), Ubuntu Server (`10.10.10.20`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

### Compilation des IOC du projet

Tous les indicateurs collectés au fil des exercices ont été regroupés dans un tableur unique.

Les IOC recensés proviennent notamment :

- du test ProcDump (Jour 5) ;
- de la simulation d'incident (Jour 7) ;
- de l'analyse mémoire et de la timeline (Jour 9) ;
- de l'analyse d'email de phishing (Jour 10) ;
- de l'analyse du malware Mirai (Jour 11).

Pour chacun, j'ai renseigné :

- le type d'IOC ;
- la journée d'origine ;
- la technique MITRE ATT&CK associée ;
- les commentaires utiles à l'investigation.

Le tableau constitue désormais une base IOC réutilisable pour un SIEM, un EDR ou une plateforme CTI.

---

### Vérification de réputation du malware

Le principal IOC du projet est le hash SHA256 extrait pendant l'analyse de malware du Jour 11.

```
11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328
```

Recherche effectuée sur **VirusTotal**.

Résultat :

- **38 moteurs sur 62 détectent le fichier comme malveillant** ;
- famille majoritaire : **Mirai** ;
- catégorie : **Trojan / Backdoor Linux** ;
- plusieurs moteurs le classent également comme malware DDoS.

Parmi les détections les plus représentatives :

- Microsoft : `Backdoor:Linux/Mirai`
- Kaspersky : `HEUR:Backdoor.Linux.Mirai`
- Symantec : `Linux.Mirai`
- ESET : `Linux/Mirai`
- TrendMicro : `Backdoor.Linux.MIRAI`

La réputation obtenue confirme que l'échantillon téléchargé pendant le Jour 11 est un malware connu et largement documenté.

---

### Recherche d'autres IOC

Les autres IOC collectés pendant le laboratoire ont également été examinés.

Les exécutables légitimes utilisés pendant les simulations (ProcDump, PowerShell, Sysmon) ne constituent pas des IOC malveillants à eux seuls puisqu'ils sont des outils système ou d'administration.

Aucune adresse IP ni domaine directement exploitable n'a été extrait de l'échantillon analysé, ce qui limite naturellement les recherches sur AbuseIPDB ou AlienVault OTX.

Cette absence est cohérente avec le type d'analyse réalisée (analyse principalement statique).

---

### Renseignement tactique

Les vérifications effectuées montrent que :

- le hash SHA256 analysé est déjà connu des plateformes de Threat Intelligence ;
- la famille Mirai est correctement identifiée par une majorité des moteurs antivirus ;
- aucun IOC réseau supplémentaire (IP ou domaine) n'a pu être confirmé à partir de cet échantillon.

Ces informations permettent à un analyste SOC de confirmer rapidement qu'il ne s'agit pas d'un fichier inconnu mais d'une menace déjà largement référencée.

---

### Renseignement stratégique

En prenant du recul sur l'ensemble du laboratoire, les différentes journées décrivent une chaîne d'attaque cohérente reposant principalement sur des techniques très répandues :

- exécution PowerShell ;
- utilisation de LOLBins (ProcDump) ;
- vol d'identifiants via LSASS ;
- persistance par clés Run du registre ;
- analyse et enrichissement des IOC.

L'ensemble correspond davantage au comportement d'un malware "commodity" ou d'une campagne opportuniste qu'à une intrusion ciblée de type APT.

Les différents IOC collectés pourront être réutilisés dans un SIEM afin d'améliorer les capacités de détection et d'accélérer les futures investigations.

---

## Comment ça s'est passé

Cette journée était très différente des précédentes.

Contrairement aux exercices orientés détection ou forensic, le travail consistait surtout à prendre du recul sur l'ensemble du laboratoire afin de transformer les différents artefacts collectés depuis le Jour 4 en renseignement exploitable.

Le principal enseignement est qu'un IOC isolé apporte peu d'informations. En revanche, lorsqu'il est enrichi par des plateformes spécialisées puis replacé dans son contexte (technique MITRE utilisée, chronologie de l'attaque, origine de l'artefact), il devient beaucoup plus utile pour un analyste SOC.

Autre constat intéressant : tous les IOC ne peuvent pas être enrichis de la même manière. Les hash sont généralement très bien référencés sur VirusTotal, tandis que certains échantillons ne contiennent aucun domaine ou aucune adresse IP directement exploitable. L'absence de résultat est donc parfois une information en soi, et non un échec de l'investigation.

---

## ✅ Points validés aujourd'hui

- [x] Compilation des IOC collectés depuis le début du laboratoire
- [x] Centralisation dans un tableau unique
- [x] Association des IOC aux techniques MITRE ATT&CK
- [x] Vérification du hash SHA256 sur VirusTotal
- [x] Identification de la famille Mirai
- [x] Distinction entre renseignement tactique et stratégique
- [x] Production d'une note de Threat Intelligence

---

## 📌 Notes pour la suite

- Conserver le tableau IOC comme référentiel principal pour le rapport final du Jour 14.
- Les IOC issus d'Atomic Red Team doivent toujours être distingués des IOC provenant d'un véritable malware afin d'éviter toute confusion pendant une investigation.
- Un enrichissement plus poussé pourrait être réalisé avec une plateforme CTI dédiée (MISP, OpenCTI ou AlienVault OTX) si davantage d'IOC réseau étaient disponibles.
- Le hash SHA256 de l'échantillon Mirai constitue désormais l'IOC le plus pertinent du laboratoire et pourra être réutilisé dans de futurs exercices de détection ou de Threat Hunting.
