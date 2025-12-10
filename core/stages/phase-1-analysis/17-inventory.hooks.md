# 🧩 Stage 17 – inventory.hooks
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 16 – inventory.conditions  
**Next :** 18 – inventory.events

---

## 🎯 Objectif

Construire l’**inventaire des hooks** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires de logique (`inventory.logic.json`) et de conditions (`inventory.conditions.json`) si disponibles,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `hooks.*`.

L’objectif est de produire un JSON `inventory.hooks.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **hooks React standards** et **hooks custom** utilisés,
- leur **rôle fonctionnel et logique**,
- leurs **dépendances** (états, dataflows, config, conditions),
- leurs liens avec les **vues** et les autres domaines (logic, events, effects, dataflows).

Cet inventaire ne traite pas directement :

- des détails de logique métier (→ `inventory.logic`),
- des conditions (→ `inventory.conditions`),
- des handlers d’événements (→ `inventory.events`),
- des effets side-effects en tant que tels (→ `inventory.effects`),
- du câblage de données (→ `inventory.dataflows`, `inventory.services`).  

Il se concentre sur le **rôle d’orchestration des hooks** (là où la logique, les conditions, les effets et la data se croisent souvent dans React).

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
    - les fichiers de composants qui utilisent des hooks,
    - les fichiers de hooks custom (ex. `useXxx`, `hooks/useXxx.ts`),
    - les modules où la logique, la data ou les effets sont orchestrés via des hooks.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Hooks**
  - `${paths.core}/guides-internals/inventory/guide.inventory.hooks.md`
  - Fournit :
    - l’**objectif** du domaine Hooks,
    - le **schéma JSON contractuel** de `inventory.hooks.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec les autres inventaires, usage des UCR),
    - les relations avec Logic / Conditions / Dataflows / Effects.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les hooks respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `hooks.*`, par exemple :
  - `hooks.state` (useState, useReducer),
  - `hooks.effect` (useEffect, useLayoutEffect),
  - `hooks.memoization` (useMemo, useCallback),
  - `hooks.customLogic` (hooks métier),
  - `hooks.data` (hooks de data fetching),
  - `hooks.form` (hooks de formulaires).
- aider à caractériser les hooks par rôle (state, data, orchestration, UI logic…).

> Si le bridge ne définit pas clairement `hooks.*`, le stage doit :
> - s’appuyer sur les patterns de nommage (`useXxx`),  
> - utiliser les API standard React comme repères,  
> - documenter les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible souhaite organiser les hooks (hooks UI, hooks métier, hooks data),
    - anticiper la projection (phase 2) des hooks Legacy vers une architecture de hooks React 19 standardisée.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître la stratégie d’architecture des hooks dans la stack cible (ex. “toute logique métier dans des hooks `useDomainXxx`”),
    - ajuster la granularité des `HookItem` en conséquence.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - permettre de mapper les hooks aux parties de l’arbre de rendu qu’ils pilotent.

- **Inventaire Logic (Stage 15) — recommandé**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json`
  - Rôle :
    - relier certains hooks à des unités logiques (ex. hooks qui encapsulent de la logique métier).

- **Inventaire Conditions (Stage 16) — recommandé**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.conditions.json`
  - Rôle :
    - relier certains hooks à des conditions (ex. hooks qui appliquent des guards ou filtrent selon des flags).

- **Autres inventaires optionnels**
  - `inventory.config.json` (flags / settings utilisés dans les hooks),
  - `inventory.dataflows.json` / `inventory.services.json` (hooks de data fetching),
  - `inventory.effects.json` (hooks de type `useEffect` fortement liés aux effets).

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.logic.json` ou `inventory.conditions.json` ne bloque pas, mais doit être notée dans `validation.issues` si cela limite les liens transverses.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.hooks.md`,
- `domain` doit valoir `"hooks"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (logic, conditions, dataflows…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.conditions.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.hooks.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - identifier les vues / containers associés à des hooks (composants principaux, containers logiques).
   - À partir de `inventory.logic.json` (si présent) :
     - indexer les `LogicItem` pour pouvoir relier certains hooks à des unités logiques.
   - À partir de `inventory.conditions.json` (si présent) :
     - indexer les `ConditionItem` pour relier certains hooks à des conditions centrales.
   - À partir du bridge :
     - extraire les patterns `hooks.*` et les indexer par `dslId`.

4. **Analyser le code Legacy pour les hooks**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les appels à `useState`, `useReducer`, `useMemo`, `useCallback`, `useEffect`, etc.,
       - les hooks custom (`useXxx`) définis dans la page ou importés,
       - les hooks de data (ex. `useQuery`, `useMutation`, etc.) s’ils existent,
       - les hooks d’intégration (formulaires, routing, store global).
     - pour chaque hook :
       - identifier son rôle principal (state, effect, logic orchestrator, data, form, routing…),
       - repérer les dépendances (états, dataflows, config, conditions).

5. **Construire les items de hooks**
   - Créer un `HookItem` par hook significatif (voir guide pour le schéma) :
     - définir le `kind` (stateHook, effectHook, dataHook, customLogicHook, etc.),
     - donner un `name` logique (souvent lié au nom du hook ou à son rôle métier),
     - associer des `targetStructureUcrs` (composants/vues impactées),
     - lier aux unités de logique / conditions / dataflows / config via les champs prévus,
     - résumer le rôle du hook dans `hookSummary`.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (logic, conditions, dataflows…),
     - les hooks critiques sont identifiés (via `metadata`).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.hooks.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "17",
  "stageName": "inventory.hooks",
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

- `Gate ✅` si `inventory.hooks.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `18-inventory.events.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
