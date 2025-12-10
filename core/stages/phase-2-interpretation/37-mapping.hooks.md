# 🧩 Stage 37 — mapping.hooks

**Phase :** Phase 2 — Interprétation  
**Précédent :** 36 — mapping.conditions  
**Suivant :** 38 — mapping.events  

---

## 🎯 Objectif

Construire le fichier `mapping.hooks.json` pour la page `${project.pageName}` en projetant chaque UCR `hooks.*` issu de `inventory.hooks.json` vers :

- des hooks de vue ;
- des hooks de formulaire ;
- des hooks de section / widget ;
- des hooks de data ;
- des hooks techniques (routing, store, contexte…).

Ces hooks seront ensuite utilisés par la Phase 3 pour générer le code des hooks eux‑mêmes (ou les adapter) et pour câbler la logique, les conditions et les dataflows.

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

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.hooks.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.logic.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.routing.json`

### 4. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`

### 5. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Hooks**
  - `${paths.core}/guides-internals/mapping/guide.mapping.hooks.md`
  - Fournit :
    - l'objectif du mapping de hooks,
    - le schéma JSON contractuel de `mapping.hooks.json`,
    - les règles de projection des UCR `hooks.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

Si `inventory.hooks.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.hooks.json`

Racine attendue :

```jsonc
{
  "domain": "hooks",
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
1.2. Charger `project-structure.json` pour identifier les dossiers `hooks` / `composables` / équivalents.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `hooks.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés à la logique et au state management.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `hooks`

2.1. Charger `inventory.hooks.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `hooks` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `hooks` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.hooks.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "hooks",
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

4.1. Charger les inventaires `logic`, `conditions`, `dataflows`, `services`, `routing` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `hooks.*` → UCR `logic.*` ;  
- UCR `hooks.*` → UCR `conditions.*` ;  
- UCR `hooks.*` → UCR `dataflows.*` / `services.*` / `routing.*` ;  
- UCR `hooks.*` → UCR `structure.*` / `layout.*`.

### Étape 5 — Projeter chaque UCR `hooks.*`

Pour chaque entrée de `inventory.hooks.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `hooks.view`, `hooks.form`, `hooks.section`, `hooks.data`, etc.) ;  
- les métadonnées (scope, criticité, dépendances listées, etc.).

5.2. Déterminer `toStack.stackKind` en fonction des guides de stack :  
- `viewHook`, `formHook`, `sectionHook`, `dataHook`, `routingHook`, `customHook`…

5.3. Construire `toStack.targetId` :  
- nom de hook aligné sur les conventions (par ex. `useCampaignsDetail`, `useCampaignForm`, `useCampaignFilters`, `useCampaignsRouting`).

5.4. Déduire `toStack.targetPath` depuis `project-structure.json` :  
- dossiers `hooks` par page ou par domaine ;  
- ou structure équivalente (ex. dossier `composables` en Vue).

5.5. Fixer `toStack.targetLayer` :  
- `"application"` pour les hooks principaux de page / formulaire / data ;  
- `"presentation"` pour des hooks locaux moins critiques.

5.6. Optionnel :  
- `toStack.targetTechnology` (React, Vue, etc.) ;  
- `toStack.targetPattern` (`viewHook`, `formHook`, etc.) ;  
- `toStack.hints[]` (bonnes pratiques, patterns).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-hooks-${item.ucr}`).  
- `fromDsl` : concept `hooks.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.hooks.json",
    "domain": "hooks",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.logicUcrs` : UCR de logique encapsulés dans ce hook ;  
- `relations.conditionsUcrs` : UCR de conditions évaluées par ce hook ;  
- `relations.dataflowUcrs` : dataflows consommés / produits ;  
- `relations.serviceUcrs` : services appelés ;  
- `relations.routingUcrs` : routes manipulées (navigation / redirections) ;  
- `relations.structureUcrs` / `relations.layoutUcrs` : composants / zones qui utilisent ce hook ;  
- `relations.configNames` : noms de configs / flags impactant le comportement ;  
- `metadata.isCritical` et `metadata.priority` configurés selon l’importance du hook (page principale, formulaire central, etc.).

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Contrôler :  
- `mappingRoot.domain === "hooks"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité de tous les `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.hooks.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` renseignés.

6.2. Si un schéma JSON existe pour `mapping.hooks.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.hooks.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "37",
  "stageName": "mapping.hooks",
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

> Continuer avec le **Stage 38 — mapping.events** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
