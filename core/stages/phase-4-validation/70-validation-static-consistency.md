# 🧩 Stage 70 – static-consistency
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 58 – generation-summary  
**Next:** 71 – tests-audit

---

## 🎯 Objective

Valider la **cohérence statique** du code généré pour la page `${project.pageName}` :  
imports, structure des fichiers, types/props basiques et principaux code smells.

Ce stage travaille exclusivement sur le **code généré Phase 3** (`src_new`), en s’appuyant sur les **stack-guides** et, si possible, sur les inventaires/mappings pour rattacher les problèmes aux UCR.

---

## ⚙️ Inputs

> **Tous les inputs doivent être dérivés de `core/configs/project-config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.**  

- **Configuration**
  - `core/configs/project-config.yaml`
    - clés utilisées : `project.pageName`, `project.name`, `paths.workspace`, `paths.legacySource`, `paths.core`, `stack.id`, `gates`

- **Code généré (Phase 3, read-only)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Stack guides (Phase 0, read-only)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.validation.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.quality-thresholds.md`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.static-consistency.md` (guide principal pour ce domaine)
  - `${paths.core}/guides-internals/globals/guide.ucr.md` (pour rattacher les issues aux UCR si possible)

- **Inventaires / mappings (optionnel mais recommandé)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/*.json`
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/*.json`

---

## 📤 Outputs

Tous les outputs doivent être générés sous :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/static-consistency/
```

- `static-consistency.report.json`
- `static-consistency.report.md`
- `.meta/validation.static-consistency.meta.json`

Règles :
- Ne rien écrire dans `core/`.
- Ne pas modifier les artefacts des phases précédentes.
- Les chemins doivent respecter l’arborescence ci-dessus.

---

## 🧠 Actions

1. **Charger la configuration et le contexte**
   - Lire `core/configs/project-config.yaml`.
   - Résoudre : `project.pageName`, `project.name`, `paths.workspace`, `paths.legacySource`.
   - Vérifier que `src_new` existe et contient au moins un fichier.

2. **Charger les guides et seuils**
   - Lire `guide.validation.md` pour récupérer les règles génériques de validation.
   - Lire `guide.quality-thresholds.md` pour déterminer :
     - `minConsistencyScore`,
     - règles sur `unresolvedImports` et `circularDependencies`.

3. **Scanner le code généré**
   - Parcourir récursivement `src_new` :
     - lister les fichiers analysables (TS/TSX/JS/JSX/etc. selon la stack),
     - collecter tous les imports (locaux et externes).

4. **Analyser les imports et la structure**
   - Pour chaque import local :
     - vérifier qu’un fichier cible existe dans `src_new` (ou dans la structure prescrite).
   - Pour chaque import externe :
     - vérifier sa plausibilité par rapport à la stack (sans exécuter de resolver).
   - Identifier les relations entre modules pour détecter des **cycles simples** :
     - A importe B et B importe A → cycle.

5. **Analyser types/props (heuristique)**
   - Identifier les principaux composants/pages/hooks.
   - Vérifier, selon les règles de stack :
     - présence de types/PropTypes si la stack le recommande,
     - absence d’incohérences évidentes entre usage et définition de props.

6. **Détecter les code smells principaux**
   - Fonctions ou composants au-delà d’un seuil de taille/complexité défini dans les guides.
   - Imbrication trop profonde de blocs conditionnels / boucles.
   - Duplications évidentes de blocs de code.

7. **Construire les métriques et le score**
   - Calculer :  
     - `totalFiles`, `totalImports`,  
     - `unresolvedImports`, `circularDependencies`,  
     - `typeInconsistencies`, `detectedSmells.*`,  
     - `consistencyScore` (score heuristique 0–100).
   - Déterminer le `validation.score` global (0–100).

8. **Déterminer le statut et le Gate**
   - Définir `validation.status` : `valid`, `warnings` ou `rejected` selon :
     - sévérité et nombre d’issues,
     - comparaison aux seuils.
   - Définir `validation.gate` :
     - Gate ✅ si :
       - `unresolvedImports == 0` sur les fichiers critiques,
       - `circularDependencies == 0`,
       - `consistencyScore >= minConsistencyScore`.
     - Gate ❌ sinon.

9. **Générer les outputs**
   - Écrire `static-consistency.report.json` selon le contrat du guide de validation.
   - Générer `static-consistency.report.md` :
     - résumé, métriques, issues, recommandations, conclusion.
   - Écrire `.meta/validation.static-consistency.meta.json` :
     - inputs, outputs, score, Gate, timestamp, nombre de checks.

---

## ✅ Auto-Checks

L’IA doit produire en sortie (dans la réponse, pas sur disque) un court résumé JSON :

```json
{
  "stageId": "70",
  "stageName": "static-consistency",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "srcNewExists": true,
    "stackGuidesLoaded": true,
    "metricsComputed": true,
    "reportsWritten": true
  }
}
```

Si un champ est `false`, expliquer clairement pourquoi dans le rapport Markdown.

---

## 🧩 Gate

Gate ✅

*(Mettre Gate ❌ si les conditions de succès ne sont pas réunies — imports non résolus critiques, cycles, score trop bas, etc.)*

---

## 📦 Next

> Continue with `71-tests-audit.md` if `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
