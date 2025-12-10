# 🧩 Stage 74 – integration-consistency
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 73 – dependencies-coherence  
**Next:** 75 – accessibility-heuristic

---

## 🎯 Objective

Vérifier que le code généré pour `${project.pageName}` s’intègre correctement dans la **structure cible** du projet :  
routing, fichiers d’entrée, layouts et conventions de la stack.

Ce stage ne lance pas l’application. Il s’appuie uniquement sur la structure des fichiers, les stack-guides et la description de la structure cible (`project-structure.json` ou équivalent).

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `paths.core`, `gates`, `stack.id`

- **Structure de projet**
  - `${paths.workspace}/projects/${project.name}/project-structure.json`  
    (ou fichier équivalent décrit dans les stack-guides)

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`
    > Adapter si la structure réelle diffère, mais sans coder en dur de chemins absolus.

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.integration-consistency.md` (guide principal pour ce domaine)

- **Stack-guides (routing & structure)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.validation.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.performance.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tests.md`
  - (et tout guide décrivant routing/layout si disponible)

---

## 📤 Outputs

Dans :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/integration-consistency/
```

Fichiers :

- `integration-consistency.report.json`
- `integration-consistency.report.md`
- `.meta/validation.integration-consistency.meta.json`

---

## 🧠 Actions

1. **Charger la configuration et la structure cible**
   - Lire `project-config.yaml` et `project-structure.json`.
   - Identifier les emplacements attendus pour :
     - la page,
     - le layout associé,
     - les éventuels wrappers/providers.

2. **Scanner le code généré**
   - Parcourir `src_new` pour lister :
     - fichiers de pages,
     - fichiers de layout,
     - fichiers de routing ou d’enregistrement de routes (selon la stack).

3. **Comparer avec la structure cible**
   - Vérifier la présence de tous les fichiers d’entrée nécessaires à `${project.pageName}`.
   - Vérifier que les fichiers générés se trouvent aux bons emplacements.

4. **Analyser le routing**
   - Extraire les routes déclarées par la page et les comparer à la configuration cible.
   - Identifier les collisions de routes (routes doublonnées sans justification).

5. **Analyser les layouts et wrappers**
   - Vérifier que la page utilise le layout/wrapper attendu (si prescrit).
   - Identifier tout conflit manifeste (mauvais layout pour une zone donnée).

6. **Construire les métriques et le score**
   - `routesDeclared`, `routeConflicts`, `layoutConflicts`, `missingEntryFiles`.
   - Calculer un score (0–100) en partant de 100 et en appliquant des pénalités.

7. **Déterminer le Gate**
   - Gate ❌ si :
     - `routeConflicts > 0`, ou
     - `missingEntryFiles > 0`, ou
     - score < `minScore` (défini dans les stack-guides).

8. **Générer les rapports**
   - JSON détaillé (`integration-consistency.report.json`).
   - Markdown synthétique (`integration-consistency.report.md`).
   - `.meta/validation.integration-consistency.meta.json` :
     - inputs, outputs, score, gate, timestamp, etc.

---

## ✅ Auto-Checks

```json
{
  "stageId": "74",
  "stageName": "integration-consistency",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "projectStructureLoaded": true,
    "generatedCodeScanned": true,
    "metricsComputed": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si conflit de route, fichier d’entrée manquant ou score insuffisant.)*

---

## 📦 Next

> Continue with `75-accessibility-heuristic.md` if Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
