# 🧭 Guide de Mapping — `mapping.routing`

*(Projection des concepts `routing.*` du DSL vers le système de navigation / router de la stack cible)*

---

## 1. 🎯 Rôle du mapping `routing`

Le domaine `routing.*` du DSL décrit **la navigation et la structure des routes** :

- routes principales d’écran ;
- routes enfants / onglets / sous-sections ;
- paramètres d’URL (path params, query params) ;
- redirections ;
- guards (conditions d’accès, permissions) ;
- liens entre événements et changements de route.

La **Phase 1 — Analyse** a produit :

- `inventory.routing.json` → inventaire des UCR `routing.*` ;  
- des liens possibles avec :
  - `structure.*` / `layout.*` (zones de vue associées aux routes) ;
  - `events.*` (événements qui déclenchent une navigation) ;
  - `conditions.*` (guards, permissions, feature flags) ;
  - `hooks.*` (hooks de navigation) ;
  - `logic.*` (orchestration sur changement de route).

La **Phase 2 — Stage 42 — mapping.routing** doit :

> **Projeter chaque UCR `routing.*` vers une route concrète de la stack cible**, par exemple :
> - entrée de router (React Router, Next.js, Vue Router, etc.) ;
> - mapping vers un composant de page ;
> - mapping des paramètres d’URL ;
> - mapping des redirections / guards.

Objectif : rendre explicite **quelle URL mène à quel écran, avec quels paramètres et quelles conditions d’accès**.

---

## 2. 📦 Format JSON racine (`mapping.routing.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.routing.json`

Structure racine attendue :

