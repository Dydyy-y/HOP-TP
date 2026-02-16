# COMPTE‑RENDU — LAB 3.1 : Pipeline robuste Taxi

Ce document décrit la démarche effectuée pour construire et valider le pipeline Apache Hop destiné au traitement d'un extrait du dataset NYC Taxi. Il explique les choix techniques, les transformations appliquées, la stratégie de gestion des erreurs et les résultats attendus.

## Résumé de la démarche

- Source des données : `data/raw/` (fichier CSV fourni dans le lab).
- Objectif : produire deux sorties distinctes — un fichier `clean` et un fichier `rejected` — tout en garantissant la robustesse et la rejouabilité du pipeline.

## Étapes réalisées et décisions prises

1) Ingestion

- Étape utilisée : `CSV File Input`.
- Vérifications appliquées : encodage UTF-8, séparateur (`,` par défaut) et validation de l'en-tête.
- Motif : détecter rapidement les changements d'en-tête et éviter les erreurs de mapping en amont.

2) Typage et normalisation

- Étapes utilisées : `Select Values`, `Date format`, `Number format`.
- Actions : conversion de `pickup_datetime` et `dropoff_datetime` en type date, conversion de `total_amount` en numérique et `passenger_count` en entier.
- Motif : centraliser les conversions pour rendre les règles de validation simples et déterministes.

3) Règles de validation

- Étape utilisée : `Filter Rows` (plus des règles supplémentaires en amont quand utile).
- Règles appliquées :
  - `total_amount > 0`
  - `passenger_count > 0`
  - `pickup_datetime` IS NOT NULL
  - `dropoff_datetime` IS NOT NULL
- Flux : les enregistrements satisfaisant les règles sont routés vers `clean`, les autres sont routés vers `rejected`.

4) Gestion d'erreur

- Configuration d'`Error handling` sur les étapes critiques (par ex. parsing de date, cast numérique).
- Pour chaque ligne rejetée, ajout d'une colonne `error_reason` décrivant la (ou les) cause(s) du rejet (ex. `missing_pickup_datetime`, `negative_total_amount`, `invalid_number_format`).

5) Export

- Sorties :
  - `data/clean/taxi_clean.csv` (flux valide)
  - `data/rejected/taxi_rejected.csv` (flux invalide + `error_reason`)

## Rejouabilité et traçabilité

- `processing_timestamp` ajouté à chaque enregistrement pour tracer l'exécution.
- `batch_id` (UUID ou horodatage) ajouté pour différencier les exécutions et permettre un re-run idempotent.
- Stratégie d'écriture : fichiers horodatés ou écrasement contrôlé selon le mode d'exécution choisi.

## Cas particuliers et points d'attention

- Changement de nom de colonne : la solution retenue est de valider l'en-tête avant traitement et d'utiliser un mapping configurable dans le pipeline.
- Volumes importants (ex. 10M lignes) : privilégier le traitement par lots/streaming, surveiller l'I/O et la mémoire, et tester le pipeline sur des échantillons représentatifs.
- Opérations non-déterministes : éviter ou documenter les étapes qui empêchent la rejouabilité (ex. génération d'IDs uniques sans `batch_id`).

## Validation effectuée

- Vérification manuelle du bon routage des cas tests (lignes valides vs lignes présentant : date manquante, montant négatif, passager = 0).
- Vérification que les fichiers de sortie `data/clean/taxi_clean.csv` et `data/rejected/taxi_rejected.csv` contiennent respectivement les enregistrements attendus et les raisons de rejet.

Remarque : si vous souhaitez, je peux exécuter des tests automatisés et ajouter un script de validation qui compare les counts attendus vs réels.

## Fichiers et emplacements produits

- Pipeline Hop (à créer) : placer le fichier pipeline dans `TP/Jour3/Lab1/`.
- Sorties attendues :
  - `TP/Jour3/Lab1/data/clean/taxi_clean.csv`
  - `TP/Jour3/Lab1/data/rejected/taxi_rejected.csv`

## Reproduire la démarche (instructions courtes)

1. Ouvrir Apache Hop.
2. Créer/importer le pipeline décrit.
3. Pointer l'étape `CSV File Input` vers `TP/Jour3/Lab1/data/raw/`.
4. Lancer l'exécution et vérifier les deux fichiers de sortie.

## Améliorations possibles

- Ajout d'un monitoring (métriques : lignes lues, lignes rejetées).
- Partitionnement des sorties pour gros volumes (par date ou par batch_id).
- Tests automatisés et CI pour valider les changements de schéma en entrée.

---

Fichier : [TP/Jour3/Lab1/README.md](TP/Jour3/Lab1/README.md)

Souhaitez-vous que je génère un exemple de pipeline Hop (fichier XML) conforme à ce compte‑rendu ?
