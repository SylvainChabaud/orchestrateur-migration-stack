# 🧭 Guide de Mapping — `mapping.effects`

*(Projection des concepts `effects.*` du DSL vers les effets de bord de la stack cible)*

---

## 1. 🎯 Rôle du mapping `effects`

Le domaine `effects.*` du DSL décrit **tout ce qui est un effet de bord** par rapport au coeur métier :

- affichage de notifications, toasts, modales d’alerte ;
- logs fonctionnels / techniques ;
- tracking (analytics, événements de monitoring) ;
- interactions avec le navigateur (localStorage, sessionStorage, window, document, etc.) ;
- télémetrie, métriques, instrumentation ;
- tout effet non purement calculatoire.

La **Phase 1 — Analyse** a produit :

- `inventory.effects.json` → inventaire des UCR `effects.*` ;  
- des liens possibles avec :
  - `events.*` (triggers qui déclenchent les effets) ;
  - `actions.*` (use-cases qui se terminent par un effet) ;
  - `dataflows.*` / `services.*` (effets déclenchés après succès / erreur) ;
  - `logic.*` (orchestration qui décide si l’on déclenche un effet ou non).

La **Phase 2 — Stage 43 — mapping.effects** doit :

> **Projeter chaque UCR `effects.*` vers un effet concret de la stack cible**, par exemple :
> - utilitaire de notifications ;
> - système de logs / tracking ;
> - wrapper de stockage local ;
> - module d’instrumentation.

Objectif : rendre explicite **où vivent les effets de bord, comment ils sont appelés, et avec quelles données**.

---

## 2. 📦 Format JSON racine (`mapping.effects.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.effects.json`

Structure racine attendue :

