<div align="center">

# 📓 SOC Lab — Carnet de bord  
### Jour 11 — Analyse de malware (statique)  
<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 06/08/2026  
**Objectif du jour :** réaliser une analyse statique d’un échantillon malware récupéré sur MalwareBazaar, extraire les indicateurs de compromission (IOC), identifier le format et les capacités du binaire, puis répondre aux questions d’analyse afin de documenter les observations dans un rapport d’investigation.  

**Environnement du lab :** Kali (`kali`/`kali`) isolée (pas d’accès Internet sortant), machine hôte pour la documentation — voir carnet Jour 1 pour le détail de l’architecture.  

**Échantillon analysé :**
```text
11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

---

## ✅ Ce qui a été fait aujourd’hui

### Vérification de l’isolation réseau
Avant toute manipulation, j’ai confirmé que la VM Kali n’avait aucun accès Internet sortant :

```bash
ping 8.8.8.8
```
→ `Network is unreachable`

En revanche, le ping vers une adresse locale (10.10.10.10) fonctionnait, ce qui valide que l’isolation est bien en place.

---

### Calcul du hash SHA256
```bash
sha256sum 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

Résultat :
```text
11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328
```

Ce hash constitue le premier IOC principal de l’échantillon.

---

### Identification du format du fichier
```bash
file 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

Résultat :
```text
ELF 32-bit LSB executable, ARM, version 1 (ARM), dynamically linked, interpreter /lib/ld-uClibc.so.0, stripped
```

Le binaire n’est donc **pas** un exécutable Windows (PE) mais un **ELF 32 bits ARM**.  
PEStudio (outil prévu initialement dans le parcours) n’étant pas adapté aux ELF, l’analyse a été adaptée avec les outils Linux natifs.

---

### Extraction des chaînes de caractères
```bash
strings -n 8 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

Chaînes notables observées :
- `SNQUERY: 127.0.0.1:AAAAAA:xsvr`
- `M-SEARCH * HTTP/1.1`
- `HOST: 255.255.255.255:1900`
- `MAN: "ssdp:discover"`
- `ST: urn:dial-multiscreen-org:service:dial:1`
- `USER-AGENT: Google Chrome/60.0.3112.90 Windows`
- Mentions de `TeamSpeak`, `Windows XP`, `nickname`, `SPOOFEDHASH`
- Interpréteur : `/lib/ld-uClibc.so.0`
- Bibliothèque : `libc.so.0`

Aucune URL HTTP/HTTPS ni domaine Internet classique n’a été trouvé.  
Les chaînes SSDP/UPnP indiquent une capacité de découverte d’équipements sur le réseau local.

---

