📘 Documentation Technique : Pipeline de Nettoyage et Analyse des Vins
Projet : Wine Analytics ETL
Outil : Apache Hop
Date : Février 2026

# Documentation technique — Pipeline "Wine Analytics ETL"

- **Projet**: Wine Analytics ETL
- **Outil**: Apache Hop
- **Date**: Octobre 2023

## Vue d'ensemble

Ce pipeline ETL automatise le traitement des données brutes de dégustation de vins en appliquant des contrôles de qualité stricts. Objectifs principaux :

- Ingestion d'un fichier plat (CSV)
- Nettoyage des lignes incomplètes
- Isolation des rejets pour audit
- Tri des résultats par pertinence
- Export final en format Excel exploitable par le métier

## Architecture du flux

Le flux principal suit un chemin linéaire pour les données valides ("happy path") et dérive les anomalies vers une sortie de rejet.

![alt text](image.png)
![alt text](image-1.png)

## Détail des étapes de transformation

### Étape 1 — Ingestion des données (Lecture)

- **Type** : `CSV File Input`
- **Action technique** : Lecture du fichier source `winemag-data.csv` (séparateur `,`). Reconnaissance et conversion des types (string, integer pour `price` et `points`).
- **Justification** : Centraliser la donnée brute et garantir l'utilisation de la version la plus récente.

### Étape 2 — Contrôle qualité & filtrage

- **Type** : `Filter rows`
- **Règle** : `Price IS NOT NULL`
- **Comportement** :
	- Si la condition est vraie : la ligne suit le flux principal vers le tri.
	- Si la condition est fausse : la ligne est déviée vers la sortie d'erreur (rejet).
- **Justification** : Les analyses nécessitent un prix ; on exclut les lignes sans prix du rapport final tout en conservant un fichier de rejets pour audit et correction.

### Étape 3 — Gestion des rejets

- **Type** : `Text file output` / `CSV`
- **Action technique** : Écriture des lignes rejetées dans un fichier de rejets pour traçabilité.
- **Justification** : Auditabilité et possibilité de réconciliation ou correction des données manquantes.

### Étape 4 — Tri stratégique

- **Type** : `Sort rows`
- **Configuration** : Tri par `points` en ordre décroissant.
- **Justification** : Mettre en avant les meilleurs vins pour faciliter la lecture métier.

### Étape 5 — Livraison du rapport

- **Type** : `Microsoft Excel Writer`
- **Action technique** : Export des données nettoyées et triées dans un fichier `.xlsx`.
- **Justification** : Format standard pour diffusion aux équipes Marketing / Ventes / Direction.

## Bilan et valeur ajoutée

Ce pipeline remplace un processus manuel long et sujet aux erreurs humaines par une exécution automatisée, traçable et reproductible.

| Avant (manuel) | Après (pipeline ETL Hop) |
|---|---|
| Risque d'oubli des suppressions et erreurs manuelles | Fiabilité élevée grâce à des règles explicites |
| Perte ou suppression silencieuse de données | Traçabilité via fichier de rejets |
| Tri manuel à chaque mise à jour | Tri automatique et immédiat |

---

Pour aller plus loin : insérer la capture d'écran du pipeline dans `./images/architecture.png` et préciser les chemins d'entrée/sortie (ex : `data/winemag-data.csv`, `output/wine_report.xlsx`, `output/rejects.csv`).


2. Architecture du Flux
(Insérer ton screenshot ici)
Le flux est linéaire pour les données valides ("Happy Path") avec une dérivation latérale pour la gestion des anomalies.

3. Détail des Étapes de Transformation
Étape 1 : Ingestion des Données (Lecture wine)

Type de transformation : CSV File Input
Action Technique : Lecture du fichier source winemag-data.csv. Configuration du délimiteur (,) et reconnaissance des types de données (String, Integer pour les prix et points).
Justification Business : Centralisation de la donnée brute. Cette étape assure que nous travaillons toujours sur la version la plus récente des données fournies.

Étape 2 : Contrôle Qualité & Filtrage (Filtre prix manquant)

Type de transformation : Filter rows
Règle de gestion : Price IS NOT NULL
Fonctionnement : Le pipeline vérifie chaque ligne pour voir si un prix est renseigné.
Flux conditionnel (La force de ce pipeline) :
✅ Vrai (True) : Si le prix existe, la donnée continue vers le tri.
❌ Faux (False) : Si le prix est vide, la donnée est déviée vers la sortie erreur.


Justification Business : Une analyse financière ou commerciale est impossible sans prix. Supprimer ces lignes du rapport final évite de fausser les moyennes, mais les conserver à part permet de remonter l'information à l'équipe IT ou Saisie pour correction.

Étape 3 : Gestion des Rejets (erreur)

Type de transformation : Text file output (ou CSV)
Action Technique : Captation de toutes les lignes ayant échoué au test de l'étape 2.
Justification Business : Auditabilité. Plutôt que de supprimer silencieusement les données, nous créons un fichier de "rejets". Cela permet de dire : "Nous avons reçu 1000 lignes, 950 sont dans le rapport, 50 sont dans le fichier d'erreur car incomplètes". C'est une preuve de rigueur.

Étape 4 : Ordonnancement Stratégique (Tri par note)

Type de transformation : Sort rows
Configuration : Tri sur la colonne points en ordre Descendant (du plus grand au plus petit).
Justification Business : Ergonomie décisionnelle. Le fichier final doit mettre en avant l'excellence. Les décideurs voient immédiatement les "Top Vins" (notes 100/100) en haut du fichier sans avoir à manipuler Excel.

Étape 5 : Livraison du Rapport (Sortie)

Type de transformation : Microsoft Excel Writer
Action Technique : Écriture des données propres et triées dans un fichier .xlsx natif.
Justification Business : Interopérabilité. Le format Excel est le standard universel pour le partage d'informations avec les équipes métier (Marketing, Ventes, Direction).


4. Bilan et Valeur Ajoutée
Ce pipeline remplace un traitement manuel qui serait fastidieux et source d'erreurs humaines.
Copier le tableau


Avant (Manuel)
Après (Pipeline ETL Hop)



Risque d'oubli de suppression des prix vides
Fiabilité 100% (Règle stricte)


Perte des données supprimées
Traçabilité (Fichier d'erreurs conservé)


Tri manuel à chaque mise à jour
Automatisation (Tri instantané)


Temps de traitement : ~30 min
Temps de traitement : < 2 secondes