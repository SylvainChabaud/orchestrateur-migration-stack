# 🧩 Stage 38 — mapping.events

**Phase :** Phase 2 — Interprétation  
**Précédent :** 37 — mapping.hooks  
**Suivant :** 39 — mapping.dataflows  

---

## 🎯 Objectif

Construire le fichier `mapping.events.json` pour la page `${project.pageName}` en projetant chaque UCR `events.*` issu de `inventory.events.json` vers :

- des callbacks de composants ;
- des handlers dans des hooks ;
- des handlers dans des contrôleurs ;
- des listeners d’event bus / store / router.

Ces informations guideront la Phase 3 pour générer / adapter les handlers d’événements et câbler les interactions entre vue, logique et dataflows.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.logic.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.routing.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.events.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels pour enrichir :
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.actions.json`
  - `inventory.effects.json`
  - `inventory.routing.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Events**
  - `${paths.core}/guides-internals/mapping/guide.mapping.events.md`
  - Fournit :
    - l'objectif du mapping d'événements,
    - le schéma JSON contractuel de `mapping.events.json`,
    - les règles de projection des UCR `events.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

### 5. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`
- `mapping.hooks.json`

Si `inventory.events.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.events.json`

Racine attendue :

```jsonc
{
  "domain": "events",
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

1.1. Charger `core/configs/project.config.yaml` et résoudre les interpolations `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître l’organisation des vues, hooks, contrôleurs, event bus, etc.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `events.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés à la logique, au routing et au state management.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `events`

2.1. Charger `inventory.events.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `events` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `events` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.events.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "events",
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

4.1. Charger les inventaires `hooks`, `logic`, `conditions`, `dataflows`, `actions`, `effects`, `routing` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `events.*` → UCR `hooks.*` ;  
- UCR `events.*` → UCR `logic.*` / `actions.*` / `effects.*` ;  
- UCR `events.*` → UCR `dataflows.*` ;  
- UCR `events.*` → UCR `structure.*` / `layout.*` / `routing.*`.

### Étape 5 — Projeter chaque UCR `events.*`

Pour chaque entrée de `inventory.events.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `events.ui.click`, `events.ui.change`, `events.ui.submit`, `events.domain.statusChanged`, etc.) ;  
- les métadonnées (type d’événement, source, criticité, contexte, liens d’inventaire).

5.2. Déterminer `toStack.stackKind` selon le type d’événement et les guides de stack :  
- `componentCallback` pour les events UI directement câblés dans un composant ;  
- `hookHandler` pour les events gérés dans un hook ;  
- `controllerHandler` pour les orchestrations plus globales ;  
- `eventBusHandler` ou `storeListener` pour les évènements partagés ;  
- `routerListener` pour les changements de route.

5.3. Fixer `toStack.eventSource` et `toStack.eventName` :  
- `eventSource` : `"component"`, `"hook"`, `"router"`, `"service"`, `"store"`, `"formLib"`, etc. ;  
- `eventName` : nom d’événement concret (ex. `"onClick"`, `"onChange"`, `"onSubmit"`, `"onSuccess"`, `"onError"`, `"onRouteChange"`, etc.).

5.4. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom explicite du handler (ex. `handleSubmit`, `handleClickEdit`, `onCampaignSaved`, etc.) ;  
- `targetPath` : module de la vue / du hook / du contrôleur, dérivé de `project-structure.json`.

5.5. Fixer `toStack.targetLayer` :  
- `"presentation"` pour les callbacks purement UI ;  
- `"application"` pour les handlers qui déclenchent du métier ;  
- `"infrastructure"` pour les listeners d’event bus / router / store partagés.

5.6. Optionnel :  
- `toStack.targetTechnology`, `toStack.targetPattern`, `toStack.hints[]` selon la stack.

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-events-${item.ucr}`).  
- `fromDsl` : concept `events.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.events.json",
    "domain": "events",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.hooksUcrs` : UCR de hooks associés ;  
- `relations.logicUcrs` : UCR de logique exécutée ;  
- `relations.conditionsUcrs` : conditions préalables ;  
- `relations.actionsUcrs`, `relations.effectsUcrs`, `relations.dataflowUcrs` : éléments affectés ;  
- `relations.structureUcrs`, `relations.layoutUcrs` : composants / zones de layout concernés ;  
- `relations.routingUcrs` : routes déclenchées ou impactées ;  
- `metadata.isCritical` / `metadata.priority` : marquer les events clés (soumissions, changements d’état, etc.).

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "events"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.events.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `eventSource`, `eventName` renseignés.

6.2. Si un schéma JSON existe pour `mapping.events.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.events.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "38",
  "stageName": "mapping.events",
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

Le fichier de stage doit se terminer par **exactement l’un** des blocs :

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

> Continuer avec le **Stage 39 — mapping.dataflows** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
