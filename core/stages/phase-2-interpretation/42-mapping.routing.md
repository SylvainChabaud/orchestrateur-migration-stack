# 🧩 Stage 42 — mapping.routing

**Phase :** Phase 2 — Interprétation  
**Précédent :** 41 — mapping.services  
**Suivant :** 43 — mapping.effects  

---

## 🎯 Objectif

Construire le fichier `mapping.routing.json` pour la page `${project.pageName}` en projetant chaque UCR `routing.*` issu de `inventory.routing.json` vers :

- des routes concrètes dans le router de la stack cible ;
- des chemins d’URL avec leurs paramètres ;
- des guards (conditions d’accès, permissions, feature flags) ;
- des liens explicites avec la structure de pages et les événements de navigation.

Ces informations guideront la Phase 3 pour générer / adapter la configuration de routing et les helpers de navigation.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.routing.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.layout.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.routing.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.structure.json`
  - `inventory.layout.json`
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Routing**
  - `${paths.core}/guides-internals/mapping/guide.mapping.routing.md`
  - Fournit :
    - l'objectif du mapping de routing,
    - le schéma JSON contractuel de `mapping.routing.json`,
    - les règles de projection des UCR `routing.*` vers la stack cible,
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

Si `inventory.routing.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.routing.json`

Racine attendue :

```jsonc
{
  "domain": "routing",
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
1.2. Charger `project-structure.json` pour connaître l’organisation des pages, layouts et fichiers de routing (si dédiés).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `routing.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés au routing et au layout.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `routing`

2.1. Charger `inventory.routing.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `routing` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `routing` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.routing.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "routing",
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

4.1. Charger les inventaires `structure`, `layout`, `events`, `conditions`, `hooks`, `logic` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.dataflows.json`, `mapping.async.json`, `mapping.services.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `routing.*` → UCR `structure.*` / `layout.*` ;  
- UCR `routing.*` → UCR `events.*` ;  
- UCR `routing.*` → UCR `hooks.*` / `logic.*` ;  
- UCR `routing.*` → conditions / permissions / flags.

### Étape 5 — Projeter chaque UCR `routing.*`

Pour chaque entrée de `inventory.routing.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `routing.page`, `routing.child`, `routing.redirect`, `routing.modal`, etc.) ;  
- les métadonnées (URL logique, paramètres, criticité, liens).

5.2. Déterminer `toStack.stackKind` et `toStack.routePattern` :  
- `pageRoute`, `nestedRoute`, `redirectRoute`, `modalRoute`, etc.  
- `routePattern` : `"page"`, `"child"`, `"modal"`, `"redirect"`, etc.

5.3. Construire `toStack.routeId`, `toStack.path`, `toStack.targetComponentPath` :  
- `routeId` : identifiant logique (`campaignsDetail`, `campaignsList`, etc.) ;  
- `path` : chemin d’URL (ex. `"/campaigns/:campaignId"`) ;  
- `targetComponentPath` : page / vue cible, dérivée de `project-structure.json`.

5.4. Fixer `toStack.targetLayer` :  
- `"presentation"` pour les routes directement mappées à une page ;  
- `"application"` pour une configuration centralisée du router.

5.5. Renseigner `url.pathParams` / `url.queryParams` / `url.hash` si disponibles.

5.6. Renseigner les guards :  
- `guards.conditionsUcrs` : UCR `conditions.*` requises pour accéder à la route ;  
- `guards.requiredPermissions` / `guards.featureFlags` si ces infos sont disponibles ;  
- `relations.configNames` pour les configs (ex. basePath, flags de routing).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-routing-${item.ucr}`).  
- `fromDsl` : concept `routing.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.routing.json",
    "domain": "routing",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.structureUcrs` / `relations.layoutUcrs` : UCR de structure / layout ;  
- `relations.eventsUcrs` : UCR `events.*` déclenchant la navigation ;  
- `relations.hooksUcrs` : UCR `hooks.*` effectuant la navigation ;  
- `relations.logicUcrs` : logique exécutée sur changement de route ;  
- `metadata.isEntryRoute` / `metadata.isCritical` / `metadata.priority` selon l’importance de la route.

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "routing"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.routing.json` ;  
- chaque `MappingItem` possède `toStack.path`, `toStack.routeId`, `toStack.targetComponentPath`, `toStack.stackKind` renseignés.

6.2. Si un schéma JSON existe pour `mapping.routing.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.routing.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "42",
  "stageName": "mapping.routing",
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

> Continuer avec le **Stage 43 — mapping.actions** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
