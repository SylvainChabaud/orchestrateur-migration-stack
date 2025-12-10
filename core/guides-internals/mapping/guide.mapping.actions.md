# 🧭 Guide de Mapping — `mapping.actions`

*(Projection des concepts `actions.*` du DSL vers les actions / use-cases concrètes de la stack cible)*

---

## 1. 🎯 Rôle du mapping `actions`

Le domaine `actions.*` du DSL décrit **les intentions métier structurées** :

- use-cases utilisateurs (créer, modifier, supprimer, dupliquer, publier, etc.) ;
- commandes métier (valider, approuver, rejeter, archiver, lancer un calcul, etc.) ;
- workflows d’actions (séquence d’appels services + effets) ;
- actions de navigation métier (ouvrir un écran dans un but précis).

La **Phase 1 — Analyse** a produit :

- `inventory.actions.json` → inventaire des UCR `actions.*` ;  
- des liens possibles avec :
  - `events.*` (triggers UI ou système) ;
  - `dataflows.*` et `services.*` (flux et services appelés par l’action) ;
  - `async.*` (retries, parallélisations, jobs) ;
  - `effects.*` (notifications, logs, tracking) ;
  - `logic.*` (orchestration interne de l’action) ;
  - `routing.*` (changement de page après l’action).

La **Phase 2 — Stage 44 — mapping.actions** doit :

> **Projeter chaque UCR `actions.*` vers une action concrète de la stack cible**, par exemple :
> - fonction de use-case ;
> - handler d’action ;
> - action creator ;
> - commande métier front orchestrant plusieurs services / dataflows.

Objectif : rendre explicite **où vit chaque use-case, comment il est appelé, et comment il relie événements, services, dataflows, async, effets, routing, etc.**

---

## 2. 📦 Format JSON racine (`mapping.actions.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.actions.json`

Structure racine attendue :

