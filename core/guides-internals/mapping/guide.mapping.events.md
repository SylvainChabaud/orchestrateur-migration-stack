# 🧭 Guide de Mapping — `mapping.events`

*(Projection des concepts `events.*` du DSL vers le système d’événements de la stack cible)*

---

## 1. 🎯 Rôle du mapping `events`

Le domaine `events.*` du DSL décrit **tout ce qui se produit dans le système**, par exemple :

- événements UI : clic, focus, blur, changement de valeur, submit, etc. ;
- événements métier : création / modification / suppression d’une entité, changement de statut, etc. ;
- événements de navigation : changement de route, ouverture/fermeture de modal, etc. ;
- événements techniques : succès / erreur d’un appel service, timeout, retry, etc.

La **Phase 1 — Analyse** a produit :

- `inventory.events.json` → inventaire des UCR `events.*` ;  
- des liens possibles avec :
  - `hooks.*` (points d’entrée qui écoutent ou déclenchent ces events) ;
  - `logic.*` (handlers) ;
  - `actions.*` (use-cases déclenchés) ;
  - `effects.*` (side-effects déclenchés) ;
  - `routing.*` (navigation liée à certains événements).

La **Phase 2 — Stage 38 — mapping.events** doit :

> **Projeter chaque UCR `events.*` vers un artefact événementiel concret de la stack cible**, par exemple :
> - callback de composant ;
> - handler dans un hook ;
> - handler dans un contrôleur ;
> - event bus / channel / topic ;
> - mapping vers des événements de librairies (router, form lib, store, etc.).

Objectif : rendre explicite **qui écoute quoi, où, et avec quelles conséquences**.

---

## 2. 📦 Format JSON racine (`mapping.events.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.events.json`

Structure racine attendue :

