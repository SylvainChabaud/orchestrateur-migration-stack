# 🧭 Guide de Mapping — `mapping.async`

*(Projection des concepts `async.*` du DSL vers les mécanismes asynchrones de la stack cible)*

---

## 1. 🎯 Rôle du mapping `async`

Le domaine `async.*` du DSL décrit **tout ce qui se passe dans le temps** :

- timeouts, délais, throttling, debouncing ;
- polling (rafraîchissement régulier des données) ;
- retries (politique de réessai après erreur) ;
- exécution en parallèle / en série ;
- tâches de fond (jobs, workers, batchs) ;
- scénarios asynchrones complexes (enchaînements d’appels).

La **Phase 1 — Analyse** a produit :

- `inventory.async.json` → inventaire des UCR `async.*` ;  
- des liens possibles avec :
  - `dataflows.*` (flux à poller, à rejouer, à synchroniser) ;
  - `services.*` (APIs sujettes aux retries, timeouts, backoff) ;
  - `events.*` (triggers asynchrones) ;
  - `effects.*` (side-effects déclenchés plus tard) ;
  - `logic.*` (orchestration async).

La **Phase 2 — Stage 40 — mapping.async** doit :

> **Projeter chaque UCR `async.*` vers un mécanisme asynchrone concret de la stack cible**, par exemple :
> - configuration de retry / timeout ;
> - hooks de polling ;
> - jobs ou tâches de fond ;
> - wrappers d’appels services (ex. client HTTP avec backoff).

Objectif : rendre explicite **qui est asynchrone, comment, et avec quelles garanties** (retries, délais, parallélisme, etc.).

---

## 2. 📦 Format JSON racine (`mapping.async.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.async.json`

Structure racine attendue :

