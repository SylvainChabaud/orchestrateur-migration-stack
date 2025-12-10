# 🧩 Stage 00 – Stack Guides Builder
**Phase:** Phase 0 – Bootstrap  
**Prev:** (none)  
**Next:** 01 – Project Structure Spec Builder

---

## 🎯 Objective

Generate all **stack-specific guides** required by the orchestrator for the target stack defined in:

- `stack.custom` → `${paths.core}/configs/stacks/my-custom-enterprise.stack.yaml`

For the current project, this stage must:

- Read the user-defined stack configuration.
- Use **MCP context 7** to fetch up-to-date documentation and best practices for each tool/library in the stack.
- Produce a coherent set of **stack guides** under `workspace/` that will be reused by later stages (inventories, mappings, generation, validation).
- Build a **summary index** (`stack-guides-summary.json`) so that following stages can easily discover and select only the relevant guides.

---

## ⚙️ Inputs

> All inputs MUST be resolved from `core/configs/project.config.yaml`.  
> Never hard-code absolute paths.

- **Configuration**
  - `core/configs/project.config.yaml`  
    - keys used:
      - `project.name`
      - `paths.core`
      - `paths.workspace`
      - `stack.custom`
      - optionally `ai`, `gates`

- **Stack definition (user config)**
  - `${stack.custom}`  
    e.g. `${paths.core}/configs/stacks/my-custom-enterprise.stack.yaml`  
    This file lists the main tools/libraries of the final stack (React, router, i18n, design system, store, API client, tests, etc.).


- **MCP / External knowledge**
  - **MCP context 7**  
    - Use it to fetch official docs & best practices for tools defined in the stack file.

- **Legacy code**
  - ❌ Not used in this stage.

---

## 📤 Outputs

All outputs must be written under:

```text
${paths.workspace}/projects/${project.name}/stack/stack-guides/
```

### Guides (markdown)

