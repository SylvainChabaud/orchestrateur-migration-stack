# 🧩 Stage 40 — mapping.async

**Phase :** Phase 2 — Interprétation  
**Précédent :** 39 — mapping.dataflows  
**Suivant :** 41 — mapping.services  

---

## 🎯 Objectif

Construire le fichier `mapping.async.json` pour la page `${project.pageName}` en projetant chaque UCR `async.*` issu de `inventory.async.json` vers :

- des hooks / wrappers asynchrones (polling, retry, timeout, debounce, throttle) ;
- des jobs / tâches de fond ;
- des mécanismes d’exécution parallèle / séquentielle.

Ces informations guideront la Phase 3 pour générer / adapter les comportements asynchrones (polling, retries, jobs, etc.) en cohérence avec les dataflows et services.

Aucune relecture du Legacy n’est autorisée dans ce stage.

---

## ⚙️ Entrées requises

> Toutes les entrées sont dérivées de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration

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

### 2. Artefacts Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.data.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.async.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.async.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels pour enrichir :
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.events.json`
  - `inventory.logic.json`
  - `inventory.effects.json`

### 4. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

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

### 5. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Async**
  - `${paths.core}/guides-internals/mapping/guide.mapping.async.md`
  - Fournit :
    - l'objectif du mapping asynchrone,
    - le schéma JSON contractuel de `mapping.async.json`,
    - les règles de projection des UCR `async.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

Si `inventory.async.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.async.json`

Racine attendue :

```jsonc
{
  "domain": "async",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

---

## 🧠 Actions (logique du stage)

### Étape 1 — Charger configuration et contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre les `${paths.*}`.  
1.2. Charger `project-structure.json` pour identifier les modules potentiels : hooks de data, clients HTTP, jobs, etc.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `async.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux données, aux services, au state management et à l’asynchrone.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `async`

2.1. Charger `inventory.async.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `async` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `async` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.async.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "async",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

Nommer cet objet `mappingRoot`.

### Étape 4 — Charger les inventaires couplés et mappings précédents

4.1. Charger les inventaires `dataflows`, `services`, `events`, `logic`, `effects` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.dataflows.json`, `mapping.services.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `async.*` → UCR `dataflows.*` ;  
- UCR `async.*` → UCR `services.*` ;  
- UCR `async.*` → UCR `events.*` / `logic.*` / `effects.*` ;  
- UCR `async.*` → UCR `structure.*` / `layout.*` si la vue dépend d’un comportement async (ex. skeletons, loaders).

### Étape 5 — Projeter chaque UCR `async.*`

Pour chaque entrée de `inventory.async.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `async.polling`, `async.retry`, `async.timeout`, `async.parallel`, `async.sequence`, etc.) ;  
- les métadonnées (détails de timing, stratégie, criticité, liens d’inventaire).

5.2. Déterminer `toStack.mode` et `toStack.stackKind` :  
- `mode` : `"polling"`, `"retry"`, `"timeout"`, `"debounce"`, `"throttle"`, `"parallel"`, `"sequence"`, `"backgroundJob"`, etc. ;  
- `stackKind` : `dataHook`, `serviceWrapper`, `job`, `schedulerEntry`, `effectHandler`, etc.

5.3. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom explicite (`useCampaignStatusPolling`, `wrapWithRetry`, `syncCampaignsJob`, etc.) ;  
- `targetPath` : chemin du hook / wrapper / job, dérivé de `project-structure.json`.

5.4. Fixer `toStack.targetLayer` :  
- `"infrastructure"` pour les wrappers clients / jobs partagés ;  
- `"application"` pour les hooks / orchestrateurs liés à une page / use-case.

5.5. Optionnel :  
- `toStack.targetTechnology` (react-query, worker, http client, etc.) ;  
- `toStack.targetPattern` (`queryPolling`, `mutationRetry`, `schedulerJob`, etc.) ;  
- `toStack.strategy` (`exponentialBackoff`, `fixedDelay`, etc.) ;  
- `toStack.hints[]` (bonnes pratiques async).

5.6. Renseigner `timing` :  
- `delayMs`, `intervalMs`, `timeoutMs` selon ce que fournit le DSL ;  
- `maxRetries`, `retryBackoff` pour les stratégies de retry.

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-async-${item.ucr}`).  
- `fromDsl` : concept `async.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.async.json",
    "domain": "async",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.dataflowsUcrs` : UCR `dataflows.*` concernés ;  
- `relations.servicesUcrs` : UCR `services.*` associés ;  
- `relations.eventsUcrs`, `relations.logicUcrs`, `relations.effectsUcrs` : éléments déclencheurs / impactés ;  
- `relations.configNames` : noms de configs impactant timing ou stratégie ;  
- `metadata.isCritical` / `metadata.priority` : marquer les comportements async critiques (opérations sensibles, jobs importants).

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "async"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.async.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `mode` renseignés.

6.2. Si un schéma JSON existe pour `mapping.async.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.async.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "40",
  "stageName": "mapping.async",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

- `inputsAvailable` = `false` si une entrée obligatoire est manquante.  
- `schemaValidated` = `false` si la validation JSON n’a pas été effectuée ou a échoué.  
- `outputsWritten` = `false` si le fichier n’a pas pu être écrit.

---

## 🧩 Gate

Fin de fichier, écrire **exactement l’un** des blocs :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` en cas de problème bloquant (input manquant, inventaire invalide, schéma non respecté, sortie non écrite, etc.).

---

## 📦 Stage suivant

> Continuer avec le **Stage 41 — mapping.services** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