```json
{
  "domain": "actions",
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

- `domain` : `"actions"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les actions

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `actions.*` issu de `inventory.actions.json` ;
- à une action / use-case concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.actions.json",
    "domain": "actions",
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
  "payload": {
    "inputShape": {},
    "outputShape": {},
    "canBePartial": false
  },
  "relations": {
    "eventsUcrs": [],
    "conditionsUcrs": [],
    "dataflowsUcrs": [],
    "servicesUcrs": [],
    "asyncUcrs": [],
    "effectsUcrs": [],
    "logicUcrs": [],
    "routingUcrs": [],
    "configNames": []
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "isIdempotent": null,
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping **unique** dans `mapping.actions.json`.  
  - Préfixe recommandé : `map-actions-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `actions.*` du DSL, par ex. :
    - `actions.command` ;
    - `actions.useCase` ;
    - `actions.bulk` ;
    - `actions.workflow`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.actions.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"actions"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des actions.

- `toStack.stackKind`  
  - Type d’action côté stack, par ex. :
    - `"useCaseFunction"` ;
    - `"actionHandler"` ;
    - `"actionCreator"` ;
    - `"commandService"`.

- `toStack.targetId`  
  - Nom de l’action / use-case dans le code, ex. :
    - `saveCampaign`, `duplicateCampaign`, `archiveCampaign`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module qui expose l’action, dérivé de `project-structure.json`, ex. :
    - `src/application/campaigns/useCases/saveCampaign.ts` ;
    - `src/pages/CampaignsDetail/actions/useCampaignsActions.ts`…

- `toStack.targetLayer`  
  - `"application"` dans la plupart des cas (use-cases front) ;  
  - éventuellement `"domain"` si tu modélises un vrai domaine côté front.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"redux"`, `"zustand"`, `"vuex"`, `"custom"`, etc. si l’action est liée à un store / framework.

- `toStack.targetPattern`  
  - `"useCaseFunction"`, `"actionCreator"`, `"thunk"`, `"saga"`, `"listener"`, etc.

- `toStack.hints[]`  
  - Conseils pour l’implémentation : `"Ne pas mettre de logique UI ici"`, `"Déléguer les effets à mapping.effects"`, etc.

- `payload.inputShape` / `payload.outputShape`  
  - Représentation synthétique des entrées et sorties (clés importantes).

- `payload.canBePartial`  
  - `true` si l’action accepte des payloads partiels.

- `relations.eventsUcrs`  
  - UCR `events.*` qui déclenchent cette action (clic bouton, submit, shortcuts, events système…).

- `relations.conditionsUcrs`  
  - UCR `conditions.*` qui doivent être vérifiées avant d’exécuter l’action.

- `relations.dataflowsUcrs` / `relations.servicesUcrs` / `relations.asyncUcrs`  
  - UCR de flux / services / comportements async orchestrés par cette action.

- `relations.effectsUcrs`  
  - UCR `effects.*` déclenchés à l’issue de l’action (toasts, logs, tracking).

- `relations.logicUcrs`  
  - UCR `logic.*` liés à la séquence interne de l’action (préparation, validation, post-traitement).

- `relations.routingUcrs`  
  - UCR `routing.*` impliqués (ex. redirection après succès).

- `relations.configNames`  
  - Clés de configuration (flags qui activent / désactivent l’action, limites, modes).

- `metadata.isCritical`  
  - `true` si l’action est métier critique (sauvegarde, paiement, validation, etc.).

- `metadata.isIdempotent`  
  - `true` si l’action peut être rejouée sans risque (utile pour `async` et `retry`).

---

## 4. ⚙️ Entrées requises pour `mapping.actions`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.actions.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.application-layer.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.actions.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.async.json`
  - `inventory.effects.json`
  - `inventory.logic.json`
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
- `mapping.events.json`
- `mapping.dataflows.json`
- `mapping.async.json`
- `mapping.services.json`
- `mapping.routing.json`
- `mapping.effects.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur :
     - les inventaires DSL ;
     - la structure cible ;
     - les guides de stack ;
     - les mappings existants.

2. **Actions = use-cases, pas logique UI**  
   - Ne pas mélanger :
     - logique de rendu (qui reste dans les vues / hooks de vue) ;
     - avec les use-cases métier (actions).

3. **Cohérence avec les dataflows / services**  
   - Une action non triviale orchestre en général un ou plusieurs `dataflows.*` / `services.*`.  
   - `mapping.actions` doit pointer vers ces flux / services.

4. **Ancrage clair dans les événements et effets**  
   - Chaque action devrait :
     - être déclenchée par au moins un `events.*` ;
     - éventuellement déclencher des `effects.*` (success, error, tracking).

5. **Prise en compte de l’asynchrone**  
   - L’action peut s’appuyer sur `async.*` pour les retries / parallélisation / jobs.  
   - Indiquer l’idempotence (`metadata.isIdempotent`) si connu.

6. **Priorisation des actions critiques**  
   - `metadata.isCritical = true` pour :
     - sauvegarde ;
     - suppression ;
     - opérations financières ;
     - actions ayant un impact important en base ou sur l’utilisateur.

---

## 6. Exemple simplifié de `mapping.actions.json`

```json
{
  "domain": "actions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-actions-saveCampaign-1",
      "fromDsl": "actions.useCase",
      "sourceInventoryRef": {
        "file": "inventory.actions.json",
        "domain": "actions",
        "itemUcr": "actions-useCase-CampaignsDetail-saveCampaign-1"
      },
      "toStack": {
        "stackKind": "useCaseFunction",
        "targetId": "saveCampaign",
        "targetPath": "src/application/campaigns/useCases/saveCampaign.ts",
        "targetLayer": "application",
        "targetTechnology": "custom",
        "targetPattern": "useCaseFunction",
        "hints": [
          "Ne pas mettre de logique UI dans ce use-case",
          "Déléguer les toasts à mapping.effects"
        ]
      },
      "payload": {
        "inputShape": {
          "campaign": {
            "id": "string | null",
            "name": "string",
            "status": "string",
            "budget": "number"
          }
        },
        "outputShape": {
          "campaign": {
            "id": "string",
            "name": "string",
            "status": "string",
            "budget": "number"
          }
        },
        "canBePartial": false
      },
      "relations": {
        "eventsUcrs": ["events.ui.submit-CampaignsDetail-form-1"],
        "conditionsUcrs": ["conditions-canEditCampaign-1"],
        "dataflowsUcrs": ["dataflows.write-CampaignsDetail-update-1"],
        "servicesUcrs": ["service.http.CampaignsService.update-1"],
        "asyncUcrs": ["async.retry-CampaignsDetail-update-1"],
        "effectsUcrs": ["effects.toast-CampaignsDetail-saveSuccess-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "routingUcrs": ["routing-page-CampaignsList-main-1"],
        "configNames": ["config.actions.saveCampaign.enabled"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "isIdempotent": false,
        "notes": "Use-case principal de sauvegarde campagne."
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

- [ ] `inventory.actions.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.events.json`, `mapping.dataflows.json`, `mapping.services.json`, `mapping.async.json`, `mapping.effects.json` cohérents avec les `relations.*Ucrs` référencés  
- [ ] Chaque UCR `actions.*` important a une projection dans `mapping.actions.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets et un `payload` raisonnablement décrit  
- [ ] Les actions critiques sont marquées (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.actions` (Stage 44 — Phase 2 : Interprétation)*
