---
name: wms
description: Analyse les performances WMS depuis Metabase. Utilise ce skill pour analyser les campagnes WMS identifiées par URL de landing page, calculer le ROAS par formation avec les CPL fixes, et formuler des recommandations pousser/maintenir/couper.
---

# WMS Analysis Skill

## Objectif
Analyser les performances WMS sur 3 fenêtres temporelles. WMS est un provider à volume fixe — on ne contrôle pas le CPL, il est fixe par formation. Le seul levier est le volume : quelles formations pousser, maintenir ou couper.

## Contexte WMS
- Les campagnes WMS sont identifiées par vendor = "wms_g_ads" dans Metabase
- Elles sont identifiées par l'URL de la landing page, pas par un nom de campagne
- CPL fixe : ACACED = 31€ / Toutes les autres formations = 35€
- On ne peut pas ajuster le CPL — on ajuste uniquement le volume via email au prestataire

## Extraction de la formation depuis l'URL
Identifie la formation à partir des patterns dans l'URL :
- "acaced" → Formation ACACED (CPL : 31€)
- "lsf" ou "langue-des-signes" → Formation LSF (CPL : 35€)
- "formateur" → Formation Formateur (CPL : 35€)
- "certibiocide" ou "certibio" → Formation Certibiocide (CPL : 35€)
- Tout autre pattern → Formation inconnue, signaler (CPL : 35€ par défaut)

## Données à récupérer

Utilise le MCP Walter Learning pour interroger Metabase.

### Report principal — Lead Gen Custom Reporting (ID: 720)
Filtre sur vendor = "wms_g_ads"
Récupère pour chaque URL de landing page :
- Spend (dépenses — correspond au CPL fixe × nombre de leads)
- Opportunities (leads générés)
- Won (ventes signées)
- Conversion %
- ROAS
- CPL (€) — doit correspondre au CPL fixe défini ci-dessus
- Estimated turnover

### Report volume — Leads Volume 2 (ID: 711)
Filtre sur lead_source = "wms_g_ads"
Récupère le volume de leads par jour et par URL.

### Fenêtres temporelles
WMS a des volumes faibles — l'analyse multi-fenêtres est donc particulièrement importante ici.
Effectue systématiquement l'analyse sur 3 périodes calculées depuis aujourd'hui :
- **7 jours** — tendances récentes, signaux immédiats
- **30 jours** — base de décision principale
- **90 jours** — validation long terme indispensable pour les faibles volumes

## Logique d'analyse

### Étape 1 — Regroupement par formation
Regroupe les URLs par formation identifiée. Une même formation peut avoir plusieurs URLs — agréger les métriques.

### Étape 2 — Scoring ROAS par formation
- 🟢 VERT : ROAS > 2 → Pousser — augmenter le volume
- 🟡 ORANGE : ROAS 1.5 à 2 → Maintenir — volume stable
- 🔴 ROUGE : ROAS 1.2 à 1.5 → Surveiller — réduire le volume prudemment
- ⚫ CRITIQUE : ROAS < 1.2 ET Leads > 20 sur 30j → Couper ou réduire fortement
- ⚪ FAIBLE VOLUME : ROAS < 1.2 ET Leads < 20 → Insuffisant pour décider, surveiller
- 🚨 PRIORITÉ ABSOLUE : Won = 0 ET Leads > 20 sur 30j → Aucune vente, couper immédiatement

### Étape 3 — Pondération multi-fenêtres
Crucial pour WMS en raison des faibles volumes :
- Anomalie sur 7 jours seulement → signal faible, ne pas agir
- Anomalie sur 7 et 30 jours → signal moyen, réduire prudemment
- Anomalie sur les 3 fenêtres → signal fort, couper ou réduire fortement

Ne jamais recommander de couper une formation basé uniquement sur 7 jours.

### Étape 4 — Analyse des tendances
Pour chaque formation, compare les fenêtres :
- ROAS en hausse entre 90j → 30j → 7j : tendance positive ↑ — signal pour pousser
- ROAS stable : tendance neutre → — maintenir
- ROAS en baisse entre 90j → 30j → 7j : tendance négative ↓ — signal pour réduire

### Étape 5 — Diagnostic SEA vs Sales
Même si le CPL est fixe, le diagnostic reste utile pour identifier si le problème vient de la qualité des leads WMS ou de la conversion commerciale :
- Taux conversion Sales faible toutes formations confondues → 📞 Problème Sales global
- Taux conversion Sales faible sur une formation spécifique → 📞 Problème Sales ciblé — signaler sans couper
- ROAS faible + Taux conversion Sales correct → 🔧 CPL trop élevé pour cette formation, envisager de couper

## Format de sortie

### Statut global WMS
Statut emoji + résumé en 2 phrases maximum.
Inclure le volume total de leads WMS sur 30 jours.

### Tableau par formation
| Formation | CPL fixe | Leads 30j | ROAS 7j | ROAS 30j | ROAS 90j | Conv. Sales | Statut | Tendance |

### Recommandations volume
Liste claire des actions à communiquer au prestataire :
- 🟢 **Pousser** : [formation] — ROAS [X] sur 30j, tendance ↑
- 🟡 **Maintenir** : [formation] — ROAS [X] stable
- 🔴 **Réduire** : [formation] — ROAS [X] sur 30j, signal [fort/moyen]
- ⚫ **Couper** : [formation] — ROAS [X], signal fort sur 3 fenêtres

### Draft email prestataire
Rédige automatiquement un email court et professionnel résumant les ajustements de volume à communiquer au prestataire WMS.
Format :
- Objet : Ajustements WMS — [date du jour]
- Corps : liste des formations à pousser/maintenir/réduire/couper avec justification courte
- Ton : professionnel et direct
