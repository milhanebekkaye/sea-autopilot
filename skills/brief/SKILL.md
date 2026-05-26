---
name: brief
description: Génère le brief SEA quotidien complet en analysant tous les canaux (Google Ads, Meta Ads, WMS). Orchestre les skills google-ads, meta-ads et wms pour produire un rapport consolidé avec un plan d'actions priorisé. Utilise ce skill comme point d'entrée principal pour toute analyse SEA.
---

# SEA Brief Skill

## Objectif
Produire un brief SEA complet, clair et actionnable en orchestrant l'analyse de tous les canaux. L'humain doit pouvoir lire ce brief en moins de 5 minutes et savoir exactement quoi faire.

## Arguments acceptés
Ce skill accepte des arguments pour personnaliser l'analyse :

- `/sea:brief` → Brief standard tous canaux, fenêtres 7/30/90 jours
- `/sea:brief complet` → Brief détaillé avec tous les sous-détails de chaque canal
- `/sea:brief google` → Uniquement Google Ads
- `/sea:brief facebook` → Uniquement Meta Ads
- `/sea:brief wms` → Uniquement WMS
- `/sea:brief 30` → Focus sur les 30 derniers jours uniquement
- `/sea:brief 7` → Focus sur les 7 derniers jours uniquement

## Exécution

### Étape 1 — Déterminer le périmètre
Lis l'argument fourni et détermine :
- Quels canaux analyser (tous par défaut, ou canal spécifique si précisé)
- Quelle fenêtre temporelle mettre en avant (7/30/90j par défaut)
- Niveau de détail (standard par défaut, détaillé si "complet")

### Étape 2 — Appeler les skills canaux
Pour chaque canal dans le périmètre, exécute le skill correspondant :
- Canal Google Ads → exécuter le skill google-ads
- Canal Meta Ads → exécuter le skill meta-ads
- Canal WMS → exécuter le skill wms

Récupère les outputs complets de chaque skill avant de passer à l'étape suivante.

### Étape 3 — Consolider et produire le brief
Assemble les outputs en suivant exactement le format défini ci-dessous.

## Format du brief

---

### 📊 Brief SEA — [Date du jour]
*Périmètre : [canaux analysés] — Fenêtres : [fenêtres analysées]*

---

### 1. Résumé exécutif
5 lignes maximum. Doit répondre à : est-ce qu'on est dans les clous globalement ?
- **ROAS global** : [moyenne pondérée tous canaux]
- **Volume leads** : [total 7 derniers jours] leads (vs [total 7j précédents])
- **Alertes critiques** : [nombre] alertes nécessitant une action aujourd'hui
- **Canaux en difficulté** : [liste ou "Aucun"]
- **Tendance générale** : ↑ En progression / → Stable / ↓ En recul

---

### 2. Alertes critiques 🚨
*Uniquement les situations nécessitant une action aujourd'hui.*
*Si aucune alerte → afficher : "✅ Aucune alerte critique — tout sous contrôle."*

Pour chaque alerte :
**[Canal] — [Campagne/Formation/Adset]**
- Situation : [description courte et chiffrée]
- Action requise : [action concrète]
- Signal : [sur combien de fenêtres]

---

### 3. Analyse par canal

#### 🔵 Google Ads — [Statut emoji]
[Tableau campagnes depuis skill google-ads]
[Résumé en 2 phrases]

#### 🟣 Meta Ads — [Statut emoji]
[Tableau campagnes agrégées depuis skill meta-ads]
[Adsets problématiques si présents]
[Résumé en 2 phrases]

#### 🟡 WMS — [Statut emoji]
[Tableau formations depuis skill wms]
[Résumé en 2 phrases]

*Note : section omise si canal non inclus dans le périmètre.*

---

### 4. Plan d'actions
*Liste consolidée de toutes les recommandations, classées par priorité.*
*Maximum 10 actions. Au-delà, regrouper les actions similaires.*
*Pas de suranalyse — uniquement ce qui mérite une action concrète.*

**Format de chaque action :**
`[Priorité] [Canal] — [Campagne/Adset/Formation] → [Action] | [Justification courte]`

**Priorités :**
- 🔴 Urgent — agir aujourd'hui
- 🟡 Important — agir cette semaine
- 🔵 Optimisation — agir quand possible

**Exemple :**
- 🔴 Meta — LSF | Stack → Couper cet adset | ROAS 0.4 sur 30j, signal fort 3 fenêtres
- 🟡 Google — Campagne ACACED → Réduire bid 15% | ROAS 1.2 en baisse sur 7j et 30j
- 🔵 WMS — Formateur → Pousser le volume | ROAS 2.3 stable, tendance ↑

---

### 5. Email WMS
*Affiché uniquement si des ajustements WMS sont recommandés.*
[Draft email depuis skill wms]

---

## Règles importantes

**Contre la suranalyse :**
- Ne pas lister une action si le signal est uniquement sur 7 jours
- Ne pas signaler les campagnes à faible volume (⚪) dans le plan d'actions principal
- Maximum 10 actions dans le plan — prioriser impitoyablement
- Si tout va bien sur un canal, le dire clairement en une phrase et passer au suivant

**Pour la clarté :**
- Chaque action doit être compréhensible sans lire le reste du brief
- Toujours justifier avec un chiffre, jamais avec une impression
- Le résumé exécutif doit suffire les jours où tout va bien