```json
{
  "domain": "async",
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

- `domain` : `"async"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour l’asynchrone

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `async.*` issu de `inventory.async.json` ;
- à un mécanisme async concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.async.json",
    "domain": "async",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "mode": "string",
    "strategy": "string",
    "hints": []
  },
  "timing": {
    "delayMs": null,
    "intervalMs": null,
    "timeoutMs": null,
    "maxRetries": null,
    "retryBackoff": "string"
  },
  "relations": {
    "dataflowsUcrs": [],
    "servicesUcrs": [],
    "eventsUcrs": [],
    "logicUcrs": [],
    "effectsUcrs": [],
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
  - Identifiant de mapping **unique** dans `mapping.async.json`.  
  - Préfixe recommandé : `map-async-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `async.*` du DSL, par ex. :
    - `async.polling` ;
    - `async.retry` ;
    - `async.timeout` ;
    - `async.parallel` ;
    - `async.sequence` ;
    - `async.backgroundJob`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.async.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"async"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire async.

- `toStack.stackKind`  
  - Type de mécanisme async dans la stack cible, par ex. :
    - `"dataHook"` (polling dans un hook de data) ;
    - `"serviceWrapper"` (HTTP client avec retry/timeout) ;
    - `"job"` (tâche de fond) ;
    - `"schedulerEntry"` ;
    - `"effectHandler"` (gestion asynchrone dans un effet).

- `toStack.targetId`  
  - Identifiant du mécanisme, ex. :
    - `useCampaignsPolling`, `wrapWithRetry`, `scheduleSyncJob`, etc.

- `toStack.targetPath`  
  - Chemin relatif du module cible, dérivé de `project-structure.json`, ex. :
    - `src/pages/CampaignsDetail/hooks/useCampaignsPolling.ts` ;  
    - `src/services/http/withRetry.ts` ;  
    - `src/jobs/campaigns/syncCampaignsJob.ts`…

- `toStack.targetLayer`  
  - `"application"` pour une orchestration async liée à une page / use-case ;  
  - `"infrastructure"` pour un client HTTP, un scheduler ou un job partagé.

- `toStack.mode`  
  - `"polling"`, `"retry"`, `"timeout"`, `"debounce"`, `"throttle"`, `"parallel"`, `"sequence"`, `"backgroundJob"`, etc.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"react-query"`, `"swr"`, `"http-client-custom"`, `"worker"`, etc.

- `toStack.targetPattern`  
  - `"queryPolling"`, `"mutationRetry"`, `"schedulerJob"`, `"wrapper"`, etc.

- `toStack.strategy`  
  - Détail de la stratégie async, ex. `"exponentialBackoff"`, `"fixedDelay"`, `"linearBackoff"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Utiliser une stratégie de backoff exponentiel"`, `"Arrêter le polling quand la page est inactive"`, etc.

- `timing.delayMs` / `timing.intervalMs` / `timing.timeoutMs`  
  - Détails temporels si connus (en millisecondes).

- `timing.maxRetries` / `timing.retryBackoff`  
  - Politique de retry (nombre max de tentatives + type de backoff).

- `relations.dataflowsUcrs`  
  - UCR `dataflows.*` concernés par le mécanisme async.

- `relations.servicesUcrs`  
  - UCR `services.*` à sécuriser via retry / timeout.

- `relations.eventsUcrs` / `relations.logicUcrs` / `relations.effectsUcrs`  
  - UCR déclencheurs ou consommateurs.

- `relations.configNames`  
  - Noms de configs (ex. intervalle de polling configurable par tenant).

---

## 4. ⚙️ Entrées requises pour `mapping.async`

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.async.md` (si tu en définis un)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.async.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.events.json`
  - `inventory.logic.json`
  - `inventory.effects.json`

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
- `mapping.services.json` (si déjà généré pour la page)

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur les inventaires DSL, la structure cible, les guides et les mappings existants.

2. **Asynchrone explicite, pas implicite**  
   - Le mapping doit rendre explicite :
     - d’où vient l’asynchronisme (service, hook, job, timer) ;
     - avec quelle stratégie (polling, retry, timeout, etc.) ;
     - sur quels flux / services cela s’applique.

3. **Centralisation des stratégies communes**  
   - Factoriser les stratégies de retry / timeout dans des wrappers ou clients partagés quand c’est pertinent.

4. **Lien fort avec les dataflows et services**  
   - Pour chaque `async.*` important, relier clairement `dataflowsUcrs` et `servicesUcrs`.

5. **Priorisation des éléments sensibles**  
   - `metadata.isCritical = true` pour :
     - les retries sur opérations sensibles ;
     - les jobs critiques (synchronisations majeures) ;
     - les timeouts impactant l’UX ou la fiabilité métier.

---

## 6. Exemple simplifié de `mapping.async.json`

```json
{
  "domain": "async",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-async-polling-CampaignsDetail-status-1",
      "fromDsl": "async.polling",
      "sourceInventoryRef": {
        "file": "inventory.async.json",
        "domain": "async",
        "itemUcr": "async-polling-CampaignsDetail-status-1"
      },
      "toStack": {
        "stackKind": "dataHook",
        "targetId": "useCampaignStatusPolling",
        "targetPath": "src/pages/CampaignsDetail/hooks/useCampaignStatusPolling.ts",
        "targetLayer": "application",
        "targetTechnology": "react-query",
        "targetPattern": "queryPolling",
        "mode": "polling",
        "strategy": "fixedDelay",
        "hints": [
          "Arrêter le polling quand la page est masquée",
          "Augmenter l'intervalle si le statut devient stable"
        ]
      },
      "timing": {
        "delayMs": 0,
        "intervalMs": 5000,
        "timeoutMs": null,
        "maxRetries": null,
        "retryBackoff": null
      },
      "relations": {
        "dataflowsUcrs": ["dataflows.read-CampaignsDetail-status-1"],
        "servicesUcrs": ["service.http.CampaignsService.getStatus-1"],
        "eventsUcrs": ["events.ui.open-CampaignsDetail-main-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "effectsUcrs": [],
        "configNames": ["config.campaigns.statusPollingInterval"]
      },
      "metadata": {
        "isCritical": false,
        "priority": "normal",
        "notes": "Polling de statut campagne, non bloquant."
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

- [ ] `inventory.async.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.dataflows.json` et `mapping.services.json` cohérents avec les `dataflowsUcrs` / `servicesUcrs` référencés  
- [ ] Chaque UCR `async.*` important a une projection dans `mapping.async.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets + `mode` renseigné  
- [ ] Les params de timing sont remplis quand le DSL les fournit  
- [ ] Les éléments critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.async` (Stage 40 — Phase 2 : Interprétation)*
