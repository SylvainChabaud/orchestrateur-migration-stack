# 🧩 Stage 43 — mapping.effects

**Phase :** Phase 2 — Interprétation  
**Précédent :** 42 — mapping.routing  
**Suivant :** 44 — mapping.actions  

---

## 🎯 Objectif

Construire le fichier `mapping.effects.json` pour la page `${project.pageName}` en projetant chaque UCR `effects.*` issu de `inventory.effects.json` vers :

- des notifications / toasts / modales ;
- des logs / tracking / instrumentation ;
- des effets liés au stockage ou à l’environnement.

Ces informations guideront la Phase 3 pour générer / adapter les appels d’effets de bord, en cohérence avec les événements, actions, dataflows, services et routing.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.effects.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.observability.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

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

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Effects**
  - `${paths.core}/guides-internals/mapping/guide.mapping.effects.md`
  - Fournit :
    - l'objectif du mapping des effets,
    - le schéma JSON contractuel de `mapping.effects.json`,
    - les règles de projection des UCR `effects.*` vers la stack cible,
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
- `mapping.events.json`
- `mapping.dataflows.json`
- `mapping.async.json`
- `mapping.services.json`
- `mapping.routing.json`

Si `inventory.effects.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.effects.json`

Racine attendue :

```jsonc
{
  "domain": "effects",
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
1.2. Charger `project-structure.json` pour identifier les emplacements possibles des modules d’effets (notifications, tracking, logs, storage…).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `effects.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux effets et à l’observabilité.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `effects`

2.1. Charger `inventory.effects.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `effects` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `effects` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.effects.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "effects",
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

4.1. Charger les inventaires `events`, `actions`, `dataflows`, `services`, `async`, `logic`, `routing` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.dataflows.json`, `mapping.async.json`, `mapping.services.json`, `mapping.routing.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `effects.*` → UCR `events.*` ;  
- UCR `effects.*` → UCR `actions.*` ;  
- UCR `effects.*` → UCR `dataflows.*` / `services.*` / `async.*` ;  
- UCR `effects.*` → UCR `logic.*` / `routing.*`.

### Étape 5 — Projeter chaque UCR `effects.*`

Pour chaque entrée de `inventory.effects.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `effects.toast`, `effects.log`, `effects.tracking`, `effects.storage`, etc.) ;  
- les métadonnées (type d’effet, criticité, liens d’inventaire).

5.2. Déterminer `toStack.effectType` et `toStack.stackKind` :  
- `effectType` : `"toast"`, `"modal"`, `"log"`, `"tracking"`, `"storage"`, `"telemetry"`, `"custom"`, etc. ;  
- `stackKind` : `notification`, `logging`, `tracking`, `storage`, `telemetry`, `customEffect`…

5.3. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom explicite (`notifyCampaignSaved`, `trackCampaignCreated`, `logError`, `storageClient`, etc.) ;  
- `targetPath` : module d’effet (hook, service, client) dérivé de `project-structure.json`.

5.4. Fixer `toStack.targetLayer` :  
- `"presentation"` pour les effets visuels (toasts, modales) ;  
- `"application"` pour les effets métier (tracking de use-cases) ;  
- `"infrastructure"` pour les effets système (logs techniques, storage, télémetrie).

5.5. Optionnel :  
- `toStack.targetTechnology` (librairie de notifications, tracking, logging…) ;  
- `toStack.targetPattern` (`hook`, `serviceModule`, `utilityFunction`, `client`, etc.) ;  
- `toStack.hints[]` (bonnes pratiques, confidentialité, non-blocage de l’UI…).

5.6. Renseigner `payload` :  
- `payload.shape` : clés principales des données envoyées à l’effet ;  
- `payload.sensitiveKeys` : clés sensibles (id utilisateur, email, tokens, etc.).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-effects-${item.ucr}`).  
- `fromDsl` : concept `effects.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.effects.json",
    "domain": "effects",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.eventsUcrs` : `events.*` déclencheurs ;  
- `relations.actionsUcrs` : `actions.*` se terminant par cet effet ;  
- `relations.dataflowsUcrs`, `relations.servicesUcrs`, `relations.asyncUcrs` : flux, services et comportements async liés ;  
- `relations.logicUcrs` : logique qui décide d’exécuter ou non l’effet ;  
- `relations.routingUcrs` : routes associées au tracking / aux logs de navigation ;  
- `relations.configNames` : configs contrôlant l’activation des effets (ex. tracking activé ou non en dev) ;  
- `metadata.isCritical` / `metadata.priority` : marquer les effets critiques (sécurité, conformité…).

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "effects"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.effects.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `effectType` renseignés.

6.2. Si un schéma JSON existe pour `mapping.effects.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.effects.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "43",
  "stageName": "mapping.effects",
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

> Continuer avec le **Stage 44 — mapping.actions** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
