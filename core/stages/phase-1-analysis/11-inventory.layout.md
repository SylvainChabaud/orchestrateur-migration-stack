# 🧩 Stage 11 – inventory.layout
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 10 – inventory.structure  
**Next :** 12 – inventory.styles

---

## 🎯 Objectif

Construire l’**inventaire de layout** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- le bridge Legacy → DSL,
- les guides internes de layout.

L’objectif est de produire un JSON `inventory.layout.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **régions de layout** (header, sidebar, main, footer, panels, drawers, overlays…),
- les **structures de grille / stack / liste / table**,
- les **relations layout → vues** (association aux `ucr` de structure),
- les premières **notations de responsive** (breakpoints, comportements principaux).

Cet inventaire :

- se base sur les `ucr` fournis par `inventory.structure.json`,
- utilise les concepts DSL de layout (`layout.*`),
- ne s’occupe pas de la logique métier ni des flux de données,
- prépare la Phase 2 (mappings.layout) et la génération des layouts React 19.

---

## ⚙️ Inputs

> Tous les chemins ci-dessous sont dérivés exclusivement de `project.config.yaml`  
> via `project.*` et `paths.*`. Aucun chemin absolu n’est autorisé.

### 1. Configuration projet (déjà chargée par le runtime)

Clés lues par ce stage :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `runtime.regenerateStackGuides`
- `stack.custom`
- `gates.*`
- `stages.*`

> Ces valeurs sont considérées comme déjà en mémoire. Le stage ne fait que les consommer.

---

### 2. Code Legacy (lecture seule)

- `${paths.legacySource}`  
  - Fichier d’entrée principal de la page Legacy (`CampaignsDetail`, etc.).
  - Doit être lu **dans son emplacement d’origine** (répertoire applicatif principal).
  - ❌ Ne jamais copier ce fichier dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Layout**
  - `${paths.core}/guides-internals/inventory/guide.inventory.layout.md`
  - Fournit :
    - l’**objectif** du domaine Layout,
    - le **schéma JSON contractuel** de `inventory.layout.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec `inventory.structure`, références `ucr` valides),
    - les relations avec les autres inventaires (structure, styles, actions…).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans cette étape :
    - confirmer la façon de référencer les vues via leurs `ucr`,
    - garantir que l’inventaire Layout **réutilise** les `ucr` existants (structure) au lieu d’en inventer de nouveaux,
    - s’assurer qu’aucune ambiguïté n’est introduite dans les références.

---

### 4. Bridge Legacy → DSL

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- fournir les patterns Legacy associés aux concepts DSL de layout :
  - `layout.region`
  - `layout.section`
  - `layout.grid`
  - `layout.stack`
  - `layout.card`
  - `layout.list`
  - `layout.table`
  - `layout.modal`
  - `layout.drawer`
  - `layout.panel`
  - `layout.overlay`
  - `layout.responsiveBreakpoint`
- éviter de coder en dur des heuristiques spécifiques à un framework,
- assurer une détection cohérente et stable des patterns de layout dans le Legacy.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - aligner la description du layout sur l’arborescence cible,
    - vérifier que les grandes régions (header, main, sidebar, footer…) existent ou sont projetables.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - comprendre les conventions d’architecture cible (layout global de la page, découpage en régions),
    - donner du contexte à l’IA pour catégoriser correctement les régions de layout.

---

### 6. Output précédent requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir la liste des vues (`items[]`) et leurs `ucr`,
    - permettre au Layout d’associer chaque région/structure de layout à un ou plusieurs `ucr`,
    - servir de base pour naviguer dans la hiérarchie parent → enfants.

Sans cet inventaire, le stage **ne peut pas** produire un layout cohérent.  
En cas d’absence ou d’invalidité, le stage doit conclure sur un `Gate ❌`.

---

## 📤 Outputs

Tous les fichiers sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.layout.md`,
- `domain` doit valoir `"layout"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références à la structure (`structureUcr`, `regionRootUcr`, etc.) doivent correspondre à des `ucr` présents dans `inventory.structure.json`,
- JSON strictement sérialisable et sans clés non documentées.

---

## 🧠 Actions

1. **Charger le contexte global**
   - Récupérer depuis la configuration :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`,
     - `paths.stages`,
     - `gates.*`.

2. **Charger les artefacts nécessaires**
   - Lire :
     - `${paths.workspace}/projects/${project.name}/stack/project-structure.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`.
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.layout.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer la vue d’ensemble**
   - Construire en mémoire :
     - un index `ucr → StructureNode` à partir de `inventory.structure.json`,
     - une vision hiérarchique de la page (root → enfants),
     - un index `dslId → legacyPatterns[]` pour les concepts `layout.*` depuis le bridge.

4. **Analyser le code Legacy pour le layout**
   - Lire `${paths.legacySource}`.
   - En utilisant :
     - les patterns du bridge (`layout.*`),
     - la hiérarchie de structure,
   - identifier :
     - les **régions de layout** (header, main, sidebar, footer, panels, modals, drawers, overlays…),
     - les structures de **grilles**, **stacks**, **listes**, **tables**,
     - les éléments ayant un comportement **responsive** explicite (breakpoints, conditions CSS/JS).

5. **Construire les items de layout**
   - Pour chaque région / structure identifiée, construire un `LayoutItem` qui :
     - référence un ou plusieurs `ucr` de structure (ex : `regionRootUcr`, `structureUcrs[]`),
     - définit les propriétés principales du layout :
       - type (`region`, `grid`, `stack`, `list`, `table`, `modal`, `drawer`, `overlay`, …),
       - nom logique (`name`),
       - position ou rôle (`"header"`, `"main"`, `"sidebar"`, `"footer"`, etc.),
       - configuration responsive (breakpoints, comportements),
       - métadonnées utiles (ordre, importance, notes).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité avec le schéma du guide.

7. **Validation interne**
   - Vérifier que :
     - toutes les références aux `ucr` de structure sont valides,
     - les champs obligatoires sont présents pour chaque item,
     - le JSON est cohérent (aucune région dupliquée de manière incohérente, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]` avec toutes les anomalies détectées.

8. **Écriture de l’output**
   - Écrire `inventory.layout.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "11",
  "stageName": "inventory.layout",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "bridgeLoaded": true,
    "structureInventoryLoaded": true,
    "legacyParsed": true,
    "itemsBuilt": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

---

## 🧩 Gate

Utiliser exactement l’un des blocs suivants :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

- `Gate ✅` si `inventory.layout.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, bridge manquant, schéma violé).

---

## 📦 Next

> Continuer avec `12-inventory.styles.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
