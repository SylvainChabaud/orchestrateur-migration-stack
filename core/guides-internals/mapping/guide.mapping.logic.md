# 🧭 Guide de Mapping — `mapping.logic`

*(Projection des concepts `logic.*` du DSL vers la logique applicative de la stack cible)*

---

## 1. 🎯 Rôle du mapping `logic`

Le domaine `logic.*` du DSL décrit :

- la logique applicative côté front (ou côté client) ;
- l’orchestration interne d’une vue (chargement, rafraîchissement, états dérivés) ;
- la logique de validation, d’activation/désactivation, de filtrage ;
- la coordination entre dataflows, events, services, effets, actions.

La **Phase 1 — Analyse** a produit :

- `inventory.logic.json` → inventaire des UCR `logic.*` ;
- des liens avec :
  - `events.*` (triggers, handlers) ;
  - `dataflows.*` (flux de données, sélecteurs, adaptateurs) ;
  - `services.*` (APIs, backends) ;
  - `actions.*` (use-cases métier).

La **Phase 2 — Stage 35 — mapping.logic** doit :

> **Projeter chaque UCR `logic.*` vers un artefact de logique de la stack cible**  
> (hook, service applicatif, contrôleur, view-model, store, etc.), en s’appuyant sur :
>
> - les guides de stack (architecture front, patterns React/Vue/etc.) ;
> - la structure cible (`project-structure.json`) ;
> - les mappings déjà produits (`mapping.structure.json`, `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, etc.) ;
> - les inventaires couplés (events, dataflows, services, actions, effects).

L’objectif est d’obtenir une cartographie claire de **“où vit la logique”** dans la stack cible, avec une traçabilité complète vers DSL et UCR.

---

## 2. 📦 Format JSON racine (`mapping.logic.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.logic.json`

Structure racine attendue :

```json
{
  "domain": "logic",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs :

- `domain` : `"logic"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour la logique

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `logic.*` issu de `inventory.logic.json` ;
- à un artefact de logique dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.logic.json",
    "domain": "logic",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "hints": []
  },
  "relations": {
    "structureUcrs": [],
    "layoutUcrs": [],
    "eventUcrs": [],
    "dataflowUcrs": [],
    "serviceUcrs": [],
    "actionUcrs": [],
    "effectUcrs": [],
    "configNames": []
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
  - Identifiant de mapping **unique** dans `mapping.logic.json`.  
  - Préfixe recommandé : `map-logic-…`, dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `logic.*` du DSL, par ex. :
    - `logic.viewLifecycle`, `logic.formValidation`,  
    - `logic.derivedState`, `logic.filtering`,  
    - `logic.userFlowStep`, etc.

- `sourceInventoryRef.file`  
  - Toujours `"inventory.logic.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"logic"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire logique.

- `toStack.stackKind`  
  - Type d’artefact de logique dans la stack cible, par ex. :
    - `"viewHook"` (hook de vue, ex. `useCampaignsDetail`) ;
    - `"formHook"` (hook de formulaire) ;
    - `"controller"` (contrôleur de page ou de section) ;
    - `"storeSlice"` (slice de store Redux/Zustand, etc.) ;
    - `"serviceFacade"` (façade applicative vers des services).

- `toStack.targetId`  
  - Nom du hook / contrôleur / service, par ex. :  
    - `useCampaignsDetail`, `useCampaignForm`, `CampaignsDetailController`, `useCampaignFiltersStore`…

- `toStack.targetPath`  
  - Chemin relatif dans la stack cible, dérivé de `project-structure.json`, par ex. :  
    - `src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts` ;  
    - `src/pages/CampaignsDetail/stores/useCampaignFiltersStore.ts`.

- `toStack.targetLayer`  
  - Couche ciblée, par ex. `"application"` (logique d’orchestration) ou `"presentation"` (logique locale simple).

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Par ex. `"react"`, `"vue"`, `"svelte"`, `"solid"`, `"redux"`, `"zustand"`, `"xstate"`…

- `toStack.targetPattern`  
  - Par ex. `"viewHook"`, `"formHook"`, `"controller"`, `"storeSlice"`, `"serviceFacade"`, `"stateMachine"`, etc.

- `toStack.hints[]`  
  - Conseils concrets (ex. `"Extraire la logique asynchrone dans un hook dédié"`, `"Utiliser le service de notifications partagé"`).

- `relations.eventUcrs`  
  - UCR d’events liés (triggers, handlers).

- `relations.dataflowUcrs`  
  - UCR de dataflows consommés ou produits.

- `relations.serviceUcrs`  
  - UCR de services (APIs, backends, adapters).

- `relations.actionUcrs`  
  - UCR d’actions métier / use-cases liés.

- `relations.effectUcrs`  
  - UCR d’effets (side-effects) associés.

- `relations.configNames`  
  - Noms de configs (feature flags, paramètres) impactant la logique.

---

## 4. ⚙️ Entrées requises pour `mapping.logic`

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

- Structure cible du projet :  
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- Bridge Legacy → DSL :  
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- Guides de stack (architecture / logique) :  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.logic.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.logic.json` (inventaire primaire)  
- `inventories-summary.json`  
- éventuellement (pour enrichir les relations) :
  - `inventory.events.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.actions.json`
  - `inventory.effects.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Toute décision de projection logique se base sur :
     - les inventaires DSL ;
     - les guides de stack ;
     - la structure cible ;
     - les mappings existants.

2. **Séparation claire des responsabilités**  
   - Bien distinguer :
     - logique de vue (view hooks) ;
     - logique de formulaire ;
     - logique de store / état partagé ;
     - logique de services / façades.

3. **Respect de `project-structure.json`**  
   - Les `targetPath` doivent toujours être compatibles avec l’arborescence prévue (dossiers `hooks`, `stores`, `services`, etc. s’ils existent).

4. **Traçabilité cross-domain**  
   - Les `relations.*` doivent refléter les dépendances entre logique, events, dataflows, services, actions, etc.

5. **Préparation de la Phase 3**  
   - Le mapping doit être suffisamment précis pour que :
     - la génération de hooks, stores, contrôleurs, etc. soit guidée ;
     - les tests puissent cibler les bons artefacts de logique.

---

## 6. Exemple simplifié de `mapping.logic.json`

```json
{
  "domain": "logic",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-logic-viewLifecycle-CampaignsDetail-1",
      "fromDsl": "logic.viewLifecycle",
      "sourceInventoryRef": {
        "file": "inventory.logic.json",
        "domain": "logic",
        "itemUcr": "logic-viewLifecycle-CampaignsDetail-1"
      },
      "toStack": {
        "stackKind": "viewHook",
        "targetId": "useCampaignsDetail",
        "targetPath": "src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts",
        "targetLayer": "application",
        "targetTechnology": "react",
        "targetPattern": "viewHook",
        "hints": [
          "Charger les données campagnes via le service CampaignsService",
          "Gérer les états loading/error/empty en sortie de hook"
        ]
      },
      "relations": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "eventUcrs": ["event.user.openCampaignDetail-1"],
        "dataflowUcrs": ["dataflow.campaigns.byId-1"],
        "serviceUcrs": ["service.http.CampaignsService-1"],
        "actionUcrs": ["action.userFlow-CampaignsDetail-open-1"],
        "effectUcrs": [],
        "configNames": ["feature.campaignsDetail.enabled"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Logique principale de la vue de détail campagne."
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

- [ ] `inventory.logic.json` présent et déclaré valide dans `inventories-summary.json`  
- [ ] Les inventaires couplés nécessaires sont disponibles (events, dataflows, services, actions, effects)  
- [ ] `mapping.structure.json` et `project-structure.json` accessibles  
- [ ] Chaque UCR `logic.*` important a une projection dans `mapping.logic.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets (`stackKind`, `targetId`, `targetPath`, `targetLayer`)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.logic` (Stage 35 — Phase 2 : Interprétation)*
