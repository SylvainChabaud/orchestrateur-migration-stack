3️⃣ Nouvelle spec Phase 4 – version réaliste pour ton orchestrateur

Je te propose cette nouvelle spec compacte (celle-ci pourra servir de base pour régénérer chaque stage + un guide global).

🎯 Rôle de la Phase 4 (version réaliste)

Phase 4 = Audit de qualité heuristique + synthèse + archivage, fondé sur :

Le code généré (src_new)

Les inventaires & mappings (UCR)

Les stack-guides (validation, tests, a11y, perf, qualité)

Les GoodPractices du projet

Sans exécution réelle de build/tests, mais avec analyse statique approfondie et cohérente.

Stage 70 – Static Consistency & Code Smells

Objectif : Vérifier la cohérence interne du code généré et détecter les principaux risques techniques.

Fait :

Analyse des imports (fichiers / deps / cycles)

Vérif basique types/props selon stack-guides

Détection de code smells (fonctions monstres, duplication évidente, etc.)

Gate ✅ si :

Pas d’incohérence structurelle majeure (imports manquants/cycliques sur des éléments critiques)

Score d’analyse statique ≥ seuil défini dans guide.quality-thresholds.md.

Stage 71 – Tests Audit (structure & couverture théorique)

Objectif : Vérifier que la stratégie de test générée est crédible et alignée avec les use-cases/UCR.

Fait :

Vérifie la présence des fichiers de tests attendus par les mappings.

Cartographie UCR ↔ tests (test-results.json devient un rapport déclaratif, pas un log d’exécution).

Estime une couverture théorique : % de UCR / behaviours ayant au moins un test associé.

Gate ✅ si :

Tous les cas d’usage critiques ont au moins un test associé.

Couverture théorique ≥ seuil fixé dans les stack-guides.

Stage 72 – Functional Equivalence Matrix

Objectif : Comparer le périmètre fonctionnel legacy vs nouveau code de façon déclarative.

Fait :

Produit une matrice d’équivalence entre behaviours legacy (inventaires Phase 1) et implémentations générées (Phase 3). 

PHASE-4-SPECIFICATION

Classe chaque behaviour en : covered, partiallyCovered, notCovered, unknown.

Liste les regressions potentielles : behaviours notCovered ou partiallyCovered sur des UCR critiques.

Gate ✅ si :

Aucune UCR critique n’est notCovered.

Les trous sont répertoriés dans regressions-detected.json pour correction manuelle.

Stage 73 – Dependencies & Imports Coherence

Objectif : S’assurer que les dépendances du code généré sont cohérentes avec le projet cible.

Fait :

Liste des deps utilisées dans src_new (imports externes).

Cross-check avec package.json et/ou stack-guides.

Repérage des imports non résolus & deps “soupçonnées à vérifier”.

Gate ✅ si :

0 import non résolu critique.

Toutes les nouvelles deps sont soit connues, soit marquées comme “à valider” mais non bloquantes.

Stage 74 – Integration Consistency Check

Objectif : Vérifier que le code généré s’intègre proprement dans la structure cible, d’un point de vue déclaratif.

Fait :

Cross-check des chemins générés avec project-structure.json. 

README

Vérif cohérence du routing (routes générées compatibles avec la config prescrite par les stack-guides).

Détection de collisions de noms / chemins / routes.

Gate ✅ si :

Aucune collision ou incohérence d’intégration critique.

Stage 75 – Accessibility Heuristic Audit

Objectif : Vérifier statiquement que le code suit les principales règles d’accessibilité définies dans la stack.

Fait :

Analyse du markup / JSX : labels, ARIA, structure sémantique, focus management “théorique”.

Calcul d’un score heuristique basé sur les règles de guide.accessibility.md.

Gate ✅ si :

Score a11y ≥ seuil WCAG défini (interprété de manière heuristique).

0 violation critique (ex : composants interactifs sans label ni texte accessible).

Stage 76 – Performance Patterns Audit

Objectif : Détecter les risques de performance les plus évidents dans le code généré.

Fait :

Repère imports “lourds” non lazy-loadés alors que les stack-guides le recommandent.

Repère composants / hooks suspects (grosse complexité visible, absence de mémoïsation sur des listes massives, etc.).

Gate ✅ si :

Aucun “risque perf” critique détecté sur les chemins principaux.

Les optimisations recommandées sont listées mais non bloquantes si non critiques.

Stage 77 – Project Quality & Guidelines Compliance

Objectif : Vérifier la conformité aux standards du projet (stack-guides qualité). 

PHASE-4-SPECIFICATION

Fait :

Vérifie nommage, structure de dossiers, patterns archi, documentation inline, types/PropTypes.

Calcule un score de conformité heuristique.

Gate ✅ si :

Score ≥ seuil “acceptable” (par ex. 85%).

Aucune violation critique (pattern archi totalement cassé, etc.).

Stage 78 – Validation Summary & Archive

Objectif : Synthétiser les résultats de la Phase 4 et générer l’archive finale. 

PHASE-4-SPECIFICATION

Fait :

Agrège tous les scores (70–77) → Score global de qualité (pondération déjà définie dans ta spec).

Produit validation-summary.json + validation-summary.report.md + actions-required.md. 

PHASE-4-SPECIFICATION

Crée l’arbo archive/${timestamp}/ avec un manifest.json listant tous les artefacts et les UCR couverts.

Gate ✅ si :

Tous les stages critiques (70, 72, 73, 74) sont Gate ✅.

Score global ≥ seuil (ex. 85%).

Manifest d’archive cohérent.