- `guide.stack.md` (overview général de la stack)
- `guide.structure.md` (architecture des dossiers et modules)
- `guide.types.md` (types et interfaces, DTO)
- `guide.naming.md` (conventions de nommage)
- `guide.conventions.md` (conventions générales de code)
- `guide.ui-components.md` (composants UI généraux)
- `guide.ui.atoms.md` (composants atoms spécifiques)
- `guide.ui.props.md` (conventions de props pour UI)
- `guide.ui-pages.md` (structure et patterns de pages)
- `guide.layout.md` (layouts et templates de page)
- `guide.styles.md` (styles et theming)
- `guide.routing.md` (configuration de routing)
- `guide.i18n.md` (internationalisation)
- `guide.state-management.md` (gestion d'état global : concepts)
- `guide.stores.md` (stores concrets : Zustand, Redux, etc.)
- `guide.state.md` (structure d'état, mutations)
- `guide.api-client.md` (clients HTTP, requêtes)
- `guide.forms.md` (formulaires et validation)
- `guide.validation.md` (schémas de validation : Zod, Yup, etc.)
- `guide.tests.md` (stratégie de tests)
- `guide.build-and-tooling.md` (build, bundling, tooling)
- `guide.monorepo.md` (monorepo si applicable)
- `guide.auth.md` (authentification et autorisation)
- `guide.performance.md` (performance et optimisations)
- `guide.accessibility.md` (accessibilité et standards WCAG)
- `guide.quality-thresholds.md` (seuils de qualité pour validation Phase 4)

> If some areas are not used (e.g. no monorepo), the corresponding guide must still exist but clearly document the choice (e.g. "Monorepo not used in this project. Rationale: …").

> **Note importante**: Tous ces guides doivent être créés (même vides avec une note explicative si non applicable) car ils sont référencés par les stages de Phase 1, 2 et 3.

### Summary index (JSON)

- `${paths.workspace}/projects/${project.name}/stack/stack-guides/stack-guides-summary.json`

Example structure:

```json
[
  {
    "id": "stack",
    "file": "guide.stack.md",
    "domain": "global",
    "tools": [],
    "topics": ["overview", "philosophy", "entrypoints"]
  },
  {
    "id": "types",
    "file": "guide.types.md",
    "domain": "types",
    "tools": ["typescript"],
    "topics": ["interfaces", "dto", "type-guards", "generics"]
  },
  {
    "id": "naming",
    "file": "guide.naming.md",
    "domain": "conventions",
    "tools": [],
    "topics": ["files", "functions", "components", "variables", "constants"]
  },
  {
    "id": "conventions",
    "file": "guide.conventions.md",
    "domain": "conventions",
    "tools": [],
    "topics": ["code-style", "imports", "exports", "comments"]
  },
  {
    "id": "stores",
    "file": "guide.stores.md",
    "domain": "state",
    "tools": ["zustand"],
    "topics": ["store-creation", "selectors", "actions", "persistence"]
  },
  {
    "id": "state",
    "file": "guide.state.md",
    "domain": "state",
    "tools": [],
    "topics": ["state-shape", "immutability", "normalization"]
  },
  {
    "id": "ui-atoms",
    "file": "guide.ui.atoms.md",
    "domain": "ui",
    "tools": [],
    "topics": ["button", "input", "icon", "props-pattern"]
  },
  {
    "id": "ui-props",
    "file": "guide.ui.props.md",
    "domain": "ui",
    "tools": [],
    "topics": ["props-typing", "default-values", "validation"]
  },
  {
    "id": "ui-pages",
    "file": "guide.ui-pages.md",
    "domain": "ui",
    "tools": [],
    "topics": ["page-structure", "composition", "data-loading"]
  },
  {
    "id": "layout",
    "file": "guide.layout.md",
    "domain": "ui",
    "tools": [],
    "topics": ["layout-templates", "zones", "responsive"]
  },
  {
    "id": "routing",
    "file": "guide.routing.md",
    "domain": "routing",
    "tools": ["react-router"],
    "topics": ["route-definition", "lazy-loading", "guards"]
  },
  {
    "id": "forms",
    "file": "guide.forms.md",
    "domain": "forms",
    "tools": ["react-hook-form", "zod"],
    "topics": ["validation", "errors", "submit"]
  }
]
```

Rules:

- `id` = stable identifier used by later stages (e.g. `routing`, `forms`, `api-client`).
- `file` = relative path within `stack-guides/`.
- `domain` = high-level area (structure, routing, forms, api, tests, …).
- `tools` = list of tools from stack config used in this guide.
- `topics` = list of key topics covered (helps filtering by need).

---

## 🧠 Actions

1. **Load configuration**
   - Read `core/configs/project.config.yaml`.
   - Resolve:
     - `project.name`
     - `paths.core`
     - `paths.workspace`
     - `stack.custom`
   - Build the base output path:
     - `${paths.workspace}/projects/${project.name}/stack/stack-guides/`.

2. **Read stack definition**
   - Open `${stack.custom}` (YAML).
   - Parse **deux sections principales** :
     - **`metadata`** : Informations structurelles et conventionnelles du projet
       - `architecture` (type, folderStructure, packageManagement)
       - `naming` (files, functions, constants)
       - `projectStructure` (srcLayout)
       - `performance` (heavyLibraries, optimization, targets)
       - `accessibility` (standard, requirements, tools, targets)
       - `qualityThresholds` (validation, globalScore)
       - `layouts` (available, responsive)
     - **`tools`** : Technologies et versions de la stack
       - runtime, frontend, routing, i18n, design, stateManagement
       - api, validation, forms, build, tests, auth
       - devTools, packageManagers, monorepo, automation, documentation
   - Pour chaque outil dans `tools`, extraire :
     - `name`, `library`, ou identifiant principal
     - `version`
     - `patterns` (bonnes pratiques recommandées)
     - `note` / `comment` si présent

3. **Fetch documentation via MCP context 7**
   
   Pour chaque outil identifié dans `tools.*` :
   
   **Étape 3.1 : Identifier l'outil et sa version**
   - Extraire : `library` (ou `name`), `version`, `patterns` si présent
   - Exemples :
     - `tools.frontend.library` = "React", `libraryVersion` = "19.2.0"
     - `tools.routing.router` = "React Router", `routerVersion` = "7.9.6"
     - `tools.design.designSystem` = "@peaksys/design-system", `designSystemVersion` = "8.42.0"
   
   **Étape 3.2 : Interroger MCP Context7**
   - Utiliser l'outil `mcp_context7_resolve-library-id` pour obtenir l'ID Context7
   - Puis `mcp_context7_get-library-docs` pour récupérer la documentation
   - **Focus sur :**
     - Best practices pour projets mid/large
     - Patterns d'architecture recommandés
     - Anti-patterns courants à éviter
     - Migration notes (si applicable)
     - Performance considerations
     - Accessibility guidelines (si applicable)
   
   **Étape 3.3 : Filtrage et synthèse**
   - Ne garder que les informations utiles pour :
     - **Migration** : patterns à reproduire du legacy vers la stack cible
     - **Code generation** : templates et structures à générer
     - **Consistency** : conventions à respecter dans tout le code
   - Ignorer :
     - Tutoriels débutants
     - Informations obsolètes (versions antérieures)
     - Détails d'implémentation trop bas niveau
   
   **Étape 3.4 : Enrichissement des guides**
   - Intégrer les best practices de MCP Context7 dans les guides
   - Ajouter les références officielles dans la section "References"
   - Documenter les patterns dans la section "Principles & Conventions"
   - Créer des exemples dans la section "Examples" basés sur la doc officielle
   
   **Exemple de requête MCP Context7 :**
   ```
   Tool: mcp_context7_get-library-docs
   Input:
     context7CompatibleLibraryID: "/facebook/react/v19.2.0"
     topic: "hooks, performance, best practices"
     tokens: 5000
   ```

4. **Generate stack guides with explicit mapping**
   
   Utiliser le mapping suivant **Section YAML → Guide(s)** :
   
   **Depuis `metadata` :**
   - `metadata.architecture` → `guide.stack.md` + `guide.monorepo.md`
   - `metadata.naming` → `guide.naming.md`
   - `metadata.projectStructure` → `guide.structure.md`
   - `metadata.performance` → `guide.performance.md`
   - `metadata.accessibility` → `guide.accessibility.md`
   - `metadata.qualityThresholds` → `guide.quality-thresholds.md`
   - `metadata.layouts` → `guide.layout.md`
   
   **Depuis `tools` :**
   - `tools.runtime` → `guide.types.md` + `guide.conventions.md`
   - `tools.frontend` → `guide.ui-components.md` + `guide.ui-pages.md`
   - `tools.routing` → `guide.routing.md`
   - `tools.i18n` → `guide.i18n.md`
   - `tools.design` → `guide.ui.atoms.md` + `guide.ui.props.md` + `guide.styles.md`
   - `tools.stateManagement.globalState` → `guide.stores.md`
   - `tools.stateManagement.serverState` → `guide.state-management.md` + `guide.state.md`
   - `tools.api` → `guide.api-client.md`
   - `tools.validation` → `guide.validation.md`
   - `tools.forms` → `guide.forms.md`
   - `tools.build` → `guide.build-and-tooling.md`
   - `tools.tests` → `guide.tests.md`
   - `tools.auth` → `guide.auth.md`
   
   **Pour chaque guide généré :**
   - Créer un fichier `.md` avec les sections suivantes :
     
     ```markdown
     # 📘 Guide [Nom] — [Stack ID]
     
     ## 🎯 Context & Scope
     - Objectif du guide
     - Phases qui l'utilisent (Phase 1, 2, 3, 4)
     - Concepts couverts
     
     ## 📦 Source Configuration
     - Sections YAML utilisées (metadata.X ou tools.Y)
     - Outils/versions concernés
     
     ## 🏗️ Principles & Conventions
     - Règles principales
     - Patterns recommandés (depuis tools.*.patterns si disponible)
     - Anti-patterns à éviter
     
     ## 📝 Examples
     - Snippets de code concrets
     - Cas d'usage typiques
     
     ## ✅ Do / ❌ Don't
     - Bonnes pratiques
     - Erreurs courantes à éviter
     
     ## 🔗 References
     - Liens vers outils dans my-custom-enterprise.stack.yaml
     - Documentation officielle (enrichie par MCP Context7)
     
     ## 📊 Usage in Orchestrator
     - Phase 1 : Inventaires qui utilisent ce guide
     - Phase 2 : Mappings qui utilisent ce guide
     - Phase 3 : Génération qui utilise ce guide
     - Phase 4 : Validations qui utilisent ce guide
     ```
   
   - **Garantir la cohérence :**
     - Les guides ne doivent pas se contredire
     - Scopes clairement séparés (routing ≠ forms ≠ api)
     - Terminologie unifiée entre tous les guides
   
   - **Si une section `metadata` ou `tools` est absente :**
     - Créer le guide avec une note explicative :
       ```markdown
       ## ⚠️ Non applicable
       Ce domaine n'est pas utilisé dans ce projet.
       **Rationale :** [Expliquer pourquoi]
       ```

5. **Build the summary index (`stack-guides-summary.json`)**
   
   Pour chaque guide généré (26 guides), créer une entrée structurée :
   
   ```json
   {
     "id": "routing",
     "file": "guide.routing.md",
     "domain": "routing",
     "sourceYaml": ["tools.routing"],
     "tools": ["React Router 7.9.6"],
     "topics": ["route-definition", "lazy-loading", "guards", "nested-routes"],
     "usedByPhases": [1, 2, 3],
     "usedByStages": [22, 42, 58]
   }
   ```
   
   **Mapping complet des 26 guides :**
   
   | id | file | domain | sourceYaml | usedByPhases |
   |----|------|--------|------------|--------------|
   | stack | guide.stack.md | global | metadata.architecture | 1,2,3 |
   | structure | guide.structure.md | structure | metadata.projectStructure | 1,2,3 |
   | types | guide.types.md | types | tools.runtime | 1,2,3 |
   | naming | guide.naming.md | conventions | metadata.naming | 1,2,3 |
   | conventions | guide.conventions.md | conventions | tools.runtime | 1,2,3 |
   | ui-components | guide.ui-components.md | ui | tools.frontend, tools.design | 1,2,3 |
   | ui-atoms | guide.ui.atoms.md | ui | tools.design | 1,2,3 |
   | ui-props | guide.ui.props.md | ui | tools.design | 1,2,3 |
   | ui-pages | guide.ui-pages.md | ui | tools.frontend | 1,2,3 |
   | layout | guide.layout.md | ui | metadata.layouts | 1,2,3 |
   | styles | guide.styles.md | ui | tools.design | 1,2,3 |
   | routing | guide.routing.md | routing | tools.routing | 1,2,3 |
   | i18n | guide.i18n.md | i18n | tools.i18n | 1,2,3 |
   | state-management | guide.state-management.md | state | tools.stateManagement | 1,2,3 |
   | stores | guide.stores.md | state | tools.stateManagement.globalState | 1,2,3 |
   | state | guide.state.md | state | tools.stateManagement | 1,2,3 |
   | api-client | guide.api-client.md | api | tools.api | 1,2,3 |
   | forms | guide.forms.md | forms | tools.forms | 1,2,3 |
   | validation | guide.validation.md | validation | tools.validation | 1,2,3,4 |
   | tests | guide.tests.md | tests | tools.tests | 1,2,3,4 |
   | build-and-tooling | guide.build-and-tooling.md | build | tools.build | 3 |
   | monorepo | guide.monorepo.md | infrastructure | metadata.architecture, tools.monorepo | 1,2,3 |
   | auth | guide.auth.md | auth | tools.auth | 1,2,3 |
   | performance | guide.performance.md | quality | metadata.performance | 3,4 |
   | accessibility | guide.accessibility.md | quality | metadata.accessibility | 3,4 |
   | quality-thresholds | guide.quality-thresholds.md | quality | metadata.qualityThresholds | 4 |
   
   **Écrire le JSON complet :**
   - Créer un tableau de 26 entrées
   - Vérifier qu'il n'y a pas d'`id` dupliqué
   - Sauvegarder dans : `${paths.workspace}/projects/${project.name}/stack/stack-guides/stack-guides-summary.json`
   
   **Structure finale du JSON :**
   ```json
   [
     {
       "id": "stack",
       "file": "guide.stack.md",
       "domain": "global",
       "sourceYaml": ["metadata.architecture"],
       "tools": ["Nx 22.1.1", "Yarn 1.22.22"],
       "topics": ["overview", "philosophy", "monorepo", "architecture"],
       "usedByPhases": [1, 2, 3],
       "usedByStages": [10, 30, 50]
     },
     ...
   ]
   ```

6. **Consistency checks**
   
   **Check 6.1 : Complétude des guides (26 obligatoires)**
   - Vérifier que les 26 fichiers `.md` existent dans `stack-guides/`
   - Vérifier qu'aucun fichier n'est vide (minimum 100 caractères)
   - Si un guide manque → **Gate ❌**
   
   **Check 6.2 : Validité du summary JSON**
   - Vérifier que `stack-guides-summary.json` existe
   - Vérifier qu'il contient exactement 26 entrées
   - Vérifier que chaque entrée a tous les champs obligatoires :
     - `id`, `file`, `domain`, `sourceYaml`, `tools`, `topics`, `usedByPhases`
   - Vérifier qu'il n'y a pas d'`id` dupliqué
   - Si invalide → **Gate ❌**
   
   **Check 6.3 : Mapping YAML → Guides**
   - Pour chaque section `metadata.*` obligatoire, vérifier qu'au moins un guide l'utilise
   - Pour chaque section `tools.*` obligatoire, vérifier qu'au moins un guide l'utilise
   - Si une section obligatoire n'est référencée par aucun guide → **Warning**
   
   **Check 6.4 : Cohérence des guides**
   - Vérifier que tous les guides référencent des outils présents dans le YAML
   - Vérifier qu'aucun outil "inconnu" n'est mentionné
   - Vérifier que les versions mentionnées correspondent au YAML
   - Si incohérence → **Warning** (mais pas Gate ❌)
   
   **Check 6.5 : Structure des guides**
   - Vérifier que chaque guide contient les sections minimales :
     - "Context & Scope"
     - "Principles & Conventions"
     - "References"
   - Si structure manquante → **Warning**
   
   **Check 6.6 : Coverage MCP Context7**
   - Vérifier que MCP Context7 a été appelé pour au moins 80% des outils
   - Si coverage < 80% → **Warning**
   
   **Générer un rapport de checks :**
   ```json
   {
     "checks": {
       "guidesCount": { "expected": 26, "actual": 26, "status": "✅" },
       "summaryValid": { "entries": 26, "duplicates": 0, "status": "✅" },
       "yamlMapping": { "metadataSections": 7, "toolsSections": 12, "status": "✅" },
       "coherence": { "unknownTools": 0, "versionMismatches": 0, "status": "✅" },
       "structure": { "guidesWithAllSections": 26, "status": "✅" },
       "mcpCoverage": { "toolsQueried": 12, "toolsTotal": 15, "percentage": 80, "status": "⚠️" }
     }
   }
   ```

---

## ✅ Auto-Checks

At the end of the stage, produce a short JSON summary (in the assistant’s answer), for example:

```json
{
  "stageId": "00",
  "stageName": "Stack Guides Builder",
  "projectName": "${project.pageName}",
  "stackFile": "${stack.custom}",
  "stackSections": {
    "metadata": {
      "read": true,
      "sections": ["architecture", "naming", "projectStructure", "performance", "accessibility", "qualityThresholds", "layouts"],
      "count": 7
    },
    "tools": {
      "read": true,
      "sections": ["runtime", "frontend", "routing", "i18n", "design", "stateManagement", "api", "validation", "forms", "build", "tests", "auth"],
      "count": 12
    }
  },
  "mcpContext7": {
    "used": true,
    "toolsQueried": 12,
    "toolsTotal": 15,
    "coverage": "80%"
  },
  "guides": {
    "generated": 26,
    "expected": 26,
    "status": "complete",
    "breakdown": {
      "fromMetadata": 7,
      "fromTools": 19
    }
  },
  "summary": {
    "generated": true,
    "entries": 26,
    "duplicateIds": 0,
    "valid": true
  },
  "checks": {
    "stackFileReadable": true,
    "metadataPresent": true,
    "toolsPresent": true,
    "allGuidesWritten": true,
    "allGuidesNonEmpty": true,
    "summaryValid": true,
    "noDuplicateIds": true,
    "yamlMappingComplete": true,
    "coherenceOk": true
  },
  "warnings": []
}
```

This JSON is **not written to disk**; it is only a synthetic summary for the user.

---

## 🧩 Gate

At the very end of the stage, decide if the stage is a success or a failure.

Write exactly one of:

```markdown
## 🧩 Gate
Gate ✅
```

or

```markdown
## 🧩 Gate
Gate ❌
```

Use **`Gate ✅`** only if:

- `${stack.custom}` has been read successfully,
- all expected guides under `stack-guides/` have been created and are non-empty,
- `stack-guides-summary.json` exists and is consistent (no duplicate ids, all guides listed).

Use **`Gate ❌`** if:

- the stack file is missing or unreadable,
- one or more required guides could not be generated,
- the summary index is missing or inconsistent.

If `Gate ❌`, briefly explain the blocking issues just before the Gate block.

---

## 📦 Next

> Continue with `01-project-structure-spec-builder.md` if `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
