<div align="center">

# 📓 SOC Lab — Carnet de bord

### Jour 10 — Analyse de phishing

<sub>Partie du parcours <strong>CyberAtaraxia Careers — SOC Analyst</strong></sub>

</div>

---

## 🧭 Contexte

**Date :** 06/08/2026  
**Objectif du jour :** apprendre à analyser un email de phishing comme le ferait un analyste SOC, en étudiant les en-têtes SMTP, les mécanismes d'authentification (SPF, DKIM, DMARC), les indicateurs de compromission (IOC) et les techniques d'ingénierie sociale utilisées pour tromper la victime.

**Environnement du lab :** Kali (`kali`/`kali`), Ubuntu Server (`victim`/`victim`), Tiny10 (`victim`/`victim`) — voir carnet Jour 1 pour le détail de l'architecture.

---

## ✅ Ce qui a été fait aujourd'hui

### Création d'un email de phishing de test

Plutôt que d'utiliser une plateforme d'envoi de mails, j'ai choisi de construire manuellement un fichier **`.eml`** afin de maîtriser entièrement les en-têtes SMTP et les éléments du scénario.

Le fichier contient volontairement plusieurs incohérences caractéristiques d'un phishing :

- expéditeur affiché se faisant passer pour Microsoft ;
- domaine expéditeur différent du domaine officiel ;
- `Return-Path` pointant vers un serveur tiers ;
- échec SPF ;
- échec DKIM ;
- échec DMARC ;
- URL invitant la victime à confirmer son compte.

L'objectif n'était pas de reproduire un phishing parfaitement réaliste mais de disposer d'un exemple suffisamment riche pour pratiquer l'analyse des en-têtes.

---

### Analyse des en-têtes SMTP

Le fichier `.eml` a ensuite été ouvert afin d'inspecter manuellement les principaux champs utilisés lors d'une investigation.

#### Analyse du champ **From**

L'expéditeur affiché était :

```text
Microsoft Support
support@microsoft-secure-login.com
```

Premier constat : le domaine utilisé n'est pas un domaine officiel Microsoft (`microsoft.com` ou `office.com`) mais un domaine ressemblant fortement au domaine légitime.

Il s'agit d'une technique classique d'usurpation destinée à tromper rapidement un utilisateur peu attentif.

---

#### Analyse du **Return-Path**

Le champ indiquait :

```text
bounce@evil-mail.ru
```

Le serveur chargé de recevoir les réponses est donc totalement différent du domaine affiché dans le champ **From**.

Cette incohérence constitue déjà un premier indicateur fort de phishing.

---

#### Analyse de **Authentication-Results**

L'en-tête contenait volontairement :

```text
spf=fail
dkim=fail
dmarc=fail
```

Analyse :

- **SPF Fail** : le serveur ayant envoyé le message n'est pas autorisé à envoyer des emails pour ce domaine.
- **DKIM Fail** : la signature cryptographique du message est absente ou invalide.
- **DMARC Fail** : le domaine propriétaire rejette ce message car il ne respecte pas les politiques d'authentification.

La combinaison de ces trois échecs constitue un très fort indicateur de falsification.

---

#### Analyse des champs **Received**

Le premier serveur ayant traité le message était :

```text
smtp.evil-mail.ru
```

Le chemin emprunté par le message ne présente aucun lien avec une infrastructure Microsoft.

La comparaison entre :

- le domaine affiché à l'utilisateur ;
- le Return-Path ;
- les serveurs Received

permet rapidement de mettre en évidence l'usurpation.

---

### Identification des IOC

Plusieurs indicateurs de compromission ont été extraits du message.

Adresse email :

```text
support@microsoft-secure-login.com
```

URL :

```text
http://microsoft-secure-login.com/verify
```

Adresse IP :

```text
185.222.111.15
```

Ces éléments constituent les principaux IOC exploitables lors d'une investigation ou d'une campagne de Threat Hunting.

---

### Defang des IOC

Avant toute documentation, les IOC ont été neutralisés afin d'éviter tout clic accidentel.

