# 🧩 Stage 19 – inventory.dataflows
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 18 – inventory.events  
**Next :** 20 – inventory.async

---

## 🎯 Objectif

Construire l’**inventaire des dataflows** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires de logique, hooks et événements (`inventory.logic.json`, `inventory.hooks.json`, `inventory.events.json`) si disponibles,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `data.*`.

L’objectif est de produire un JSON `inventory.dataflows.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **flux de données** (read / write) : queries, mutations, chargement initial, rafraîchissements,
- les **sources** (APIs HTTP, services internes, localStorage, etc.),
- les **cibles** (états, vues, domaines métier),
- les **entrées/sorties** de chaque dataflow,
- les liens entre dataflows, **vues**, **logique**, **hooks**, **événements**, **conditions**.

Cet inventaire ne détaille pas :

- l’implémentation asynchrone fine (timeouts, retries → `inventory.async`),
- la définition des services/facades techniques (`inventory.services`),
- le routing (même si des dataflows peuvent dépendre de la route → `inventory.routing`),
- les effets visuels ou de tracking (`inventory.effects`).  

Il se concentre sur les **flux de données métier** qui alimentent ou modifient l’état de la page.

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
    - les hooks de data (ex. `useQuery`, `useMutation`, hooks maison de fetch),
    - les modules d’accès au store global, caches, localStorage,
    - les modules de mapping entre data brute et modèles métier.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Dataflows**
  - `${paths.core}/guides-internals/inventory/guide.inventory.dataflows.md`
  - Fournit :
    - l’**objectif** du domaine Dataflows,
    - le **schéma JSON contractuel** de `inventory.dataflows.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Logic / Hooks / Events / Services / Async),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les dataflows respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `data.*`, par exemple :
  - `data.query`
  - `data.mutation`
  - `data.endpoint`
  - `data.cache`
  - `data.subscription`
- aider à distinguer :
  - les dataflows de **lecture** (read/query),
  - les dataflows d’**écriture** (write/mutation),
  - les dataflows **réactifs** (subscriptions, websockets).  

> Si le bridge ne définit que partiellement `data.*`, utiliser les clients HTTP connus, les hooks de data et les patterns d’appel API, puis documenter les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment les dataflows seront structurés dans la stack cible (services, hooks, facades),
    - anticiper les regroupements et factorisations futures.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de data layer dans la stack finale (ex. architecture “services + hooks de data”),
    - influencer la granularité des `DataflowItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - ancrer les dataflows aux parties de l’UI qui consomment les données.

- **Inventaires recommandés**
  - `inventory.logic.json` (Stage 15)
    - pour relier les dataflows aux unités de logique qui consomment/produisent des données,
  - `inventory.hooks.json` (Stage 17)
    - pour relier les dataflows aux hooks de data ou hooks métier,
  - `inventory.events.json` (Stage 18)
    - pour relier les dataflows aux événements déclencheurs.

- **Autres inventaires optionnels (si déjà générés dans un run précédent)**  
  *(ne sont normalement pas présents lors du premier passage, mais le stage doit pouvoir les lire s’ils existent)*
  - `inventory.async.json`,
  - `inventory.services.json`,
  - `inventory.effects.json`,
  - `inventory.routing.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence des autres inventaires n’est pas bloquante, mais doit être notée dans `validation.issues` si elle limite la qualité des liens.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.dataflows.md`,
- `domain` doit valoir `"dataflows"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (logic, hooks, events, async, services…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.dataflows.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - identifier les vues/composants consommateurs de données (listes, graphiques, sections détaillées…).
   - À partir des autres inventaires (si présents) :
     - indexer `LogicItem`, `HookItem`, `EventItem`, etc. pour lier les dataflows.
   - À partir du bridge :
     - extraire les patterns `data.*` et les indexer par `dslId`.

4. **Analyser le code Legacy pour les dataflows**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les appels HTTP (fetch, axios, clients maison, etc.),
       - les hooks de data (queries/mutations, subscriptions),
       - les accès à des stores/caches (Redux, Zustand, Query cache, etc.) en tant que dataflows,
       - les lectures/écritures dans localStorage/sessionStorage quand elles ont un impact métier,
       - les échanges via bus d’événements / websockets si pertinents.
     - pour chaque dataflow significatif :
       - identifier le **type** (query, mutation, subscription, sync cache…),
       - identifier le **contexte** (événement déclencheur, vue consommatrice),
       - identifier les **données métier** manipulées (campagnes, utilisateurs, etc.).

5. **Construire les items de dataflows**
   - Créer un `DataflowItem` par dataflow significatif (voir guide pour le schéma) :
     - définir le `kind` (query, mutation, subscription, cacheSync, etc.),
     - donner un `name` logique (ex. `"fetchCampaignsList"`, `"saveCampaign"`, `"loadUserPermissions"`),
     - associer des `targetStructureUcrs` (vues/composants consommateurs),
     - définir clairement les **inputs** et **outputs** métier dans `dataflowSummary`,
     - relier aux événements, hooks, logique, services, async, routing via les champs prévus.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (logic, hooks, events, services, async, routing…),
     - les dataflows critiques sont identifiés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.dataflows.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "19",
  "stageName": "inventory.dataflows",
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

- `Gate ✅` si `inventory.dataflows.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `20-inventory.async.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
