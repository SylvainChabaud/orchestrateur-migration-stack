# 🧩 Stage 10 – inventory.structure
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 02 – legacy-to-dsl-bridge-builder  
**Next :** 11 – inventory.layout

---

## 🎯 Objectif

Construire l’**inventaire de structure** pour la page `${project.pageName}` en analysant le code Legacy situé à `${paths.legacySource}`.  
L’objectif est de produire un JSON `inventory.structure.json` décrivant de manière **canonique** :

- la hiérarchie des vues / composants de la page,
- les types de vues (root, container, presentational, fragment, slot, portal…),
- les liens parent → enfant et les régions de layout associées (si connues),
- les points d’accroche vers la logique, les événements et les données (sans les détailler).

Cet inventaire doit :

- respecter le **DSL interne** (`structure.*`, éventuellement quelques `layout.*` de haut niveau),
- respecter le **guide d’inventaire Structure** et le **guide UCR**,
- être complètement **agnostique de la stack cible** (React 19),
- servir de base à tous les autres inventaires (logic, layout, events, dataflows…).

---

## ⚙️ Inputs

> Tous les chemins ci‑dessous sont **dérivés exclusivement** du fichier de configuration actuellement chargé (`project.config.yaml`).  
> Aucun chemin absolu ne doit être codé en dur dans la logique d’exécution.

### 1. Configuration projet

Le runtime a déjà chargé le fichier de configuration principal.  
Ce stage s’appuie uniquement sur les clés suivantes :

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

> Le stage **ne doit pas** redéfinir ces valeurs, seulement les lire.

### 2. Code Legacy (lecture seule)

- `${paths.legacySource}`  
  - Fichier d’entrée principal de la page Legacy (ex : composant racine, template, module…).
  - Le code doit être lu **in situ** dans le repo (répertoire applicatif principal).
  - ❌ Ne pas copier le Legacy dans `${paths.workspace}`.

### 3. Guides core (lecture seule)

- **Guide d’inventaire Structure**
  - `${paths.core}/guides-internals/inventory/guide.inventory.structure.md`
  - Rôle :
    - définir le **schéma JSON contractuel** de `inventory.structure.json`,
    - préciser les champs obligatoires / optionnels,
    - expliciter les contraintes (unicité UCR, absence de cycles, etc.),
    - décrire les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Rôle :
    - définir la notion d’**UCR** (identifiant canonique unique),
    - expliquer les règles de nommage et de stabilité,
    - préciser comment référencer les entités entre inventaires.
  - Utilisation dans cette étape :
    - générer un `ucr` unique pour chaque nœud de structure,
    - garantir la stabilité des `ucr` entre les différents runs,
    - préparer la réutilisation des `ucr` par les inventaires Logic, Layout, Events, Dataflows…

### 4. Bridge Legacy → DSL

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
  - Fichier généré en Phase 0 (Stage 02).  
  - Rôle dans cette étape :
    - indiquer comment reconnaître dans le Legacy les concepts DSL :
      - `structure.viewNode`
      - `structure.rootView`
      - `structure.childView`
      - `structure.containerView`
      - `structure.presentationalView`
      - `structure.fragment`
      - `structure.slot`
      - `structure.portal`
      - `structure.viewHierarchy`
    - éviter d’encoder des heuristiques de framework dans le stage,
    - assurer une interprétation cohérente et stable de la structure Legacy.

### 5. Guides de stack & structure cible (Phase 0)

- **Guides de stack (Stage 00 – stack-guides-builder)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais recommandé)
  - Utilisation :
    - comprendre la forme générale attendue de la structure cible,
    - donner du contexte à l’IA sur les conventions d’architecture (pages, layouts, composants).

- **Spécification de structure cible (Stage 01 – project-structure-spec-builder)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - aligner l’inventaire de structure Legacy sur la structure cible,
    - vérifier que les vues inventoriées pourront être projetées dans l’arborescence cible,
    - repérer les écarts majeurs (vues orphelines, module non mappé…).

### 6. Outputs précédents

- Aucun inventaire d’analyse n’est requis pour ce stage.  
  `inventory.structure` est le **premier inventaire** de la Phase 1 – Analyse.

---

## 📤 Outputs

