# 🧭 Guide de Mapping — `mapping.dataflows`

*(Projection des concepts `dataflows.*` du DSL vers les flux de données concrets de la stack cible)*

---

## 1. 🎯 Rôle du mapping `dataflows`

Le domaine `dataflows.*` du DSL décrit **comment les données circulent** entre :

- les sources de données (APIs, services, stores, caches) ;
- les adaptateurs / normaliseurs ;
- les sélecteurs / projections pour la vue ;
- les écritures (create / update / delete) ;
- les synchronisations et rafraîchissements.

La **Phase 1 — Analyse** a produit :

- `inventory.dataflows.json` → inventaire des UCR `dataflows.*` ;  
- des liens possibles avec :
  - `services.*` (APIs et backends concrets) ;
  - `hooks.*` (hooks de data) ;
  - `logic.*` (orchestration de data) ;
  - `actions.*` (use-cases qui consomment / produisent ces dataflows) ;
  - `events.*` (triggers qui déclenchent un dataflow).

La **Phase 2 — Stage 39 — mapping.dataflows** doit :

> **Projeter chaque UCR `dataflows.*` vers un flux de données concret de la stack cible**, par exemple :
> - appel de service (HTTP, RPC, etc.) ;
> - hook de data (ex. `useQuery`, `useSWR`, hook custom) ;
> - sélecteur de store ;
> - adaptateur entre schéma backend et modèle de vue ;
> - pipeline de lecture/écriture avec normalisation.

Objectif : rendre explicite **d’où viennent les données, où elles vont, et sous quelle forme** dans la stack cible.

---

## 2. 📦 Format JSON racine (`mapping.dataflows.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.dataflows.json`

Structure racine attendue :

