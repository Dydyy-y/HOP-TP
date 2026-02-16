# LAB 3.1 — Pipeline robuste Taxi

## 🎯 Objectif pédagogique

Construire un pipeline Apache Hop robuste pour :

- Ingestion d’un dataset Taxi (CSV)
- Nettoyage des données
- Validation des champs critiques
- Rejet des lignes invalides
- Séparation des flux "clean" et "errors"

À la fin du lab vous devrez comprendre comment structurer un pipeline industriel, gérer les erreurs et rendre un pipeline rejouable.

## 🧠 Contexte

Nous travaillons sur un extrait du dataset NYC Taxi. Certaines lignes peuvent contenir :

- valeurs nulles
- dates incohérentes
- montants négatifs
- coordonnées invalides

Votre mission est de construire un pipeline qui nettoie, valide, sépare et exporte les données conformément à la structure de dossiers ci‑dessous.

## 📂 Structure attendue

data/
├── raw/
├── clean/
└── rejected/

Exemples de chemins relatifs au projet : `TP/Jour3/Lab1/data/raw/`, `TP/Jour3/Lab1/data/clean/`, `TP/Jour3/Lab1/data/rejected/`.

## 🧪 Étapes détaillées

1) Ingestion CSV

- Créer un nouveau pipeline Hop.
- Ajouter une étape `CSV File Input`.
- Lire le fichier depuis `data/raw/`.
- Vérifier l'encodage, le séparateur et le mapping des colonnes.

2) Typage & Normalisation

- Ajouter `Select Values` pour caster les types.
- Utiliser `Date format` et `Number format` si nécessaire.
- Champs critiques à vérifier : `pickup_datetime`, `dropoff_datetime`, `total_amount`, `passenger_count`.

3) Règles de validation

- Ajouter une étape `Filter Rows` (ou équivalent).
- Règles recommandées :
  - `total_amount > 0`
  - `passenger_count > 0`
  - `pickup_datetime` NOT NULL
  - `dropoff_datetime` NOT NULL
- Séparer les flux :
  - Flux valide -> vers `clean`
  - Flux invalide -> vers `rejected`

4) Gestion d’erreur

- Activer l’`Error handling` sur les transformations critiques.
- Capturer les lignes qui échouent et enrichir chaque ligne rejetée avec une colonne `error_reason` expliquant la cause (ex : `missing_pickup_datetime`, `negative_amount`).

5) Export

- Flux valide : `CSV Output` → `data/clean/taxi_clean.csv`
- Flux invalide : `CSV Output` → `data/rejected/taxi_rejected.csv`

## 🔧 Bonnes pratiques

- Rendre le pipeline idempotent et rejouable :
  - utiliser `batch_id` et `processing_timestamp` pour chaque enregistrement
  - écrire dans des fichiers horodatés ou écraser de manière contrôlée
- Journaliser les erreurs et métriques (nombre de lignes traitées / rejetées).
- Tester le pipeline avec échantillons (cas limités, cas limites, données corrompues).

## 🔍 Questions de réflexion

- Que se passe-t-il si une colonne change de nom ?
  - prévoir un mapping de colonnes configurable, ou valider l'en-tête avant ingestion.
- Que se passe-t-il si le fichier contient 10 millions de lignes ?
  - envisager un traitement par lots, streaming, partitionnement, et monitoring mémoire/IO.
- Votre pipeline est-il rejouable ?
  - oui si les sorties sont déterministes, si l'on gère les IDs de batch et si l'on évite les opérations non-déterministes sans checkpoint.

## 🎓 Bonus (optionnel)

- Ajouter automatiquement une colonne `processing_timestamp` (timestamp d'exécution).
- Ajouter une colonne `batch_id` (UUID ou date/heure) pour tracer un lot.

## ✅ Validation finale

Le pipeline doit :

- Ne jamais planter (s'assurer d'un error handling robuste)
- Séparer `clean` et `rejected`
- Être clair visuellement (noms d'étapes explicites)
- Être documenté (ce README + commentaires dans le pipeline)

---

Placez ce README dans le répertoire du lab : `TP/Jour3/Lab1/README.md`.

Bon travail — dites-moi si vous voulez que je génère un exemple de pipeline Hop ou un template XML pour démarrer.
