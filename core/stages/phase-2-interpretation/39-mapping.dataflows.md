# 🧩 Stage 39 — mapping.dataflows

**Phase :** Phase 2 — Interprétation  
**Précédent :** 38 — mapping.events  
**Suivant :** 40 — mapping.async  

---

## 🎯 Objectif

Construire le fichier `mapping.dataflows.json` pour la page `${project.pageName}` en projetant chaque UCR `dataflows.*` issu de `inventory.dataflows.json` vers :

- des appels de services ;
- des hooks de data ;
- des sélecteurs / mutations de store ;
- des adaptateurs entre backend et vue.

Ces informations guideront la Phase 3 pour générer / adapter les flux de données, leurs adaptateurs et les contrats de données côté vue.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.dataflows.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.services.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.actions.json`
  - `inventory.events.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Dataflows**
  - `${paths.core}/guides-internals/mapping/guide.mapping.dataflows.md`
  - Fournit :
    - l'objectif du mapping de flux de données,
    - le schéma JSON contractuel de `mapping.dataflows.json`,
    - les règles de projection des UCR `dataflows.*` vers la stack cible,
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
- `mapping.services.json` (si déjà généré pour la page)

Si `inventory.dataflows.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.dataflows.json`

Racine attendue :

```jsonc
{
  "domain": "dataflows",
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
1.2. Charger `project-structure.json` pour identifier les dossiers `services`, `state`, `hooks`, etc.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `dataflows.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux données, aux services et au state management.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `dataflows`

2.1. Charger `inventory.dataflows.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `dataflows` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `dataflows` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.dataflows.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "dataflows",
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

4.1. Charger les inventaires `services`, `hooks`, `logic`, `actions`, `events` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.services.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `dataflows.*` → UCR `services.*` ;  
- UCR `dataflows.*` → UCR `hooks.*` ;  
- UCR `dataflows.*` → UCR `logic.*` / `actions.*` / `events.*` ;  
- UCR `dataflows.*` → UCR `structure.*` / `layout.*`.

### Étape 5 — Projeter chaque UCR `dataflows.*`

Pour chaque entrée de `inventory.dataflows.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `dataflows.read`, `dataflows.write`, `dataflows.sync`, etc.) ;  
- les métadonnées (direction, source, cible, criticité, liens d’inventaire).

5.2. Déterminer `toStack.stackKind` et `toStack.direction` :  
- `stackKind` : `serviceCall`, `dataHook`, `storeSelector`, `storeMutation`, `cacheSync`, etc. ;  
- `direction` : `"read"`, `"write"`, `"readWrite"`, `"sync"`.

5.3. Fixer `toStack.sourceKind` et `toStack.targetKind` :  
- ex. `service` → `viewModel`, `store` → `viewSelector`, etc.

5.4. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom explicite du flux (`fetchCampaignById`, `updateCampaignBudget`, `selectCampaignsList`, etc.) ;  
- `targetPath` : module de service / hook / sélecteur, dérivé de `project-structure.json`.

5.5. Fixer `toStack.targetLayer` :  
- `"infrastructure"` pour les flux collés aux services / HTTP ;  
- `"application"` pour les flux scénarisés pour la vue ;  
- `"state"` pour les flux de store.

5.6. Optionnel :  
- `toStack.targetTechnology` (react-query, swr, http client, redux, etc.) ;  
- `toStack.targetPattern` (`query`, `mutation`, `selector`, `adapter`, `syncJob`, etc.) ;  
- `toStack.hints[]` (bonnes pratiques, mise en cache, normalisation…).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-dataflows-${item.ucr}`).  
- `fromDsl` : concept `dataflows.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.dataflows.json",
    "domain": "dataflows",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.servicesUcrs` : UCR `services.*` sous-jacent ;  
- `relations.hooksUcrs` : hooks qui consomment / exposent ce flux ;  
- `relations.logicUcrs`, `relations.actionsUcrs`, `relations.eventsUcrs` : logique / actions / events qui déclenchent ou consomment ce flux ;  
- `relations.structureUcrs`, `relations.layoutUcrs` : vues / zones dépendantes ;  
- `relations.configNames` : noms de configs / flags influençant le flux ;  
- `mapping.inputShape` / `mapping.outputShape` : description synthétique de la forme des données ;  
- `mapping.adapterName` : nom d’un adaptateur dédié si nécessaire ;  
- `metadata.isCritical` / `metadata.priority` : marquer les flux critiques (lecture principale, opérations sensibles).

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "dataflows"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.dataflows.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `direction`, `sourceKind`, `targetKind` renseignés.

6.2. Si un schéma JSON existe pour `mapping.dataflows.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.dataflows.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "39",
  "stageName": "mapping.dataflows",
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

> Continuer avec le **Stage 40 — mapping.async** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
