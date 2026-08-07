# Résumé exécutif — Incident simulé (laboratoire)

**Parcours :** CyberAtaraxia Careers — SOC Analyst  
**Document :** version direction / RSSI  
**Date :** 07/08/2026  
**Référence :** INC-LAB-2026-08-04-001  
**Statut :** **Jour final** — documentation de clôture pédagogique  

---

## Synthèse

Un **exercice contrôlé** a rejoué sur un poste Windows de laboratoire une enchaînement d’actions souvent vu lors d’incidents réels : document leurre, outils de récupération d’identifiants, et mécanisme pour survivre à une reconnexion.  

Les **journaux de sécurité** ont enregistré ces actions. Les **règles de détection** conçues pendant le parcours ont correctement signalé l’accès sensible au composant Windows qui gère les identifiants. Les éléments ajoutés pour la persistance ont été **retirés** et le poste a été **contrôlé**.  

Un fichier sensible issu d’un **test antérieur**, resté par erreur sur le disque, a été **découvert lors des vérifications** — illustrant la nécessité d’un nettoyage systématique après chaque exercice.

Un autre volet du parcours a porté sur un logiciel malveillant **Linux** de type **Mirai** (souvent lié aux objets connectés) : l’empreinte numérique du fichier est **déjà connue** des bases antivirus.

---

## Impact

| Contexte | Impact |
|----------|--------|
| **Laboratoire isolé** | Aucun impact sur le SI de production ni sur Internet |
| **Si les mêmes gestes survenaient en production** | Risque élevé d’exposition d’identifiants et de maintien d’accès ; risque organisationnel si les traces ne sont pas effacées après traitement |

---

## Messages clés pour la direction

1. La **détection fonctionne** sur ce scénario d’accès aux identifiants, grâce à la journalisation fine et aux règles testées.  
2. La **réponse** doit toujours inclure une **vérification post-nettoyage** (fichiers, redémarrage automatique, comptes).  
3. Le profil global des exercices correspond à des **menaces courantes et opportunistes**, pas à une campagne ultra-ciblée de type APT — les priorités doivent porter sur les **comportements** répétés, pas seulement sur des fichiers isolés.  
4. Le livrable final (rapports, playbook, registre d’indicateurs, règles) renforce la **capacité à reproduire** investigation et reporting en conditions réelles.

---

## Décision / suite

- Clôture de l’exercice de lab sur le plan documentaire (**jour final**).  
- Conservation du registre d’indicateurs et des règles de détection pour le portfolio et d’éventuels exercices de hunting.  
- Pas de mesure d’urgence production requise (environnement d’apprentissage uniquement).

---

*Document d’une page — sans détail d’exploitation technique.*