```json
{
  "domain": "routing",
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

- `domain` : `"routing"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour le routing

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `routing.*` issu de `inventory.routing.json` ;
- à une route / configuration de router concrète dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.routing.json",
    "domain": "routing",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "routeId": "string",
    "path": "string",
    "targetComponentPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "routePattern": "string",
    "hints": []
  },
  "url": {
    "pathParams": [],
    "queryParams": [],
    "hash": null
  },
  "guards": {
    "conditionsUcrs": [],
    "requiredPermissions": [],
    "featureFlags": []
  },
  "relations": {
    "structureUcrs": [],
    "layoutUcrs": [],
    "eventsUcrs": [],
    "hooksUcrs": [],
    "logicUcrs": [],
    "configNames": []
  },
  "metadata": {
    "isEntryRoute": false,
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping **unique** dans `mapping.routing.json`.  
  - Préfixe recommandé : `map-routing-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `routing.*` du DSL, par ex. :
    - `routing.page` ;
    - `routing.child` ;
    - `routing.redirect` ;
    - `routing.modal`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.routing.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"routing"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire du routing.

- `toStack.stackKind`  
  - Type de route côté stack, par ex. :
    - `"pageRoute"` ;
    - `"nestedRoute"` ;
    - `"redirectRoute"` ;
    - `"modalRoute"`.

- `toStack.routeId`  
  - Identifiant logique de la route (nom de route), ex. `campaignsDetail`, `campaignsList`, etc.

- `toStack.path`  
  - Pattern d’URL (par ex. `"/campaigns/:campaignId"` ou `"/campaigns"`).

- `toStack.targetComponentPath`  
  - Chemin du composant de page cible, dérivé de `project-structure.json`, ex. :
    - `src/pages/CampaignsDetail/CampaignsDetail.view.tsx` ;
    - `src/pages/CampaignsList/CampaignsList.view.tsx`…

- `toStack.targetLayer`  
  - `"presentation"` pour les routes directement associées à des composants de page ;  
  - `"application"` pour des routes gérées par un router central.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"react-router"`, `"next-router"`, `"vue-router"`, etc.

- `toStack.routePattern`  
  - `"page"`, `"child"`, `"modal"`, `"redirect"`, `"notFound"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Utiliser un layout parent pour cette route"`, `"Prévoir un guard de permissions"`, etc.

- `url.pathParams` / `url.queryParams`  
  - Noms de paramètres d’URL importants (`"campaignId"`, `"tab"`, `"page"`, `"sort"`, etc.).

- `guards.conditionsUcrs`  
  - UCR `conditions.*` utilisés comme guards.

- `guards.requiredPermissions`  
  - Codes de permissions / rôles nécessaires (si connus).

- `guards.featureFlags`  
  - Flags requis pour activer l’accès à cette route.

- `relations.structureUcrs` / `relations.layoutUcrs`  
  - UCR de structure / layout de la page correspondante.

- `relations.eventsUcrs`  
  - UCR `events.*` déclenchant une navigation vers cette route.

- `relations.hooksUcrs`  
  - UCR `hooks.*` qui effectuent la navigation (ex. via `useNavigate`).

- `relations.logicUcrs`  
  - Logique exécutée sur changement de route (chargement initial, nettoyage, etc.).

- `metadata.isEntryRoute`  
  - `true` si c’est la route principale / initiale de la page / feature.

- `metadata.isCritical`  
  - `true` pour les routes métier majeures (page principale, accès critique).

---

## 4. ⚙️ Entrées requises pour `mapping.routing`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.routing.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.layout.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.routing.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.structure.json`
  - `inventory.layout.json`
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`

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
- `mapping.services.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Tout est basé sur :
     - les inventaires ;
     - les guides ;
     - la structure cible ;
     - les mappings précédents.

2. **Routing comme “squelette navigable” de la page**  
   - Les routes doivent refléter :
     - les écrans principaux ;
     - les sous-zones / onglets ;
     - les modales métiers importantes.

3. **Alignement fort avec la structure de pages**  
   - Chaque route doit pointer sur un composant de page / layout cohérent.  
   - Le router ne doit pas être en désaccord avec `mapping.structure` et `mapping.layout`.

4. **Guards explicites**  
   - Les conditions d’accès (permissions, flags, état métier) doivent être référencées via `guards.conditionsUcrs`, `guards.requiredPermissions`, `guards.featureFlags`.

5. **Traçabilité des navigations**  
   - Les `relations.eventsUcrs` et `relations.hooksUcrs` doivent permettre de comprendre :
     - quel événement déclenche quelle route ;
     - quel hook exécute la navigation.

6. **Priorisation des routes critiques**  
   - `metadata.isEntryRoute = true` pour la route principale ;  
   - `metadata.isCritical = true` pour les routes métier majeures.

---

## 6. Exemple simplifié de `mapping.routing.json`

```json
{
  "domain": "routing",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-routing-page-CampaignsDetail-main-1",
      "fromDsl": "routing.page",
      "sourceInventoryRef": {
        "file": "inventory.routing.json",
        "domain": "routing",
        "itemUcr": "routing-page-CampaignsDetail-main-1"
      },
      "toStack": {
        "stackKind": "pageRoute",
        "routeId": "campaignsDetail",
        "path": "/campaigns/:campaignId",
        "targetComponentPath": "src/pages/CampaignsDetail/CampaignsDetail.view.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "react-router",
        "routePattern": "page",
        "hints": [
          "Utiliser un layout parent commun aux pages campagnes"
        ]
      },
      "url": {
        "pathParams": ["campaignId"],
        "queryParams": ["tab"],
        "hash": null
      },
      "guards": {
        "conditionsUcrs": ["conditions-canViewCampaign-1"],
        "requiredPermissions": ["CAMPAIGNS_READ"],
        "featureFlags": ["feature.campaigns.enabled"]
      },
      "relations": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "eventsUcrs": ["events.ui.open-CampaignsDetail-main-1"],
        "hooksUcrs": ["hooks.routing-CampaignsDetail-main-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "configNames": ["config.routing.campaignsDetail"]
      },
      "metadata": {
        "isEntryRoute": true,
        "isCritical": true,
        "priority": "high",
        "notes": "Route principale d'accès au détail campagne."
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

- [ ] `inventory.routing.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.structure.json` et `mapping.layout.json` cohérents avec les `structureUcrs` / `layoutUcrs` référencés  
- [ ] `mapping.events.json` et `mapping.hooks.json` disponibles pour relier les navigations  
- [ ] Chaque UCR `routing.*` important a une projection dans `mapping.routing.json`  
- [ ] Tous les `MappingItem` ont `path` + `routeId` + `targetComponentPath` correctement renseignés  
- [ ] Les guards critiques sont renseignés (`guards.conditionsUcrs` / `requiredPermissions` / `featureFlags`) si connus  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.routing` (Stage 42 — Phase 2 : Interprétation)*
