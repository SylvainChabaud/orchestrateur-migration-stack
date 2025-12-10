# 🧩 Stage 21 – inventory.services
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 20 – inventory.async  
**Next :** 22 – inventory.routing

---

## 🎯 Objectif

Construire l’**inventaire des services** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires Dataflows et Async (`inventory.dataflows.json`, `inventory.async.json`) si disponibles,
- les autres inventaires utiles (Logic, Hooks, Events, Config, Routing, Effects),
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `data.*` / `services.*` et apparentés.

L’objectif est de produire un JSON `inventory.services.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **services/facades techniques** (clients API, repositories, adaptateurs),
- leurs **opérations exposées** (méthodes publiques côté domaine),
- les **dataflows** qu’ils encapsulent ou orchestrent,
- les **relations** avec l’UI, la logique métier, l’async et la config.

Cet inventaire ne :

- ne redécrit pas chaque appel HTTP individuel (`inventory.dataflows`),
- ne remplace pas l’inventaire async (`inventory.async`),
- ne modélise pas la logique métier pure (`inventory.logic`).  

Il se concentre sur la **couche de services**, c’est‑à‑dire les “portes d’entrée techniques” utilisées par la page pour accéder aux données et intégrations externes.

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
    - les fichiers `*Service.*`, `*Api.*`, `*Repository.*`, etc.,
    - les clients HTTP génériques et wrappers métiers,
    - les modules d’intégration tierce (tracking externe, outils 3rd party…).
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Services**
  - `${paths.core}/guides-internals/inventory/guide.inventory.services.md`
  - Fournit :
    - l’**objectif** du domaine Services,
    - le **schéma JSON contractuel** de `inventory.services.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Dataflows / Async / Logic / Config),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les services respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL de couche data/services, par exemple :
  - `data.endpoint`
  - `data.serviceFacade`
  - `data.repository`
- aider à distinguer :
  - un **dataflow** (flux concret de données),
  - d’un **service** (abstraction ou facade encapsulant des dataflows).

> Si le bridge ne définit pas explicitement les services, l’IA se base sur les patterns de naming/dossiers (`services/`, `api/`, `repositories/`) et documente les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible structure la couche services (services fonctionnels, repositories, adaptateurs),
    - anticiper la projection des services Legacy dans cette architecture cible.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de services dans la stack finale (ex. `CampaignsService`, `UserService`, etc.),
    - ajuster la granularité des `ServiceItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - rattacher les services à des zones de l’UI (vues qui s’appuient sur eux).

- **Inventaires fortement recommandés**
  - `inventory.dataflows.json` (Stage 19)
    - pour relier les services aux dataflows qu’ils encapsulent,
  - `inventory.async.json` (Stage 20)
    - pour relier les services aux patterns async associés.

- **Autres inventaires optionnels (si déjà générés)**  
  - `inventory.logic.json`,
  - `inventory.hooks.json`,
  - `inventory.events.json`,
  - `inventory.config.json`,
  - `inventory.routing.json`,
  - `inventory.effects.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
Sans `inventory.dataflows.json`, la qualité de l’inventaire Services est fortement dégradée et le stage doit l’indiquer dans `validation.issues`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.services.md`,
- `domain` doit valoir `"services"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (dataflows, async, logic, hooks, config, routing…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.services.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - repérer les vues/composants utilisant des services (indirectement via dataflows/hooks).
   - À partir de `inventory.dataflows.json` (si présent) :
     - indexer les `DataflowItem` par endpoints/services utilisés.
   - À partir de `inventory.async.json` (si présent) :
     - repérer les patterns async associés à certains services ou groupes d’appels.
   - À partir des autres inventaires (logic, hooks, events, config, routing, effects) :
     - constituer des index pour lier les opérations de service aux usages concrets.
   - À partir du bridge :
     - exploiter les patterns `data.serviceFacade` / `data.repository` si définis.

4. **Analyser le code Legacy pour identifier les services**
   - Partir de `${paths.legacySource}` et :
     - suivre les imports vers les modules de services / API / repositories,
     - repérer :
       - les classes ou objets de service (ex. `CampaignsService`, `apiClient`),
       - les fonctions de service pures (ex. `fetchCampaigns`, `saveCampaign`),
       - les adaptateurs ou wrappers au-dessus des clients HTTP,
       - les services d’intégration externe (tracking, feature flags, etc.).
     - cartographier pour chaque service :
       - ses **opérations exposées**,
       - les **dataflows** manipulés (si `inventory.dataflows` disponible),
       - les **patterns async** associés (retry, parallel, etc.),
       - la **logique métier** reliée (préparation des payloads, transformations).

5. **Construire les items de services**
   - Créer un `ServiceItem` par service significatif (voir guide pour le schéma) :
     - définir le `kind` (httpService, domainService, repository, externalIntegration, cacheService, featureService, etc.),
     - donner un `name` logique (ex. `"CampaignsService"`, `"PromoBoostApi"`, `"FeatureFlagService"`),
     - décrire les opérations exposées dans `serviceSummary` (liste d’opérations, responsabilités),
     - associer des `targetStructureUcrs` (vues/composants dépendants),
     - relier aux dataflows, async, logique, hooks, config, routing via les champs prévus.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (dataflows, async, logic, hooks, config, routing…),
     - les services critiques ou surchargés sont signalés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.services.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "21",
  "stageName": "inventory.services",
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

- `Gate ✅` si `inventory.services.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `22-inventory.routing.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