```json
{
  "domain": "events",
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

- `domain` : `"events"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les événements

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `events.*` issu de `inventory.events.json` ;
- à un handler / callback / channel concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.events.json",
    "domain": "events",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "eventSource": "string",
    "eventName": "string",
    "hints": []
  },
  "relations": {
    "hooksUcrs": [],
    "logicUcrs": [],
    "conditionsUcrs": [],
    "actionsUcrs": [],
    "effectsUcrs": [],
    "dataflowUcrs": [],
    "structureUcrs": [],
    "layoutUcrs": [],
    "routingUcrs": []
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
  - Identifiant de mapping **unique** dans `mapping.events.json`.  
  - Préfixe recommandé : `map-events-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `events.*` du DSL, par ex. :
    - `events.ui.click`, `events.ui.change`, `events.ui.submit` ;
    - `events.domain.statusChanged` ;
    - `events.routing.navigated`, etc.

- `sourceInventoryRef.file`  
  - Toujours `"inventory.events.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"events"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des events.

- `toStack.stackKind`  
  - Type d’artefact côté stack, par ex. :
    - `"componentCallback"` ;
    - `"hookHandler"` ;
    - `"controllerHandler"` ;
    - `"eventBusHandler"` ;
    - `"storeListener"` ;
    - `"routerListener"`.

- `toStack.targetId`  
  - Nom du handler / callback, ex. :
    - `handleSubmit`, `handleClickEdit`, `handleStatusChange`, `onCampaignSaved`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module cible, dérivé de `project-structure.json`, ex. :
    - `src/pages/CampaignsDetail/CampaignsDetail.view.tsx` (callback de composant) ;
    - `src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts` (handler dans un hook) ;
    - `src/pages/CampaignsDetail/controllers/CampaignsDetailController.ts`…

- `toStack.targetLayer`  
  - `"presentation"` pour les callbacks UI simples ;  
  - `"application"` pour les handlers métier / orchestrateurs ;  
  - éventuellement `"infrastructure"` pour un event bus partagé.

- `toStack.eventSource`  
  - Source de l’événement, ex. :
    - `"component"`, `"hook"`, `"router"`, `"store"`, `"service"`, `"formLib"`, etc.

- `toStack.eventName`  
  - Nom d’événement côté stack (ex. `"onClick"`, `"onSubmit"`, `"onChange"`, `"onSuccess"`, `"onError"`, etc.).

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"react"`, `"vue"`, `"redux"`, `"router"`, `"rxjs"`, etc.

- `toStack.targetPattern`  
  - `"componentCallback"`, `"hookHandler"`, `"controllerHandler"`, `"eventBusHandler"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Ne pas mettre de logique métier lourde ici, déléguer au hook"`, etc.

- `relations.hooksUcrs`  
  - UCR `hooks.*` qui déclarent ou consomment ces événements.

- `relations.logicUcrs`  
  - UCR `logic.*` exécutés en réponse à l’événement.

- `relations.conditionsUcrs`  
  - UCR `conditions.*` vérifiées avant d’exécuter les handlers.

- `relations.actionsUcrs` / `relations.effectsUcrs` / `relations.dataflowUcrs`  
  - UCR impactés par cet événement.

- `relations.structureUcrs` / `relations.layoutUcrs`  
  - Composants / zones de layout où l’événement a lieu.

- `relations.routingUcrs`  
  - Routes éventuellement déclenchées suite à l’événement.

---

## 4. ⚙️ Entrées requises pour `mapping.events`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.logic.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.routing.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.events.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.actions.json`
  - `inventory.effects.json`
  - `inventory.routing.json`

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

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur les inventaires DSL, la structure cible, les guides et les mappings existants.

2. **Claire séparation Event → Handler → Logique**  
   - L’événement décrit **ce qui se passe** (ex. `onClick`) ;  
   - Le handler appelle la logique (hook / service / action) ;  
   - La logique est décrite dans `mapping.logic.json` / `mapping.actions.json`.

3. **Localisation cohérente des handlers**  
   - Les callbacks UI simples restent proches des composants ;  
   - Les handlers métier complexes sont dans des hooks / contrôleurs.

4. **Traçabilité complète**  
   - Pour chaque event important, on doit pouvoir remonter à :
     - la vue / section qui le déclenche ;
     - le hook / contrôleur qui le traite ;
     - les actions / effects / dataflows impactés.

5. **Priorisation des events critiques**  
   - `metadata.isCritical = true` pour :
     - les events de soumission de formulaire ;
     - les events de changement de statut métier ;
     - les events liés à la sécurité / aux droits.

---

## 6. Exemple simplifié de `mapping.events.json`

```json
{
  "domain": "events",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-events-ui-submit-CampaignsDetail-main-1",
      "fromDsl": "events.ui.submit",
      "sourceInventoryRef": {
        "file": "inventory.events.json",
        "domain": "events",
        "itemUcr": "events-ui-submit-CampaignsDetail-main-1"
      },
      "toStack": {
        "stackKind": "componentCallback",
        "targetId": "handleSubmit",
        "targetPath": "src/pages/CampaignsDetail/CampaignsDetail.view.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "react",
        "targetPattern": "componentCallback",
        "eventSource": "component",
        "eventName": "onSubmit",
        "hints": [
          "Déléguer la logique métier au hook useCampaignForm"
        ]
      },
      "relations": {
        "hooksUcrs": ["hooks-form-CampaignsDetail-main-1"],
        "logicUcrs": ["logic.formValidation-CampaignsDetail-1"],
        "conditionsUcrs": ["conditions-canSubmit-CampaignsDetail-main-1"],
        "actionsUcrs": ["action.saveCampaign-1"],
        "effectsUcrs": ["effect.showSuccessToast-1"],
        "dataflowUcrs": ["dataflow.campaignForm.payload-1"],
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "routingUcrs": ["routing.CampaignsDetail.afterSubmit-1"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Soumission principale du formulaire de campagne."
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

- [ ] `inventory.events.json` est présent et déclaré valide dans `inventories-summary.json`  
- [ ] `project-structure.json` est accessible  
- [ ] `mapping.hooks.json` et `mapping.logic.json` sont accessibles  
- [ ] Chaque UCR `events.*` important a une projection dans `mapping.events.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets (`stackKind`, `targetId`, `targetPath`, `targetLayer`, `eventSource`, `eventName`)  
- [ ] Les events critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.events` (Stage 38 — Phase 2 : Interprétation)*
