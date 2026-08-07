<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 12 — Surveillance de sécurité réseau

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 06-07/08/2026

**Objectif du jour :** activer le mode promiscuous sur le segment isolé, déployer Suricata en mode IDS avec le ruleset ET Open sur Kali, capturer du trafic avec Wireshark en parallèle, puis corréler toute alerte générée avec les paquets exacts observés.

**Environnement du lab :** Kali (`10.10.10.30`), Tiny10 (`10.10.10.10`), Ubuntu Server (`10.10.10.20`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

### Mode promiscuous sur ESXi

Activé sur le port group isolé (Security → Promiscuous Mode → Accept), pour que Kali puisse observer l'ensemble du trafic du segment, pas seulement celui qui lui est destiné directement.

### Installation et configuration de Suricata

Suricata était déjà présent sur Kali. Mise à jour du ruleset ET Open :

```bash
sudo suricata-update
```

**Confirmation du chargement des règles :**

```bash
sudo grep "rules successfully loaded" /var/log/suricata/suricata.log
```

Résultat : **52 220 règles chargées, 0 échec, 0 ignorée** — le ruleset complet est opérationnel.

### Lancement de Suricata — plusieurs itérations nécessaires

Le lancement propre a demandé plusieurs corrections successives (détaillées dans la section Troubleshooting) : mauvaise valeur passée en interface, conflit entre lancement manuel et service systemd, pidfile orphelin. Une fois stabilisé via `systemctl`, confirmation finale :

```bash
sudo systemctl status suricata
```

```
Active: active (running) since Fri 2026-08-07 09:31:30 CEST
Main PID: 11928 (Suricata-Main)
```

**Interface confirmée correcte :**

```bash
sudo grep -A2 "af-packet" /etc/suricata/suricata.yaml
```
```
af-packet:
  - interface: eth0
```

### Génération de trafic et capture

Contrainte du lab à respecter : les VMs restent isolées d'Internet en permanence — donc pas de test générant du trafic externe (comme le téléchargement Mimikatz du Jour 4/7). J'ai généré du trafic **interne au lab**, plus représentatif d'un scénario de reconnaissance/mouvement latéral :

```bash
ping 10.10.10.10   # Kali → Tiny10
ping 10.10.10.20   # Kali → Ubuntu Server
```

En parallèle, capture Wireshark sur l'interface `eth0`, exportée en `070826.pcapng`.

### Résultat de la capture

**855 paquets capturés au total, dont 213 ICMP**, sur une fenêtre de 08:03 à 08:32 (UTC, soit 10:03-10:32 CEST — cohérent avec les horodatages du terminal).

Le pcap confirme du trafic ICMP bidirectionnel réel entre les trois machines :
- `10.10.10.30 (Kali) ↔ 10.10.10.10 (Tiny10)` — échanges echo-request/echo-reply réussis.
- `10.10.10.20 (Ubuntu) ↔ 10.10.10.30 (Kali)` — échanges réussis, entrecoupés de plusieurs `Destination Host Unreachable` (ICMP type 3) au début de la capture, correspondant exactement aux pertes de paquets observées dans le terminal (`72,8% packet loss` initialement, avant stabilisation de la route).

### Corrélation alerte Suricata ↔ paquets Wireshark

En comparant `fast.log` avec la fenêtre de capture du pcap : **aucune alerte Suricata spécifique n'est liée à ce trafic ICMP.** Seule l'alerte récurrente "ET INFO Possible Kali Linux hostname in DHCP Request Packet" apparaît en continu dans les logs, sans lien avec les pings effectués.

**Interprétation :** ce n'est pas un échec de la chaîne de capture, mais un résultat cohérent et attendu. Un simple ping ICMP n'a rien de malveillant en soi — aucune règle ET Open standard n'est censée se déclencher dessus, sinon l'IDS serait en alerte permanente sur du trafic totalement bénin. La véritable validation du Jour 12 n'est donc pas "une alerte a été trouvée", mais **"le pipeline complet fonctionne correctement de bout en bout"** : mode promiscuous actif, Suricata opérationnel avec ruleset complet, Wireshark capturant fidèlement le trafic multi-VM, et surtout — le comportement de non-détection sur du trafic bénin est lui-même la preuve qu'un IDS bien réglé ne crie pas au loup pour rien.

---

## Comment ça s'est passé

Journée avec beaucoup de troubleshooting réseau/système, plus que de découvertes offensives — logique pour du Jour 12 orienté infrastructure.

### Confusion interface réseau vs adresse IP

Première erreur : avoir lancé Suricata avec `-i 10.10.10.30` (l'adresse IP de Kali) au lieu du nom de l'interface (`eth0`). Suricata capture au niveau de l'interface réseau, pas d'une IP — la commande ne plantait pas franchement, mais n'écoutait rien de valide. Corrigé en identifiant le bon nom via `ip a`.

### Conflit entre lancement manuel et service systemd

Après avoir basculé sur `systemctl start suricata`, une tentative de relancer Suricata manuellement en parallèle (`sudo suricata -i eth0 -D`) a provoqué :
```
E: pidfile: pid file '/var/run/suricata.pid' exists but appears stale.
```
Cause : deux méthodes de lancement différentes (manuelle vs systemd) qui se marchent dessus via le même fichier pid. Résolu en arrêtant proprement le service (`systemctl stop`), supprimant le pidfile orphelin, puis en ne gardant plus **qu'une seule méthode de lancement** (systemd) pour le reste de la journée.

**Leçon retenue :** ne jamais mélanger lancement manuel et service systemd pour le même daemon — choisir une méthode et s'y tenir.

### Déconnexions réseau intermittentes sur Kali

Plusieurs `ping: connect: Network is unreachable` sont apparus en cours de session, sans action de ma part. Diagnostic : le **NetworkManager** de Kali effectue périodiquement des vérifications de connectivité Internet, qui échouent en boucle puisque le lab est volontairement isolé — ce qui perturbait la stabilité de la connexion locale. Confirmé le lien avec le bruit DHCP répétitif déjà visible dans `fast.log` (Kali retentant régulièrement d'obtenir/renouveler une adresse).

**Correction :**
```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```
Ajout de `uri=` vide dans la section `[connectivity]`, puis :
```bash
sudo systemctl restart NetworkManager
```

### Rappel de contrainte oublié un instant

À un moment du guide, j'ai proposé de relancer un test générant du trafic vers Internet (téléchargement Mimikatz) — oubli temporaire de la contrainte fondamentale du lab (isolation totale, sauf fenêtres ponctuelles volontaires). Corrigé en basculant sur un test générant uniquement du trafic interne (ping entre VMs), plus cohérent avec l'architecture du lab et, avec le recul, plus pertinent pédagogiquement pour ce jour précis (le Jour 12 porte sur la surveillance du trafic *interne*, pas sur la détection d'exfiltration externe).

---

## ✅ Points validés aujourd'hui

- [x] Mode promiscuous activé sur le port group isolé (ESXi)
- [x] Suricata installé, ruleset ET Open à jour (52 220 règles)
- [x] Service Suricata stabilisé via systemd (après résolution des conflits pidfile/interface)
- [x] Déconnexions réseau NetworkManager diagnostiquées et corrigées
- [x] Trafic ICMP généré entre les 3 VMs du lab
- [x] Capture Wireshark réussie (855 paquets, 213 ICMP, fenêtre 08:03-08:32)
- [x] Corrélation tentée entre `fast.log` et le pcap
- [x] Absence d'alerte sur trafic bénin comprise et documentée comme résultat attendu, pas comme échec

---

## 📌 Notes pour la suite

- Toujours utiliser `ip a` pour confirmer le nom exact d'une interface avant de configurer un outil réseau — ne jamais supposer ou utiliser une IP à la place.
- Ne pas mélanger lancement manuel et service systemd pour un même daemon.
- La correction NetworkManager (`uri=` vide) est permanente — plus besoin d'y revenir pour les prochains jours du lab utilisant Kali en réseau isolé.
- Pour un vrai test de détection Suricata avec alerte positive, il faudrait générer du trafic correspondant à une signature ET Open connue (ex. un user-agent suspect, un pattern de commande C2 identifiable) — piste à explorer si le programme prévoit un exercice de tuning IDS plus poussé.
- Le pcap `070826.pcapng` et les logs Suricata sont conservés comme preuves pour le rapport final du Jour 14 ; à supprimer ensuite pour libérer l'espace disque, conformément à la consigne de nettoyage du Jour 12.
