# 🧩 Stage 22 – inventory.routing
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 21 – inventory.services  
**Next :** 23 – inventory.effects

---

## 🎯 Objectif

Construire l’**inventaire du routing** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires Dataflows, Async et Services (`inventory.dataflows.json`, `inventory.async.json`, `inventory.services.json`) si disponibles,
- les autres inventaires utiles (Logic, Hooks, Events, Config, Effects),
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `routing.*` / `navigation.*`.

L’objectif est de produire un JSON `inventory.routing.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **routes** pertinentes pour `${project.pageName}` (routes principales, nested routes, modales, redirections),
- les **paramètres** d’URL (route params, query params, fragments),
- les **flows de navigation** (depuis / vers la page, navigation conditionnelle),
- les **guards** (auth, droits, feature flags, préconditions métier),
- les liens entre le routing et :
  - les vues,
  - les dataflows,
  - les services,
  - l’async,
  - les événements / hooks,
  - la configuration.

Cet inventaire ne :

- ne re-spécifie pas toute la config globale du router de l’application,
- ne décrit pas tous les flux de données (cf. `inventory.dataflows`),
- ne duplique pas la logique métier (cf. `inventory.logic`).  

Il se concentre sur la **cartographie des routes et navigations** directement liées à la page `${project.pageName}`.

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
    - les fichiers de configuration de routes (router principal, sous-routers),
    - les composants de navigation (`Link`, `NavLink`, boutons qui appellent `history.push`, `router.push`, etc.),
    - les hooks de routing (`useHistory`, `useLocation`, `useParams`, `useRouter`, etc.).
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Routing**
  - `${paths.core}/guides-internals/inventory/guide.inventory.routing.md`
  - Fournit :
    - l’**objectif** du domaine Routing,
    - le **schéma JSON contractuel** de `inventory.routing.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure / Dataflows / Async / Services / Config),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour le routing respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL :
  - `routing.pageRoute`
  - `routing.nestedRoute`
  - `routing.modalRoute`
  - `routing.redirect`
  - `routing.guard`
  - `routing.navigationFlow`
- aider à identifier :
  - les définitions de route,
  - les usages de navigation (links, pushes),
  - les guards et préconditions.

> Si le bridge ne définit pas explicitement certains concepts `routing.*`, l’IA s’appuie sur les APIs du router utilisé (React Router, Next Router, etc.) et documente les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible structure les routes (fichiers, dossiers `pages/`, router custom),
    - anticiper la projection des routes Legacy dans cette architecture.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de routing dans la stack finale (naming des routes, segmentation par features, etc.),
    - ajuster la granularité des `RouteItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - rattacher les routes à des vues/conteneurs (page, layout, modales, etc.).

- **Inventaires recommandés**
  - `inventory.dataflows.json` (Stage 19)
    - pour relier les routes aux dataflows déclenchés sur navigation,
  - `inventory.async.json` (Stage 20)
    - pour relier les routes aux patterns async déclenchés lors des changements de page (préchargement, loaders),
  - `inventory.services.json` (Stage 21)
    - pour relier les routes aux services utilisés lors de l’entrée/sortie de la page.

- **Autres inventaires optionnels (si déjà générés)**  
  - `inventory.logic.json`,
  - `inventory.hooks.json`,
  - `inventory.events.json`,
  - `inventory.config.json`,
  - `inventory.effects.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.dataflows.json` / `inventory.services.json` réduit la qualité des liens fonctionnels et doit être mentionnée dans `validation.issues`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.routing.md`,
- `domain` doit valoir `"routing"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (dataflows, async, services, logic, hooks, config, effects…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.routing.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - identifier les vues/conteneurs qui représentent :
       - la page principale `${project.pageName}`,
       - des layouts / shells / wrappers,
       - des modales / panels conditionnels liés au routing.
   - À partir de `inventory.dataflows.json` (si présent) :
     - repérer les dataflows déclenchés sur navigation ou dépendant de paramètres de route.
   - À partir de `inventory.async.json` (si présent) :
     - repérer les patterns async déclenchés sur changement de route (préchargement de données, loaders).
   - À partir de `inventory.services.json` (si présent) :
     - lier les services typiquement appelés lors de l’entrée sur la page.
   - À partir du bridge :
     - exploiter les patterns `routing.*` / `navigation.*`.

4. **Analyser le code Legacy pour identifier les routes et navigations**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - la route principale associée à `${project.pageName}`,
       - les sous-routes potentielles (tabs, sections, modales associées à l’URL),
       - les chemins de navigation sortants (links/boutons qui redirigent ailleurs),
       - les guards (auth, feature flags, préconditions) autour de la page.
     - prendre en compte :
       - les définitions de route dans des fichiers de config (router),
       - les appels de navigation (`router.push`, `history.push`, `navigate`, etc.),
       - les hooks de routing (`useParams`, `useSearchParams`, `useRouter`…), pour détecter l’usage de params.

5. **Construire les items de routing**
   - Créer un `RouteItem` par :
     - route entrante significative,
     - route sortante majeure,
     - guard ou flow de navigation structurant.  
   - Pour chaque item (voir guide pour le schéma) :
     - définir le `kind` (pageRoute, nestedRoute, modalRoute, redirect, guard, navigationFlow…),
     - définir le `pathPattern` (ou identifiant de route) si applicable,
     - associer des `targetStructureUcrs` (vues/layouts/modales correspondants),
     - définir `routingSummary` :
       - `routeId`, `params`, `queryParams`,
       - `entryConditions`, `exitDestinations`,
       - `navigationTriggers` (événements / actions),
       - `dataDependencies`,
       - etc.
     - relier aux dataflows, services, async, events, hooks, config via les champs prévus.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (dataflows, async, services, logic, hooks, config, effects…),
     - les flows de navigation critiques sont bien couverts.
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.routing.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "22",
  "stageName": "inventory.routing",
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

- `Gate ✅` si `inventory.routing.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `23-inventory.effects.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
