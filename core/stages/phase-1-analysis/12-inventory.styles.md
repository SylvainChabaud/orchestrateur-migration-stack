# 🧩 Stage 12 – inventory.styles
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 11 – inventory.layout  
**Next :** 13 – inventory.i18n

---

## 🎯 Objectif

Construire l’**inventaire des styles** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- les inventaires précédents (structure, layout),
- les guides internes,
- et, si disponible, le bridge Legacy → DSL pour les aspects liés au style.

L’objectif est de produire un JSON `inventory.styles.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **sources de styles** (CSS, CSS-in-JS, thèmes, tokens),
- les **règles de styles** significatives associées aux vues (via leurs `ucr`),
- les **groupes de styles** (thèmes, variantes, modes light/dark…),
- les **points d’accroche** pour la future génération des styles dans la stack cible (React 19 + conventions entreprise).

Cet inventaire :

- **réutilise** les `ucr` de `inventory.structure.json` pour pointer vers les vues,
- s’appuie sur le layout si nécessaire pour contextualiser certaines zones,
- reste centré sur la **description fonctionnelle des styles** (pas sur la syntaxe exacte CSS/JS).

---

## ⚙️ Inputs

> Tous les chemins sont dérivés de `project.config.yaml` via `project.*` et `paths.*`.  
> Aucun chemin absolu ou valeur hors de ce contrat ne doit être utilisé.

### 1. Configuration projet (en mémoire)

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

---

### 2. Code Legacy (lecture seule)

- `${paths.legacySource}`  
  - Fichier d’entrée principal de la page Legacy.
  - Le stage peut suivre les imports/export liés aux styles (CSS, modules, hooks de thème, etc.) à partir de ce point d’entrée.
  - ❌ Ne jamais copier les fichiers Legacy dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Styles**
  - `${paths.core}/guides-internals/inventory/guide.inventory.styles.md`
  - Fournit :
    - l’**objectif** du domaine Styles,
    - le **schéma JSON contractuel** de `inventory.styles.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure & Layout),
    - les relations avec les autres inventaires (layout, i18n, actions…).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans cette étape :
    - garantir que les références vers les vues (`targetStructureUcr`, etc.) utilisent des UCR valides de `inventory.structure.json`,
    - si le domaine Styles introduit des UCR propres à certains items (ex : groupes de styles), s’assurer qu’ils respectent les règles UCR globales.

---

### 4. Bridge Legacy → DSL (optionnel pour ce domaine)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage (si des entrées pertinentes existent) :

- exploiter les mappings DSL qui touchent aux aspects visuels / de présentation si le bridge en dispose (ex. tags liés au thème, à la mise en forme),  
- assurer une cohérence de vocabulaire avec les autres domaines (ex. utilisation indirecte de `structure.*` ou `layout.*` pour contextualiser les styles).

> Si le bridge ne définit pas encore de concepts dédiés aux styles, le stage reste fonctionnel en décrivant les styles de manière générique, tout en se basant sur les UCR de Structure & Layout.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre la structure cible dans laquelle les styles devront être projetés,
    - repérer les zones susceptibles d’avoir des thèmes ou des variations fortes.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - aligner l’inventaire des styles sur les conventions de la stack cible (ex. usage de design tokens, de tailwind, d’un système de design interne),
    - orienter l’IA sur le **niveau de granularité** attendu (styles par composant, par région, par thème…).

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir la liste des vues (`items[]`) et leurs `ucr`,
    - permettre de lier chaque information de style à une ou plusieurs vues cibles.

- **Inventaire Layout (Stage 11) — fortement recommandé**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json`
  - Rôle :
    - contextualiser les styles par région (header, sidebar, main, footer, overlays…),
    - repérer les zones structurantes où s’appliquent des styles globaux (fond de page, barres latérales, conteneurs principaux).

Si `inventory.structure.json` est absent ou invalide, le stage doit conclure sur un `Gate ❌`.  
Si `inventory.layout.json` est absent, le stage peut continuer mais doit le signaler dans `validation.issues`.

---

## 📤 Outputs

Tous les fichiers sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.styles.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.styles.md`,
- `domain` doit valoir `"styles"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références vers `inventory.structure.json` (`targetStructureUcr`, etc.) doivent être valides,
- JSON strictement sérialisable, sans clés non documentées.

---

## 🧠 Actions

1. **Charger le contexte global**
   - Utiliser les valeurs de configuration déjà en mémoire :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`,
     - `paths.stages`,
     - `gates.*`.

2. **Charger les artefacts nécessaires**
   - Lire :
     - `${paths.workspace}/projects/${project.name}/stack/project-structure.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json` (optionnel pour ce domaine),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.styles.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`.
   - À partir de `inventory.layout.json` (si présent) :
     - construire un index `layoutUcr → LayoutItem` pour contextualiser certaines informations de style (par région).
   - À partir du bridge (si pertinent) :
     - récupérer les patterns Legacy liés à la présentation si le DSL les définit (sinon, ignorer cette partie sans erreur bloquante).

4. **Analyser le code Legacy pour les styles**
   - Partir de `${paths.legacySource}` et suivre les éléments liés aux styles, par exemple :
     - imports de fichiers CSS ou SCSS,
     - utilisation de CSS-in-JS (styled-components, emotion, MUI `sx`, etc.),
     - hooks ou contextes de thème (dark/light, design system),
     - classes utilitaires (tailwind-like, utility classes) si le projet en utilise.
   - Identifier :
     - les **styles globaux** (ex. appliqués à toute la page ou à une région principale),
     - les **styles locaux par composant/vue**,
     - les **thèmes** ou **variantes** (primary, secondary, danger, etc.).

5. **Construire les items de styles**
   - Pour chaque groupe de styles significatif, construire un `StyleItem` qui décrit :
     - la cible (souvent via un `targetStructureUcr` ou une liste de `targetStructureUcrs`),
     - le type de style (global, local, thématique, variant),
     - la nature des propriétés stylées (layout, couleur, typographie, etc.),
     - les éventuels liens avec le layout (région, type de structure).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité avec le schéma du guide.

7. **Validation interne**
   - Vérifier que :
     - toutes les références `targetStructureUcr` pointent vers des `ucr` existants de Structure,
     - les champs obligatoires sont présents pour chaque `StyleItem`,
     - la structure globale du JSON est valide.
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]` avec toutes les anomalies (styles introuvables, références cassées, etc.).

8. **Écriture de l’output**
   - Écrire `inventory.styles.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.styles.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "12",
  "stageName": "inventory.styles",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "structureInventoryLoaded": true,
    "layoutInventoryLoaded": true,
    "legacyParsed": true,
    "itemsBuilt": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

Si `inventory.layout.json` est absent, `layoutInventoryLoaded` peut être `false` et cette information doit apparaître dans `validation.issues`.

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

- `Gate ✅` si `inventory.styles.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `13-inventory.i18n.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
