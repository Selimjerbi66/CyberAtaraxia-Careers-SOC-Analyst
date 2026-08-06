<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 9 — Forensique mémoire & analyse de timeline

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 06/08/2026

**Objectif du jour :** analyser le dump mémoire acquis au Jour 8 avec Volatility 3, corréler les résultats avec les journaux Sysmon du Jour 7 ainsi que les artefacts KAPE et Autopsy du Jour 8, puis reconstruire la chronologie complète de l'incident afin d'identifier le *Patient Zero*.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`), machine hôte (poste d'analyse) — voir carnet Jour 1 pour le détail de l'architecture.

**Dump mémoire analysé :**

```text
/home/kali/DESKTOP-33KBPU8-20260804-110954.raw
```

---

## ✅ Ce qui a été fait aujourd'hui

### Analyse des processus actifs

La première étape a consisté à ouvrir le dump mémoire avec Volatility afin d'obtenir la liste des processus présents au moment de la capture.

```bash
vol -f /home/kali/DESKTOP-33KBPU8-20260804-110954.raw windows.pslist
```

Cette commande a permis de vérifier que la mémoire contenait toujours les principaux processus Windows ainsi que les éléments nécessaires à l'analyse de l'incident.

Pour compléter cette première vue, j'ai ensuite affiché l'arbre des processus :

```bash
vol -f /home/kali/DESKTOP-33KBPU8-20260804-110954.raw windows.pstree
```

L'objectif était de visualiser les relations parent/enfant afin de comprendre comment les différents processus avaient été lancés.

---

### Analyse des connexions réseau

J'ai ensuite recherché les connexions réseau encore présentes au moment de la capture mémoire.

```bash
vol -f /home/kali/DESKTOP-33KBPU8-20260804-110954.raw windows.netscan
```

Aucune connexion particulièrement suspecte n'a été observée.

Ce résultat est cohérent avec le scénario Atomic Red Team utilisé dans le lab, qui ne simule pas de communication réseau persistante.

---

### Analyse des lignes de commande

Pour identifier précisément les commandes utilisées pendant l'incident, j'ai exécuté :

```bash
vol -f /home/kali/DESKTOP-33KBPU8-20260804-110954.raw windows.cmdline
```

Cette analyse permet de récupérer les arguments utilisés par les processus encore présents en mémoire.

Elle constitue un bon complément aux journaux Sysmon puisqu'elle apporte une vision directement issue de la mémoire vive.

---

### Recherche d'injection mémoire

Enfin, j'ai exécuté le plugin **malfind** afin de rechercher d'éventuelles injections de code.

```bash
vol -f /home/kali/DESKTOP-33KBPU8-20260804-110954.raw windows.malfind
```

Aucune injection mémoire significative n'a été détectée.

Ce résultat est attendu : la simulation Atomic Red Team utilisée jusque-là repose principalement sur PowerShell, ProcDump et Mimikatz, sans utiliser de techniques avancées d'injection mémoire.

---

### Corrélation avec les journées précédentes

Une fois les différentes analyses mémoire terminées, j'ai confronté les résultats obtenus avec les preuves déjà collectées.

**Journée 7 — Sysmon**

Les journaux Sysmon montraient déjà :

- l'exécution de PowerShell ;
- le lancement d'Invoke-AtomicTest ;
- l'exécution de ProcDump ;
- l'accès mémoire à `lsass.exe` (Event ID 10).

**Journée 8 — KAPE**

Les artefacts KAPE avaient confirmé indépendamment l'exécution de ProcDump grâce aux fichiers Prefetch ainsi qu'aux différents artefacts Windows extraits.

**Journée 8 — Autopsy**

Autopsy avait permis de confirmer plusieurs activités réalisées pendant le lab (historique navigateur, fichiers récents, Shell Bags, etc.), venant renforcer les informations obtenues via Sysmon et KAPE.

---

### Reconstruction de la chaîne d'attaque

En regroupant les différentes sources de preuves, j'ai pu reconstruire la chaîne complète de l'incident.

```text
explorer.exe
        │
        ▼
powershell.exe
        │
        ▼
Invoke-AtomicTest
        │
        ▼
procdump64.exe
        │
        ▼
lsass.exe
```

Même si toutes ces étapes ne sont pas visibles dans une seule et même source, leur corrélation permet de reconstituer fidèlement le scénario exécuté pendant le lab.

---

### Identification du Patient Zero

La dernière étape consistait à identifier le premier événement ayant déclenché toute la chaîne.

Au vu des différentes preuves collectées, le **Patient Zero** est l'exécution de **PowerShell** lançant **Invoke-AtomicTest**.

Cette commande entraîne ensuite :

- l'exécution de ProcDump ;
- l'accès mémoire à LSASS ;
- la génération de l'Event ID 10 dans Sysmon ;
- puis la détection par la règle Sigma développée au Jour 5.

---

## Comment ça s'est passé

Cette journée a surtout consisté à **mettre en relation des preuves déjà collectées** plutôt qu'à découvrir de nouveaux artefacts.

Au début, je cherchais à retrouver toute la chaîne d'attaque directement dans Volatility. En réalité, je me suis rapidement rendu compte qu'aucun outil ne donne à lui seul toute l'histoire d'un incident.

Volatility montre ce qui est encore présent en mémoire au moment de la capture.

Sysmon raconte ce qui s'est produit pendant l'exécution.

KAPE extrait les principaux artefacts du système.

Autopsy permet d'explorer ces artefacts sous un autre angle.

C'est uniquement en croisant ces différentes sources que la chronologie devient réellement claire.

Autre difficulté rencontrée : identifier le *Patient Zero* n'était finalement pas une simple question de "premier processus observé". Il a fallu raisonner sur le scénario complet exécuté pendant le lab.

Même si `procdump64.exe` est l'action directement responsable de l'accès à LSASS, il n'est que la conséquence de l'exécution précédente de PowerShell via `Invoke-AtomicTest`.

Cette distinction est importante : le premier événement malveillant n'est pas forcément celui qui produit l'indicateur de compromission le plus visible.

---

## ✅ Points validés aujourd'hui

- [x] Analyse mémoire réalisée avec Volatility 3
- [x] Liste des processus analysée (`windows.pslist`)
- [x] Arbre des processus analysé (`windows.pstree`)
- [x] Connexions réseau vérifiées (`windows.netscan`)
- [x] Lignes de commande récupérées (`windows.cmdline`)
- [x] Recherche d'injection mémoire effectuée (`windows.malfind`)
- [x] Corrélation avec les journaux Sysmon du Jour 7
- [x] Corrélation avec les artefacts KAPE du Jour 8
- [x] Corrélation avec les analyses Autopsy du Jour 8
- [x] Reconstruction complète de la chaîne d'attaque
- [x] Patient Zero identifié et documenté

---

## 📌 Notes pour la suite

- La mémoire seule ne permet pas de reconstruire un incident complet : elle doit toujours être corrélée avec les journaux système et les artefacts disque.
- La complémentarité entre Volatility, Sysmon, KAPE et Autopsy est le principal enseignement de cette journée.
- Les étapes réalisées depuis le Jour 5 forment désormais une chaîne complète : détection (Sigma), réponse à incident, collecte de preuves, analyse disque et analyse mémoire.
- Cette chronologie complète servira directement de base au rapport d'incident final prévu à la fin du parcours.
