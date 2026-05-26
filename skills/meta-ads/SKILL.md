---
name: meta-ads
description: Analyse les performances Meta Ads depuis Metabase. Utilise ce skill pour analyser les campagnes et adsets Meta (Facebook), détecter les adsets qui sous-performent au sein d'une campagne globalement correcte, diagnostiquer les problèmes SEA vs Sales, et formuler des recommandations actionnables.
---

# Meta Ads Analysis Skill

## Objectif
Analyser les performances Meta Ads à deux niveaux — campagne et adset — sur 3 fenêtres temporelles. Détecter les adsets qui plombent une campagne globalement correcte, diagnostiquer chaque situation, et produire des recommandations classées par priorité.

## Données à récupérer

Utilise le MCP Walter Learning pour interroger Metabase.

### Report principal — Lead Gen Custom Reporting (ID: 720)
Filtre sur vendor = "facebook"
Les lignes apparaissent au niveau adset avec le format : "1: [type] | 2: [formation] | 3: [audience]"
Récupère pour chaque adset :
- Spend (dépenses)
- Opportunities (leads générés)
- Won (ventes signées)
- Conversion %
- ROAS
- CPL (€)
- Estimated turnover

### Report volume — Leads Volume 2 (ID: 711)
Filtre sur lead_source = "f_ads"
Récupère le volume de leads par jour et par adset.

### Fenêtres temporelles
Effectue systématiquement l'analyse sur 3 périodes calculées depuis aujourd'hui :
- **7 jours** — signal court terme, anomalies immédiates
- **30 jours** — base de décision principale
- **90 jours** — validation long terme, filtre anti-saisonnalité

## Logique d'analyse

### Étape 1 — Regroupement par campagne
Avant l'analyse individuelle, regroupe les adsets par formation (niveau 2 du nom : "2: [formation]") pour calculer le ROAS agrégé de chaque campagne. Cela permet de détecter les cas où une campagne semble correcte globalement mais cache des adsets sous-performants.

### Étape 2 — Scoring ROAS
Applique le scoring à deux niveaux — campagne agrégée ET adset individuel :
- 🟢 VERT : ROAS > 2 → Performant, envisager de scaler
- 🟡 ORANGE : ROAS 1.5 à 2 → Satisfaisant, optimiser
- 🔴 ROUGE : ROAS 1.2 à 1.5 → Limite acceptable, action requise
- ⚫ CRITIQUE : ROAS < 1.2 ET (Spend > 500€ OU Leads > 20 sur 30j) → Action urgente
- ⚪ FAIBLE VOLUME : ROAS < 1.2 ET Spend < 500€ ET Leads < 20 → Signaler sans alarme
- 🚨 PRIORITÉ ABSOLUE : Won = 0 ET (Spend > 500€ OU Leads > 20 sur 30j) → Aucune vente malgré un volume significatif

### Étape 3 — Détection des adsets cachés
Pour chaque campagne avec un ROAS global correct (🟢 ou 🟡), vérifie si certains adsets individuels sont en ⚫ ou 🚨. Si oui, signaler explicitement : "La campagne [X] semble correcte globalement mais l'adset [Y] plombe les résultats."

### Étape 4 — Pondération multi-fenêtres
Avant toute recommandation, croise les 3 fenêtres :
- Anomalie sur 7 jours seulement → signal faible, surveiller sans agir
- Anomalie sur 7 et 30 jours → signal moyen, recommander une action
- Anomalie sur les 3 fenêtres → signal fort, action prioritaire

### Étape 5 — Diagnostic SEA vs Sales
Pour chaque adset sous-performant :
- CPL élevé + Taux conversion Sales correct → 🔧 Problème SEA — ajuster budget ou CPL cible, couper si trop critique
- CPL correct + Taux conversion Sales faible → 📞 Problème Sales — signaler uniquement, ne pas toucher
- CPL élevé + Taux conversion Sales faible → 🚨 Double problème — situation critique
- CPL correct + Taux conversion Sales correct → ✅ OK — optimiser pour scaler

### Étape 6 — Recommandation par adset sous-performant
- ROAS limite → Ajuster budget ou CPL cible de l'adset
- ROAS critique avec signal fort → Couper l'adset
- CPL par défaut hérité de la campagne parente sauf exception explicite

### Étape 7 — Analyse des tendances
Pour chaque campagne, compare les fenêtres :
- ROAS en hausse entre 90j → 30j → 7j : tendance positive ↑
- ROAS stable : tendance neutre →
- ROAS en baisse : tendance négative ↓

## Format de sortie

### Statut global Meta Ads
Statut emoji + résumé en 2 phrases maximum.

### Tableau par campagne (agrégé)
| Campagne | ROAS 7j | ROAS 30j | ROAS 90j | CPL moy. | Conv. Sales | Statut | Tendance |

### Détail des adsets problématiques
Uniquement les adsets en 🚨, ⚫, ou 🔴 — pas besoin de lister tous les adsets corrects.
| Campagne | Adset | ROAS 30j | CPL | Conv. Sales | Statut | Diagnostic |

### Recommandations classées par priorité
**Urgentes** — adsets 🚨 et ⚫
**Importantes** — adsets 🔴
**À surveiller** — campagnes 🟡
**Faible volume** — ⚪ listés brièvement

Pour chaque recommandation urgente ou importante :
- Campagne et adset concernés
- Action concrète (ajuster budget, ajuster CPL cible, couper)
- Justification chiffrée
- Diagnostic SEA ou Sales
- Force du signal (fenêtres concernées)
