# 🧩 Stage 16 – inventory.conditions
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 15 – inventory.logic  
**Next :** 17 – inventory.hooks

---

## 🎯 Objectif

Construire l’**inventaire des conditions** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- l’inventaire de logique (`inventory.logic.json`),
- éventuellement les inventaires config / data / events,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `condition.*`.

L’objectif est de produire un JSON `inventory.conditions.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **conditions** qui pilotent l’affichage, le comportement ou les transitions de la page,
- les **guards** (conditions de garde avant d’exécuter une logique ou un effet),
- les conditions basées sur la **configuration**, les **données**, les **états locaux**,
- les **issues critiques** (branches sans else, conditions complexes, duplication, etc.).

Cet inventaire **ne traite pas** :

- de la logique métier en elle-même (`inventory.logic`),
- des événements et handlers (`inventory.events`),
- des effets / side-effects (`inventory.effects`),
- des dataflows / services (`inventory.dataflows`, `inventory.services`),
- du routing (`inventory.routing`).

Il se concentre sur la **structure des conditions** et leur rôle dans le comportement global.

---

## ⚙️ Inputs

> Tous les chemins sont dérivés de `project.config.yaml` via `project.*` et `paths.*`.  
> Aucun chemin absolu ne doit être utilisé.

### 1. Configuration projet (en mémoire)

Clés utilisées :

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
  - Le stage peut suivre les imports vers :
    - les composants / modules contenant des conditions (if/else, ternaires, guards, etc.),
    - les hooks ou fonctions utilitaires qui encapsulent des conditions,
    - les modules de permissions / feature flags utilisés dans des conditions.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Conditions**
  - `${paths.core}/guides-internals/inventory/guide.inventory.conditions.md`
  - Fournit :
    - l’**objectif** du domaine Conditions,
    - le **schéma JSON contractuel** de `inventory.conditions.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure & Logic, usage des UCR),
    - les relations avec les autres inventaires (logic, config, dataflows, events…).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les références aux vues (`targetStructureUcrs`, etc.) utilisent des UCR valides,
    - s’assurer que les UCR introduits pour les conditions (`conditionUcr`, etc.) respectent le contrat global.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `condition.*`, par exemple :
  - `condition.featureFlag`
  - `condition.branchIf`
  - `condition.guard`
  - `condition.dataPresence`
  - `condition.permission`
- aider à distinguer :
  - les conditions purement visuelles (affichage/masquage),
  - les conditions métier (activation de logique),
  - les guards critiques (préconditions).

> Si les concepts `condition.*` ne sont que partiellement définis dans le bridge, le stage doit rester robuste : utiliser les patterns disponibles, et documenter les limitations dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment les conditions devront être projetées (guards dans des hooks, conditions de rendu, etc.),
    - anticiper la granularité souhaitée au niveau des composants/container.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les patterns de conditions recommandés dans la stack cible (ex. guards centralisés, hooks de permissions),
    - orienter la découpe des `ConditionItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues,
    - ancrer les conditions à des composants/sections concrets.

- **Inventaire Logic (Stage 15) — fortement recommandé**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json`
  - Rôle :
    - distinguer ce qui relève de la logique pure vs des conditions,
    - associer les conditions à certaines unités logiques si pertinent.

- **Autres inventaires optionnels**
  - `inventory.config.json` (Stage 14)
    - utile pour les conditions basées sur des flags / settings,
  - `inventory.dataflows.json` / `inventory.services.json` (plus tard dans la phase)
    - utiles pour les conditions de disponibilité de données,
  - `inventory.events.json` (Stage 18)
    - pour relier certaines conditions à des triggers d’événements (en lecture seule si déjà disponible).

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.logic.json` n’est pas bloquante mais doit être notée dans `validation.issues` si elle empêche de distinguer correctement logique/conditions.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.conditions.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.conditions.md`,
- `domain` doit valoir `"conditions"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (logic, config…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.conditions.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - repérer les vues critiques (ex. zones conditionnellement affichées).
   - À partir de `inventory.logic.json` (si présent) :
     - indexer les unités logiques pour potentiellement les lier à des conditions (via `relatedLogicUcrs`).
   - À partir du bridge :
     - extraire les patterns `condition.*` et les indexer par `dslId`.

4. **Analyser le code Legacy pour les conditions**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les `if`, `else`, `else if`, `switch`, ternaires, coalescences,
       - les guards (conditions avant exécution de logique ou d’effets),
       - les conditions dans le JSX/templating (`&&`, ternaires de rendu),
       - les conditions basées sur :
         - des flags de config,
         - la présence/absence de données,
         - des permissions/roles,
         - des états locaux.
     - ignorer les détails de logique métier (déjà couverts dans `inventory.logic`), sauf pour les référencer dans les summaries.

5. **Construire les items de conditions**
   - Créer un `ConditionItem` par condition significative (voir guide pour le schéma) :
     - définir le `kind` (branch, guard, featureFlagCheck, dataPresence, permissionCheck, etc.),
     - lier la condition à une ou plusieurs vues (`targetStructureUcrs`),
     - lier la condition à des unités de logique ou de config si pertinent (`relatedLogicUcrs`, `relatedConfigNames`),
     - résumer la condition dans `conditionSummary` (inputs, outcomes, description).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers la logique/config (si présents) sont cohérents,
     - les conditions critiques ou complexes sont signalées dans `validation.issues` si nécessaire.
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.conditions.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.conditions.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "16",
  "stageName": "inventory.conditions",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "bridgeLoaded": true,
    "structureInventoryLoaded": true,
    "logicInventoryLoaded": true,
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

- `Gate ✅` si `inventory.conditions.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `17-inventory.hooks.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
