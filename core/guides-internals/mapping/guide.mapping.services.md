# 🧭 Guide de Mapping — `mapping.services`

*(Projection des concepts `services.*` du DSL vers les services / clients backend de la stack cible)*

---

## 1. 🎯 Rôle du mapping `services`

Le domaine `services.*` du DSL décrit **les points d’accès aux systèmes externes ou aux backends** :

- appels HTTP / REST / GraphQL / RPC ;
- services métier (façades applicatives) ;
- adaptateurs vers des systèmes tiers ;
- clients de base de données exposés via un service ;
- gateways (agrégation de plusieurs backends).

La **Phase 1 — Analyse** a produit :

- `inventory.services.json` → inventaire des UCR `services.*` ;  
- des liens possibles avec :
  - `dataflows.*` (flux de données qui dépendent de ces services) ;
  - `async.*` (retries, timeouts, polling autour des services) ;
  - `logic.*` (orchestration applicative qui consomme les services) ;
  - `actions.*` (use-cases qui appellent les services).

La **Phase 2 — Stage 41 — mapping.services** doit :

> **Projeter chaque UCR `services.*` vers un service concret dans la stack cible**, par exemple :
> - module de service HTTP ;
> - client spécifique d’API ;
> - façade métier côté front ;
> - adaptateur / gateway qui encapsule plusieurs appels.

Objectif : rendre explicite **où vivent les services, comment ils sont nommés, et à quels dataflows / use-cases ils sont reliés**.

---

## 2. 📦 Format JSON racine (`mapping.services.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.services.json`

Structure racine attendue :

```json
{
  "domain": "services",
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

- `domain` : `"services"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les services

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `services.*` issu de `inventory.services.json` ;
- à un service / client concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.services.json",
    "domain": "services",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "transport": "string",
    "hints": []
  },
  "relations": {
    "dataflowsUcrs": [],
    "asyncUcrs": [],
    "logicUcrs": [],
    "actionsUcrs": [],
    "eventsUcrs": [],
    "configNames": []
  },
  "contract": {
    "endpoint": "string",
    "method": "string",
    "inputShape": {},
    "outputShape": {}
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
  - Identifiant de mapping **unique** dans `mapping.services.json`.  
  - Préfixe recommandé : `map-services-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `services.*` du DSL, par ex. :
    - `services.http` ;
    - `services.domain` ;
    - `services.gateway` ;
    - `services.thirdParty`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.services.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"services"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des services.

- `toStack.stackKind`  
  - Type de service côté stack, par ex. :
    - `"httpService"` ;
    - `"domainService"` ;
    - `"gateway"` ;
    - `"thirdPartyClient"`.

- `toStack.targetId`  
  - Nom du service / client, ex. :
    - `CampaignsService`, `BudgetService`, `FeatureFlagsClient`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module dans la stack, dérivé de `project-structure.json`, ex. :
    - `src/services/CampaignsService.ts` ;
    - `src/services/FeatureFlagsClient.ts`…

- `toStack.targetLayer`  
  - `"infrastructure"` pour les services proches du transport ;  
  - `"application"` pour des façades métier côté front.

- `contract.endpoint` / `contract.method`  
  - Description minimale de l’endpoint côté backend (URL logique + verbe HTTP / type d’appel).

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"http-client-custom"`, `"axios"`, `"fetch"`, `"graphql-client"`, etc.

- `toStack.targetPattern`  
  - `"serviceClass"`, `"serviceModule"`, `"clientFactory"`, `"gatewayModule"`, etc.

- `toStack.transport`  
  - `"http"`, `"graphql"`, `"rpc"`, `"websocket"`, etc.

- `toStack.hints[]`  
  - Conseils sur l’implémentation : `"Centraliser tous les endpoints campagne ici"`, etc.

- `relations.dataflowsUcrs`  
  - UCR `dataflows.*` qui reposent sur ce service.

- `relations.asyncUcrs`  
  - UCR `async.*` définissant retries, timeouts, polling autour de ce service.

- `relations.logicUcrs` / `relations.actionsUcrs` / `relations.eventsUcrs`  
  - UCR qui consomment ou déclenchent ce service.

- `relations.configNames`  
  - Clés de configuration (base URL, timeouts globaux, feature flags).

- `contract.inputShape` / `contract.outputShape`  
  - Représentation simplifiée des payloads, au moins sur les champs critiques.

---

## 4. ⚙️ Entrées requises pour `mapping.services`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.data.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.services.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.dataflows.json`
  - `inventory.async.json`
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
- `mapping.dataflows.json`
- `mapping.async.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur :
     - les inventaires DSL ;
     - la structure cible ;
     - les guides de stack ;
     - les mappings déjà produits.

2. **Services comme point unique d’accès backend**  
   - Les composants et hooks ne doivent pas directement gérer les endpoints ;
   - Les services exposent des méthodes stables (`getById`, `list`, `update`, etc.).

3. **Cohérence avec les dataflows**  
   - Pour chaque `dataflows.*` important, identifier le ou les `services.*` utilisés ;
   - `mapping.services` doit permettre à `mapping.dataflows` de pointer sur des services clairs.

4. **Prise en compte de l’asynchrone**  
   - Les stratégies de retry / timeout / backoff vivent dans les services ou dans des wrappers ;  
   - Relier les UCR `async.*` correspondants.

5. **Priorisation des services critiques**  
   - `metadata.isCritical = true` pour les services :
     - liés à des opérations sensibles ;
     - utilisés massivement ;
     - ou exposant des données critiques.

---

## 6. Exemple simplifié de `mapping.services.json`

```json
{
  "domain": "services",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-services-http-CampaignsService-1",
      "fromDsl": "services.http",
      "sourceInventoryRef": {
        "file": "inventory.services.json",
        "domain": "services",
        "itemUcr": "services-http-CampaignsService-1"
      },
      "toStack": {
        "stackKind": "httpService",
        "targetId": "CampaignsService",
        "targetPath": "src/services/CampaignsService.ts",
        "targetLayer": "infrastructure",
        "targetTechnology": "http-client-custom",
        "targetPattern": "serviceModule",
        "transport": "http",
        "hints": [
          "Centraliser tous les endpoints campagnes ici",
          "Prévoir une couche d'adaptation vers les viewModels"
        ]
      },
      "relations": {
        "dataflowsUcrs": [
          "dataflows-read-CampaignsDetail-byId-1",
          "dataflows-write-CampaignsDetail-update-1"
        ],
        "asyncUcrs": [
          "async.retry-CampaignsDetail-update-1"
        ],
        "logicUcrs": [
          "logic.viewLifecycle-CampaignsDetail-1"
        ],
        "actionsUcrs": [
          "action.saveCampaign-1"
        ],
        "eventsUcrs": [
          "events.ui.submit-CampaignsDetail-main-1"
        ],
        "configNames": [
          "config.http.baseUrl",
          "config.campaigns.timeoutMs"
        ]
      },
      "contract": {
        "endpoint": "/campaigns/{campaignId}",
        "method": "GET",
        "inputShape": {
          "pathParams": ["campaignId"]
        },
        "outputShape": {
          "id": "string",
          "name": "string",
          "status": "string"
        }
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Service principal pour la gestion des campagnes."
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

- [ ] `inventory.services.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.dataflows.json` et `mapping.async.json` cohérents avec les `dataflowsUcrs` / `asyncUcrs` référencés  
- [ ] Chaque UCR `services.*` important a une projection dans `mapping.services.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets + contrat minimal (`endpoint`, `method`)  
- [ ] Les services critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.services` (Stage 41 — Phase 2 : Interprétation)*
