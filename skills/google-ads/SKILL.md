---
name: google-ads
description: Analyse les performances Google Ads depuis Metabase. Utilise ce skill pour analyser les campagnes Google Ads, détecter les anomalies de ROAS, diagnostiquer les problèmes SEA vs Sales, et formuler des recommandations d'optimisation actionnables.
---

# Google Ads Analysis Skill

## Objectif
Analyser les performances Google Ads sur 3 fenêtres temporelles, diagnostiquer chaque campagne, et produire des recommandations classées par priorité.

## Données à récupérer

Utilise le MCP Walter Learning pour interroger Metabase.

### Report principal — Lead Gen Custom Reporting (ID: 720)
Filtre sur vendor = "google"
Récupère pour chaque campagne :
- Spend (dépenses)
- Opportunities (leads générés)
- Won (ventes signées)
- Conversion %
- ROAS
- CPL (€)
- Estimated turnover

### Report volume — Leads Volume 2 (ID: 711)
Filtre sur lead_source = "g_ads"
Récupère le volume de leads par jour et par campagne.

### Fenêtres temporelles
Effectue systématiquement l'analyse sur 3 périodes calculées depuis aujourd'hui :
- **7 jours** — signal court terme, anomalies immédiates
- **30 jours** — base de décision principale
- **90 jours** — validation long terme, filtre anti-saisonnalité

## Logique d'analyse

### Étape 1 — Scoring ROAS par campagne
Pour chaque campagne, évalue le ROAS sur les 3 fenêtres :
- 🟢 VERT : ROAS > 2 → Performant, envisager de scaler
- 🟡 ORANGE : ROAS 1.5 à 2 → Satisfaisant, optimiser
- 🔴 ROUGE : ROAS 1.2 à 1.5 → Limite acceptable, action requise
- ⚫ CRITIQUE : ROAS < 1.2 ET (Spend > 500€ OU Leads > 20 sur 30j) → Action urgente
- ⚪ FAIBLE VOLUME : ROAS < 1.2 ET Spend < 500€ ET Leads < 20 → Signaler sans alarme
- 🚨 PRIORITÉ ABSOLUE : Won = 0 ET (Spend > 500€ OU Leads > 20 sur 30j) → Aucune vente malgré un volume significatif

### Étape 2 — Pondération multi-fenêtres
Avant toute recommandation, croise les 3 fenêtres pour évaluer la fiabilité du signal :
- Anomalie sur 7 jours seulement → signal faible, surveiller sans agir
- Anomalie sur 7 et 30 jours → signal moyen, recommander une action
- Anomalie sur les 3 fenêtres → signal fort, action prioritaire

### Étape 3 — Diagnostic SEA vs Sales
Pour chaque campagne sous-performante, identifie la cause racine :
- CPL élevé + Taux conversion Sales correct → 🔧 Problème SEA — agir sur les campagnes
- CPL correct + Taux conversion Sales faible → 📞 Problème Sales — signaler uniquement, ne pas toucher aux campagnes
- CPL élevé + Taux conversion Sales faible → 🚨 Double problème — situation critique
- CPL correct + Taux conversion Sales correct → ✅ OK — optimiser pour scaler

### Étape 4 — Analyse des tendances
Pour chaque campagne, compare les fenêtres pour identifier la direction :
- ROAS en hausse entre 90j → 30j → 7j : tendance positive ↑
- ROAS stable sur les 3 fenêtres : tendance neutre →
- ROAS en baisse entre 90j → 30j → 7j : tendance négative ↓

## Format de sortie

### Statut global Google Ads
Statut emoji + résumé en 2 phrases maximum.

### Tableau par campagne
Pour chaque campagne active :
| Campagne | ROAS 7j | ROAS 30j | ROAS 90j | CPL | Conv. Sales | Statut | Tendance | Diagnostic |

### Recommandations classées par priorité
**Urgentes** — campagnes 🚨 et ⚫
**Importantes** — campagnes 🔴
**À surveiller** — campagnes 🟡
**Faible volume** — campagnes ⚪ (listées brièvement, sans développement)

Pour chaque recommandation urgente ou importante :
- Campagne concernée
- Action concrète suggérée (ajuster bid, revoir ciblage, couper, scaler)
- Justification chiffrée (ROAS sur quelle fenêtre, écart vs cible)
- Diagnostic SEA ou Sales
- Force du signal (fenêtres concernées)