Tous les outputs doivent vivre sous `${paths.workspace}` et dériver de `project.name` et `project.pageName`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`

Contraintes :

- doit respecter exactement le schéma défini dans `guide.inventory.structure.md`,
- JSON strictement sérialisable (aucun commentaire, aucune clé inconnue),
- `domain` doit valoir `"structure"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- chaque entrée de `items[]` doit contenir un `ucr` unique (règles UCR),
- toutes les références (`parentUcr`, `childrenUcrs[]`) doivent être cohérentes.

---

## 🧠 Actions

Dérouler les actions suivantes dans l’ordre :

1. **Charger la configuration en mémoire (si ce n’est pas déjà fait)**
   - Utiliser le contexte déjà initialisé à partir de `project.config.yaml`.
   - Récupérer :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`, `paths.stages`,
     - `stack.custom` si nécessaire pour le contexte,
     - `gates.*` pour la logique de Gate.

2. **Charger le contexte structurel cible**
   - Lire `${paths.workspace}/projects/${project.name}/stack/project-structure.json` (si présent).
   - Lire les stack-guides `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` si disponibles.

3. **Charger les guides core**
   - Lire `${paths.core}/guides-internals/inventory/guide.inventory.structure.md` pour connaître :
     - le schéma JSON attendu,
     - les champs obligatoires / optionnels,
     - les contraintes contractuelles.
   - Lire `${paths.core}/guides-internals/globals/guide.ucr.md` pour :
     - appliquer les règles de génération d’UCR,
     - garantir l’unicité et la stabilité des identifiants.

4. **Charger le bridge Legacy → DSL**
   - Lire `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`.
   - Filtrer les `dslConcepts` relatifs à la Structure (`structure.*`) et, si pertinent, quelques concepts `layout.*` de haut niveau.
   - Construire en mémoire un mini-index :
     - `dslId` → `legacyPatterns[]`.

5. **Analyser le code Legacy**
   - Lire `${paths.legacySource}`.
   - Identifier :
     - la **vue racine** (`structure.rootView`),
     - les **child views** (`structure.childView`),
     - les **container/presentational views** (`structure.containerView`, `structure.presentationalView`),
     - les fragments, slots, portails si applicables.
   - Construire la hiérarchie `parent → children` complète.

6. **Construire les items d’inventaire**
   - Pour chaque vue détectée :
     - générer un `ucr` (via les règles du guide UCR),
     - déterminer :
       - `name` logique,
       - `type` (root/container/presentational/fragment/slot/portal…),
       - `sourcePath`,
       - `parentUcr` (sauf root),
       - `childrenUcrs[]`,
       - `dslTags[]` (ex : `["structure.viewNode", "structure.containerView"]`),
       - `metadata` minimal (ex : `layoutRegion`, `displayName`, `isRoutedEntry`, etc.).

7. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Ne pas ajouter de clés non documentées dans `guide.inventory.structure.md`.

8. **Validation interne**
   - Vérifier que :
     - tous les `ucr` sont uniques,
     - tous les `parentUcr` référencent un `ucr` existant ou sont `null`,
     - tous les `childrenUcrs[]` référencent des `ucr` existants,
     - la hiérarchie n’a pas de cycle,
     - les champs obligatoires sont présents.
   - Renseigner `validation.status` :
     - `"valid"` si aucune erreur bloquante,
     - `"rejected"` sinon, avec détail dans `validation.issues[]`.

8. **Validation du schéma JSON (optionnelle)**
   - Si `validation.enableSchemaValidation = true` dans la configuration :
     - Charger le schéma depuis `${validation.schemasPath}/inventory.structure.schema.json`
     - Valider `inventoryRoot` contre ce schéma
     - En cas d'erreur de validation :
       - Mode `strict` : ajouter les erreurs dans `validation.issues[]`, fixer `status = "rejected"`, préparer `Gate ❌`
       - Mode `warning` : ajouter des warnings dans `validation.issues[]`, continuer normalement
     - Voir `${paths.core}/guides-internals/globals/guide.json-schema-validation.md` pour les détails d'implémentation

9. **Écriture de l'output**
   - Écrire `inventory.structure.json` dans le chemin cible :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
   - Remplacer tout JSON existant pour cette page / ce domaine.
   - Ne pas modifier les inventaires des autres domaines.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "10",
  "stageName": "inventory.structure",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "bridgeLoaded": true,
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

- `Gate ✅` si `inventory.structure.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : Legacy illisible, bridge manquant, schéma violé).

---

## 📦 Next

> Continuer avec `11-inventory.layout.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
