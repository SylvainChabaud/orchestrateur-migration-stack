# 🧩 Stage 20 – inventory.async
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 19 – inventory.dataflows  
**Next :** 21 – inventory.services

---

## 🎯 Objectif

Construire l’**inventaire de l’asynchrone** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires dataflows, services, hooks et events (`inventory.dataflows.json`, `inventory.services.json`, `inventory.hooks.json`, `inventory.events.json`) si disponibles,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `effect.async`, `data.*` et apparentés.

L’objectif est de produire un JSON `inventory.async.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **stratégies asynchrones** utilisées (promises, `async/await`, parallélisme, séquentialisation),
- les **mécanismes de retry / backoff / timeout / cancellation**,
- les **patterns de concurrence** (appels parallèles, en série, “race”),
- les **mécanismes de throttle / debounce / polling**,
- les liens entre ces mécanismes et les **dataflows**, **services**, **events**, **effects** et **vues**.

Cet inventaire ne :

- ne répète pas la description des flux de données (`inventory.dataflows`),
- ne redéfinit pas la liste des services (`inventory.services`),
- ne liste pas tous les effets (`inventory.effects`),
- ne se substitue pas à la logique métier (`inventory.logic`).  

Il se concentre sur **“comment” les choses se passent dans le temps** (timing, ordre, concurrence, fiabilité).

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
    - les services / clients HTTP,
    - les hooks de data ou hooks métier asynchrones,
    - les utilitaires d’orchestration d’appels (helpers de retry, parallélisation, debounce, etc.).
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Async**
  - `${paths.core}/guides-internals/inventory/guide.inventory.async.md`
  - Fournit :
    - l’**objectif** du domaine Async,
    - le **schéma JSON contractuel** de `inventory.async.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Dataflows / Services / Events / Effects),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour l’asynchrone respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts :
  - `effect.async`
  - `data.query`, `data.mutation`, `data.subscription`
  - autres patterns asynchrones (promises, callbacks, timers).
- aider à caractériser :
  - les **points d’async** (où commence l’asynchrone),
  - les **stratégies** (retry, timeout, parallélisme, séquence).  

> Si le bridge ne définit pas explicitement les concepts async, s’appuyer sur les constructions JS/TS (`Promise`, `async/await`, `setTimeout`, helpers de retry…) et documenter les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible souhaite structurer l’asynchrone (services, hooks, middlewares),
    - anticiper la projection vers des patterns standard (React Query, services centralisés, etc.).

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de gestion d’async dans la stack finale (où mettre le retry, comment gérer les spinners, etc.),
    - ajuster la granularité des `AsyncItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - rattacher les patterns async à des vues concrètes (spinners, loaders, boutons, etc.).

- **Inventaires recommandés**
  - `inventory.dataflows.json` (Stage 19)
    - pour relier les stratégies async aux dataflows sous-jacents,
  - `inventory.services.json` (Stage 21 — si déjà disponible dans un run précédent)
    - pour relier l’async aux facades/services techniques,
  - `inventory.events.json` (Stage 18)
    - pour relier l’async aux événements déclencheurs,
  - `inventory.effects.json` (Stage 23 — si déjà disponible dans un run précédent)
    - pour relier l’async à certains effets asynchrones (toasts différés, tracking, etc.).

- **Autres inventaires optionnels (si déjà générés)**  
  - `inventory.hooks.json`,
  - `inventory.routing.json`,
  - `inventory.config.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.dataflows.json` réduit fortement la qualité de l’inventaire Async et doit être mentionnée dans `validation.issues`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.async.md`,
- `domain` doit valoir `"async"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (dataflows, services, events, hooks, routing…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.async.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - repérer les vues/composants affichant des loaders, spinners, états d’attente/erreur.
   - À partir de `inventory.dataflows.json` (si présent) :
     - indexer les `DataflowItem` et repérer ceux qui ont une dimension async marquée (polling, calls multiples).
   - À partir des autres inventaires (services, events, hooks, effects, routing, config) :
     - indexer les éléments utiles pour relier l’asynchrone aux autres domaines.
   - À partir du bridge :
     - extraire les patterns `effect.async` et les indexer par `dslId`.

4. **Analyser le code Legacy pour les patterns async**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les fonctions marquées `async`,
       - les usages de `await` / `Promise.then` / `Promise.all` / `Promise.race`,
       - les helpers de retry / backoff / timeout / cancellation,
       - les patterns de polling (setInterval, requêtes répétées),
       - les mécanismes de debounce/throttle,
       - les parallélisations explicites d’appels (plusieurs dataflows en parallèle).
     - relier ces patterns :
       - aux dataflows identifiés,
       - aux services/facades,
       - aux événements déclencheurs,
       - aux effets asynchrones (ex. affichage de toast après réponse).

5. **Construire les items async**
   - Créer un `AsyncItem` par **pattern async significatif** (voir guide pour le schéma) :
     - définir le `kind` (simpleAsyncCall, parallelCalls, sequencedCalls, retryPattern, pollingPattern, debouncePattern, etc.),
     - donner un `name` logique (ex. `"parallelFetchCampaignsAndStats"`, `"retrySaveCampaignWithBackoff"`),
     - associer des `targetStructureUcrs` (vues impactées par l’async : spinners, boutons, sections),
     - lier aux dataflows / services / events / hooks via les champs prévus,
     - résumer la stratégie dans `asyncSummary` (ordre, gestion d’erreurs, cancellation, etc.).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (dataflows, services, events, hooks, routing, config…),
     - les patterns async risqués sont signalés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.async.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "20",
  "stageName": "inventory.async",
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

- `Gate ✅` si `inventory.async.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `21-inventory.services.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