```json
{
  "domain": "dataflows",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs principaux :

- `domain` : `"dataflows"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les dataflows

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `dataflows.*` issu de `inventory.dataflows.json` ;
- à un flux de données concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.dataflows.json",
    "domain": "dataflows",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "direction": "string",
    "sourceKind": "string",
    "targetKind": "string",
    "hints": []
  },
  "relations": {
    "servicesUcrs": [],
    "hooksUcrs": [],
    "logicUcrs": [],
    "actionsUcrs": [],
    "eventsUcrs": [],
    "structureUcrs": [],
    "layoutUcrs": [],
    "configNames": []
  },
  "mapping": {
    "inputShape": {},
    "outputShape": {},
    "adapterName": "string"
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping **unique** dans `mapping.dataflows.json`.  
  - Préfixe recommandé : `map-dataflows-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `dataflows.*` du DSL, par ex. :
    - `dataflows.read` ;
    - `dataflows.write` ;
    - `dataflows.sync` ;
    - `dataflows.selector` ;
    - `dataflows.adapter`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.dataflows.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"dataflows"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des dataflows.

- `toStack.stackKind`  
  - Type de flux côté stack, par ex. :
    - `"serviceCall"` ;
    - `"dataHook"` ;
    - `"storeSelector"` ;
    - `"storeMutation"` ;
    - `"cacheSync"`.

- `toStack.targetId`  
  - Identifiant du flux dans la stack, ex. :
    - `fetchCampaignById`, `updateCampaignBudget`, `selectCampaignsList`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module cible, dérivé de `project-structure.json`, ex. :
    - `src/services/CampaignsService.ts` ;
    - `src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts` ;
    - `src/state/campaigns/selectors.ts`…

- `toStack.targetLayer`  
  - `"application"` pour les flux métiers ;  
  - `"infrastructure"` pour les services / accès externe ;  
  - `"state"` pour les sélecteurs / mutations de store.

- `toStack.direction`  
  - `"read"`, `"write"`, `"readWrite"`, `"sync"`.

- `toStack.sourceKind` / `toStack.targetKind`  
  - Ex. `sourceKind = "service"` et `targetKind = "viewModel"` ;  
  - ou `sourceKind = "store"`, `targetKind = "viewSelector"`…

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"react-query"`, `"swr"`, `"redux"`, `"zustand"`, `"http-client-custom"`, etc.

- `toStack.targetPattern`  
  - `"query"`, `"mutation"`, `"selector"`, `"adapter"`, `"syncJob"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Mettre ce flux derrière CampaignsService"`, `"Cacher la forme backend et exposer un viewModel"`, etc.

- `relations.servicesUcrs`  
  - UCR `services.*` sous-jacents.

- `relations.hooksUcrs`  
  - UCR `hooks.*` qui consomment / exposent ce flux.

- `relations.logicUcrs` / `relations.actionsUcrs` / `relations.eventsUcrs`  
  - UCR déclencheurs ou consommateurs du flux.

- `relations.structureUcrs` / `relations.layoutUcrs`  
  - Parties d’UI impactées par ce flux.

- `relations.configNames`  
  - Configs / flags pouvant influencer ce flux (pagination, limites, options).

- `mapping.inputShape` / `mapping.outputShape`  
  - Représentation simplifiée de la forme d’entrée et de sortie (clés importantes).

- `mapping.adapterName`  
  - Nom d’un adaptateur dédié, ex. `adaptCampaignFromApi`, `buildCampaignPayload`, etc.

---

## 4. ⚙️ Entrées requises pour `mapping.dataflows`

### 4.1. Configuration (obligatoire)

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `stack.custom`
- `gates.*`
- `stages.*`

### 4.2. Artefacts Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.data.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.dataflows.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.services.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.actions.json`
  - `inventory.events.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`
- `mapping.hooks.json`
- `mapping.events.json`
- `mapping.services.json` (lorsqu’il sera produit)

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions s’appuient sur les inventaires, la structure cible, les guides et les mappings existants.

2. **Séparation claire Service ↔ Dataflow ↔ Vue**  
   - Le service représente l’accès brut ;  
   - Le dataflow représente **le flux logique** pour un cas d’usage (avec adaptateurs) ;  
   - La vue consomme le résultat (via hook / sélecteur).

3. **Mise en avant des adaptateurs**  
   - Quand le schéma backend ne correspond pas à la vue, rendre explicite `mapping.adapterName`, `inputShape`, `outputShape`.

4. **Positionnement correct des dataflows**  
   - `targetLayer = "infrastructure"` pour les flux collés aux services ;  
   - `targetLayer = "application"` pour des flux scénarisés pour la vue.

5. **Priorisation des flux critiques**  
   - `metadata.isCritical = true` pour les flux de données majeurs (chargement principal d’écran, opérations d’écriture sensibles).

---

## 6. Exemple simplifié de `mapping.dataflows.json`

```json
{
  "domain": "dataflows",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-dataflows-read-CampaignsDetail-byId-1",
      "fromDsl": "dataflows.read",
      "sourceInventoryRef": {
        "file": "inventory.dataflows.json",
        "domain": "dataflows",
        "itemUcr": "dataflows-read-CampaignsDetail-byId-1"
      },
      "toStack": {
        "stackKind": "dataHook",
        "targetId": "useCampaignById",
        "targetPath": "src/pages/CampaignsDetail/hooks/useCampaignById.ts",
        "targetLayer": "application",
        "targetTechnology": "react-query",
        "targetPattern": "query",
        "direction": "read",
        "sourceKind": "service",
        "targetKind": "viewModel",
        "hints": [
          "Déléguer l'appel HTTP à CampaignsService.getById",
          "Normaliser la réponse vers un modèle de vue unique"
        ]
      },
      "relations": {
        "servicesUcrs": ["service.http.CampaignsService.getById-1"],
        "hooksUcrs": ["hooks.view-CampaignsDetail-main-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "actionsUcrs": [],
        "eventsUcrs": ["events.ui.open-CampaignsDetail-main-1"],
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "configNames": ["feature.campaignsDetail.enabled"]
      },
      "mapping": {
        "inputShape": {
          "params": ["campaignId"]
        },
        "outputShape": {
          "campaign": {
            "id": "string",
            "name": "string",
            "status": "string"
          }
        },
        "adapterName": "adaptCampaignFromApi"
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Flux principal de lecture du détail campagne."
      }
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 7. ✅ Checklist de validation

- [ ] `inventory.dataflows.json` présent et déclaré valide dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.services.json` (si présent) cohérent avec les `servicesUcrs` référencés  
- [ ] Chaque UCR `dataflows.*` important a une projection dans `mapping.dataflows.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets + `direction` + `sourceKind` + `targetKind`  
- [ ] Les flux critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.dataflows` (Stage 39 — Phase 2 : Interprétation)*