Transformation de l'URL :

```text
http://microsoft-secure-login.com/verify
```

devient

```text
hxxp://microsoft-secure-login[.]com/verify
```

Transformation du domaine :

```text
microsoft-secure-login.com
```

devient

```text
microsoft-secure-login[.]com
```

Transformation de l'adresse IP :

```text
185.222.111.15
```

devient

```text
185[.]222[.]111[.]15
```

Cette pratique est systématiquement utilisée dans les rapports d'incident afin d'éviter l'activation involontaire d'un lien malveillant.

---

### Vérification de la réputation

Sans visiter directement l'URL dans un navigateur, la réputation du domaine a été vérifiée à l'aide d'un service spécialisé (VirusTotal ou urlscan.io).

Cette étape permet de déterminer rapidement si :

- le domaine est déjà connu comme malveillant ;
- l'URL a déjà été analysée ;
- d'autres moteurs antivirus la détectent.

L'objectif est d'enrichir l'investigation sans exposer le poste de travail à une éventuelle compromission.

---

### Analyse des techniques d'ingénierie sociale

Le contenu du message met en œuvre plusieurs techniques classiques de phishing.

**Usurpation d'identité**

L'attaquant se fait passer pour Microsoft afin de bénéficier immédiatement de la confiance de la victime.

**Sentiment d'urgence**

Le message indique que le compte sera suspendu dans un délai très court.

Cette pression temporelle pousse l'utilisateur à agir rapidement sans vérifier la légitimité du message.

**Menace de perte d'accès**

Le risque de suppression du compte est utilisé comme levier psychologique.

**Demande d'authentification**

La victime est invitée à cliquer sur un lien afin de "confirmer son identité", objectif classique des campagnes de vol d'identifiants.

---

## Comment ça s'est passé

Cette journée m'a permis de comprendre qu'une grande partie de l'analyse d'un phishing repose sur l'observation méthodique des en-têtes plutôt que sur le contenu du message lui-même.

Au premier regard, l'email paraît relativement crédible : logo implicite de Microsoft, ton professionnel, demande de confirmation de compte. Pourtant, quelques minutes d'analyse suffisent à mettre en évidence plusieurs incohérences majeures :

- domaine expéditeur trompeur ;
- Return-Path différent ;
- serveur d'origine sans lien avec Microsoft ;
- triple échec SPF, DKIM et DMARC.

J'ai également découvert l'intérêt du **defang**, pratique très répandue dans les rapports SOC mais que je n'avais encore jamais appliquée moi-même.

Enfin, cette journée montre qu'un analyste ne doit jamais cliquer directement sur un lien suspect : la réputation d'une URL peut être vérifiée via des services spécialisés avant toute interaction avec celle-ci.

---

## ✅ Points validés aujourd'hui

- [x] Création d'un email de phishing de test au format `.eml`
- [x] Analyse manuelle des principaux en-têtes SMTP
- [x] Identification d'un échec SPF
- [x] Identification d'un échec DKIM
- [x] Identification d'un échec DMARC
- [x] Comparaison entre le champ **From**, le **Return-Path** et les serveurs **Received**
- [x] Extraction des IOC (email, domaine, URL, adresse IP)
- [x] Defang des IOC avant documentation
- [x] Vérification de la réputation de l'URL via un service spécialisé
- [x] Identification des principales techniques d'ingénierie sociale utilisées dans le message

---

## 📌 Notes pour la suite

- Continuer à analyser d'autres exemples d'emails de phishing présentant différents niveaux de sophistication (Business Email Compromise, spear phishing, QR phishing, etc.).
- Approfondir le fonctionnement de SPF, DKIM et DMARC afin de mieux comprendre leurs limites et leurs contournements.
- Lors des prochains laboratoires, intégrer systématiquement les IOC extraits dans une chronologie d'incident afin de les corréler avec les journaux Sysmon et les autres artefacts forensiques étudiés les jours précédents.
- Conserver l'habitude de **defang** systématiquement tous les IOC avant toute publication dans un rapport ou un carnet de bord.