```json
{
  "domain": "effects",
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

- `domain` : `"effects"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les effets

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `effects.*` issu de `inventory.effects.json` ;
- à un effet concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.effects.json",
    "domain": "effects",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "effectType": "string",
    "hints": []
  },
  "payload": {
    "shape": {},
    "sensitiveKeys": []
  },
  "relations": {
    "eventsUcrs": [],
    "actionsUcrs": [],
    "dataflowsUcrs": [],
    "servicesUcrs": [],
    "asyncUcrs": [],
    "logicUcrs": [],
    "routingUcrs": [],
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
  - Identifiant de mapping **unique** dans `mapping.effects.json`.  
  - Préfixe recommandé : `map-effects-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `effects.*` du DSL, par ex. :
    - `effects.notification` ;
    - `effects.toast` ;
    - `effects.log` ;
    - `effects.tracking` ;
    - `effects.storage`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.effects.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"effects"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des effets.

- `toStack.stackKind`  
  - Type d’effet côté stack, par ex. :
    - `"notification"` ;
    - `"logging"` ;
    - `"tracking"` ;
    - `"storage"` ;
    - `"telemetry"` ;
    - `"customEffect"`.

- `toStack.targetId`  
  - Nom de l’API / utilitaire / hook / service d’effet, ex. :
    - `notifySuccess`, `notifyError`, `logEvent`, `trackCampaignSaved`, `storageClient`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module de l’effet, dérivé de `project-structure.json`, ex. :
    - `src/ui/notifications/useNotifications.ts` ;
    - `src/instrumentation/tracking.ts` ;
    - `src/infrastructure/storage/storageClient.ts`…

- `toStack.targetLayer`  
  - `"presentation"` pour les effets purement UI (toasts, modales) ;  
  - `"application"` pour les effets d’orchestration (tracking métier) ;  
  - `"infrastructure"` pour les effets liés à l’environnement (storage, logs techniques).

- `toStack.effectType`  
  - `"toast"`, `"modal"`, `"log"`, `"tracking"`, `"storage"`, `"telemetry"`, `"custom"`…

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"custom"`, `"sentry"`, `"datadog"`, `"segment"`, `"localStorage-api"`, etc.

- `toStack.targetPattern`  
  - `"hook"`, `"serviceModule"`, `"utilityFunction"`, `"client"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Ne pas bloquer l'UX avec cet effet"`, `"Masquer les données sensibles dans les logs"`, etc.

- `payload.shape`  
  - Description synthétique des données envoyées à l’effet (clés importantes).

- `payload.sensitiveKeys`  
  - Clés à traiter avec prudence (RGPD, secrets, etc.).

- `relations.eventsUcrs`  
  - UCR `events.*` déclenchant l’effet.

- `relations.actionsUcrs`  
  - UCR `actions.*` qui terminent sur cet effet.

- `relations.dataflowsUcrs` / `relations.servicesUcrs` / `relations.asyncUcrs`  
  - UCR de flux / services / comportements async liés à cet effet (succès, erreurs, retries).

- `relations.logicUcrs`  
  - UCR `logic.*` qui décident de déclencher ou non l’effet.

- `relations.routingUcrs`  
  - Routes impliquant un effet (ex. tracking de navigation).

- `relations.configNames`  
  - Clés de configuration : activation / désactivation de tracking, niveaux de logs, etc.

---

## 4. ⚙️ Entrées requises pour `mapping.effects`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.effects.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.observability.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.effects.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.events.json`
  - `inventory.actions.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.async.json`
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

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur les inventaires, la structure cible, les guides et les mappings existants.

2. **Effets de bord clairement séparés du métier**  
   - Le coeur métier ne doit pas être couplé directement aux APIs de notifications / logs / tracking ;
   - `mapping.effects` doit rendre explicite où se trouvent ces appels.

3. **Prise en compte de la confidentialité et de la sécurité**  
   - Identifier les `payload.sensitiveKeys` ;
   - Prévoir des hints sur l’anonymisation / pseudonymisation.

4. **Traçabilité des triggers**  
   - Lien clair entre :
     - l’événement (souvent `events.*`) ;
     - l’action / use-case (`actions.*`) ;
     - les flux / services concernés ;
     - l’effet déclenché.

5. **Priorisation des effets critiques**  
   - `metadata.isCritical = true` pour :
     - les logs de sécurité ;
     - les notifications bloquantes ;
     - les effets liés à la conformité (audit, RGPD, etc.).

---

## 6. Exemple simplifié de `mapping.effects.json`

```json
{
  "domain": "effects",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-effects-toast-CampaignsDetail-saveSuccess-1",
      "fromDsl": "effects.toast",
      "sourceInventoryRef": {
        "file": "inventory.effects.json",
        "domain": "effects",
        "itemUcr": "effects-toast-CampaignsDetail-saveSuccess-1"
      },
      "toStack": {
        "stackKind": "notification",
        "targetId": "notifyCampaignSaved",
        "targetPath": "src/ui/notifications/useCampaignNotifications.ts",
        "targetLayer": "presentation",
        "targetTechnology": "custom",
        "targetPattern": "hook",
        "effectType": "toast",
        "hints": [
          "Utiliser un toast non bloquant",
          "Prévoir un message i18n spécifique pour la campagne"
        ]
      },
      "payload": {
        "shape": {
          "messageKey": "campaign.saved.success",
          "campaignId": "string"
        },
        "sensitiveKeys": []
      },
      "relations": {
        "eventsUcrs": ["events.ui.submit-CampaignsDetail-main-1"],
        "actionsUcrs": ["action.saveCampaign-1"],
        "dataflowsUcrs": ["dataflows.write-CampaignsDetail-update-1"],
        "servicesUcrs": ["service.http.CampaignsService.update-1"],
        "asyncUcrs": ["async.retry-CampaignsDetail-update-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "routingUcrs": [],
        "configNames": ["config.ui.notifications.enabled"]
      },
      "metadata": {
        "isCritical": false,
        "priority": "normal",
        "notes": "Toast de succès standard après sauvegarde campagne."
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

- [ ] `inventory.effects.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.events.json`, `mapping.actions.json`, `mapping.dataflows.json`, `mapping.services.json`, `mapping.async.json` cohérents avec les `relations.*Ucrs` référencés  
- [ ] Chaque UCR `effects.*` important a une projection dans `mapping.effects.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets + `effectType` renseigné  
- [ ] Les effets critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] Les `payload.sensitiveKeys` sont renseignées dès que nécessaire  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.effects` (Stage 43 — Phase 2 : Interprétation)*