### Analyse de l’en-tête ELF et des sections
```bash
readelf -h 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
readelf -S 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

Points clés :
- Architecture : ARM
- Type : EXEC (exécutable)
- Point d’entrée : `0x8e60`
- Sections classiques présentes : `.text`, `.rodata`, `.data`, `.bss`, `.dynamic`, `.got`, `.plt`, etc.
- Binaire **stripped** (symboles de debug absents)

Rien n’indique un packing évident (pas de section UPX, pas d’entropie anormalement élevée visible, structure ELF standard).

---

### Analyse des imports dynamiques
```bash
objdump -T 11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328.elf
```

Bibliothèque principale : `libc.so.0`

Fonctions réseau et système importées (extrait) :
- `socket`, `connect`, `bind`, `listen`, `accept`
- `send`, `recv`, `sendto`, `recvfrom`
- `setsockopt`, `getsockopt`, `getsockname`
- `fork`, `prctl`, `setsid`
- `open`, `close`, `read`, `write`
- `malloc`, `calloc`, `realloc`, `free`
- `strcpy`, `strstr`, `strcasestr`, `inet_addr`

Ces imports montrent clairement des capacités de communication réseau et de manipulation de processus.

---

### Tableau des IOC extraits

| IOC              | Valeur                                                                 |
|------------------|------------------------------------------------------------------------|
| SHA256           | `11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328`   |
| Architecture     | ELF 32-bit ARM, dynamiquement lié, stripped                            |
| Interpréteur     | `/lib/ld-uClibc.so.0`                                                  |
| Bibliothèque     | `libc.so.0`                                                            |
| Adresse IP       | `127.0.0.1` (présente dans une chaîne)                                 |
| Protocole        | SSDP / UPnP (`M-SEARCH * HTTP/1.1`, `HOST: 255.255.255.255:1900`)     |
| Domaines / URLs  | Aucun trouvé                                                           |
| Chemins Windows  | Aucun (binaire Linux)                                                  |
| Mutex            | Aucun identifié                                                        |

---

### Réflexions

**Quel est le SHA256 ?**  
`11f14d6270f713437c36e92de64afa6f3149ff35e11f3a6f59346ffde45e8328`

**Le fichier semble-t-il packé ?**  
Non. Structure ELF classique, sections normales, binaire dynamiquement lié. Le fait qu’il soit *stripped* est courant sur Linux et ne constitue pas une preuve de packing.

**Quelles DLL importe-t-il ?**  
Sur Linux on parle de bibliothèques partagées. Principale : `libc.so.0`.  
Imports réseau et système nombreux (socket, connect, send, recv, fork, etc.).

**Contient-il des URLs ou des domaines ?**  
Non.  
Présence en revanche de chaînes SSDP/UPnP destinées à la découverte d’appareils sur le réseau local.

**Quels chemins Windows apparaissent ?**  
Aucun.  
Seul chemin significatif : `/lib/ld-uClibc.so.0`.

**Crée-t-il une persistance ?**  
Impossible à confirmer en analyse purement statique. Aucun script init, service systemd, cron ou chemin de persistance visible dans les chaînes.

**Quels IOC peuvent être utilisés dans un SIEM ou un EDR ?**
- Hash SHA256
- Architecture ELF ARM + interpréteur uClibc
- Imports réseau (socket/connect/send/recv…)
- Chaînes SSDP (`M-SEARCH * HTTP/1.1`, `HOST: 255.255.255.255:1900`)
- Adresse `127.0.0.1` dans le contexte SNQUERY

---

## Comment ça s’est passé

Le parcours prévoyait initialement l’analyse d’un exécutable Windows avec PEStudio. L’échantillon récupéré sur MalwareBazaar s’est avéré être un **ELF 32 bits ARM**.  

J’ai donc adapté la méthodologie en utilisant les outils Linux standard (`file`, `sha256sum`, `strings`, `readelf`, `objdump`).  

Cette adaptation a été instructive : elle montre qu’un analyste SOC doit savoir s’adapter au format réel de l’échantillon plutôt que de suivre rigidement un outil prévu pour un autre type de binaire.  

L’analyse dynamique a volontairement été ignorée (étape indiquée comme optionnelle) afin de rester dans le cadre d’une analyse 100 % statique et sécurisée.

---

## ✅ Points validés aujourd’hui

- [x] Récupération d’un échantillon depuis MalwareBazaar  
- [x] Vérification de l’isolation réseau de la VM  
- [x] Calcul du hash SHA256  
- [x] Identification du format ELF 32-bit ARM  
- [x] Extraction des chaînes de caractères (`strings`)  
- [x] Analyse de l’en-tête et des sections (`readelf`)  
- [x] Analyse des imports dynamiques (`objdump -T`)  
- [x] Extraction et documentation des IOC  
- [x] Réflexions  
- [x] Analyse dynamique volontairement non réalisée (optionnelle)

---

## 📌 Notes pour la suite

- Toujours commencer par `file` et `sha256sum` avant toute autre analyse.  
- Adapter les outils au format réel de l’échantillon (PE → PEStudio / Detect It Easy ; ELF → readelf / objdump / strings).  
- Les chaînes SSDP/UPnP et les imports réseau sont des indicateurs intéressants pour une détection comportementale.  
- Conserver l’habitude de documenter systématiquement le hash, l’architecture et les imports dans le carnet de bord.  
- Prochaine étape possible : réaliser une analyse dynamique contrôlée dans une VM isolée si un nouvel échantillon le justifie.
