# 🧩 Stage 35 — mapping.logic

**Phase :** Phase 2 — Interprétation  
**Précédent :** 34 — mapping.config  
**Suivant :** 36 — mapping.conditions 

---

## 🎯 Objectif

Construire le fichier `mapping.logic.json` pour la page `${project.pageName}` en projetant chaque UCR `logic.*` issu de `inventory.logic.json` vers :

- des hooks de vue ;
- des hooks de formulaire ;
- des stores / slices d’état ;
- des contrôleurs de page / de section ;
- des façades applicatives vers des services.

La logique ainsi mappée devra être exploitable directement par la Phase 3 pour générer le code de logique applicative et les tests associés.

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

- `inventory.logic.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels (pour enrichir les relations) :
  - `inventory.events.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.actions.json`
  - `inventory.effects.json`

### 4. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`

### 5. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Logic**
  - `${paths.core}/guides-internals/mapping/guide.mapping.logic.md`
  - Fournit :
    - l'objectif du mapping de logique,
    - le schéma JSON contractuel de `mapping.logic.json`,
    - les règles de projection des UCR `logic.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

Si `inventory.logic.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.logic.json`

Racine attendue :

```jsonc
{
  "domain": "logic",
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
1.2. Charger `project-structure.json` pour connaître l’organisation des dossiers (pages, hooks, stores, services, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte de détection des concepts `logic.*` (sans réanalyse du Legacy).  
1.4. Charger les guides de stack pertinents (architecture, logique, state management).

### Étape 2 — Vérifier la présence et l’état de l’inventaire logique

2.1. Charger `inventory.logic.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `logic` pour `${project.pageName}` est :  
- présent ;
- déclaré comme valide (ou au moins exploitable).

2.3. Si l’inventaire `logic` est manquant ou marqué comme non exploitable :

- initialiser un `mapping.logic` minimal en mémoire ;
- ajouter une issue explicite dans `validation.issues` (ex. `"Missing or invalid inventory.logic.json"`) ;
- fixer `validation.status = "rejected"` ;
- écrire tout de même le fichier `mapping.logic.json` minimal ;
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "logic",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

### Étape 4 — Charger les inventaires couplés et mappings précédents

4.1. Charger les inventaires : `events`, `dataflows`, `services`, `actions`, `effects` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json` si disponibles.  
4.4. Construire des tables d’index (en mémoire) pour relier rapidement :  
- UCR `logic.*` → UCR events / dataflows / services / actions / effects ;
- UCR `logic.*` → UCR structure (vue / section / composant) ;
- UCR `logic.*` → UCR layout (zone).

### Étape 5 — Projeter chaque UCR `logic.*`

Pour chaque entrée de `inventory.logic.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `logic.viewLifecycle`, `logic.formValidation`, `logic.derivedState`, etc.) ;  
- les métadonnées utiles (scope, criticité, liens vers d’autres domaines…).

5.2. Décider du `toStack.stackKind` en fonction des guides de stack et du concept DSL :  
- `viewHook` pour la logique de vue ;  
- `formHook` pour la logique d’un formulaire ;  
- `storeSlice` pour un état partagé ;  
- `controller` pour une orchestration plus large ;  
- `serviceFacade` si la logique encapsule un ensemble d’appels services.

5.3. Construire `toStack.targetId` :  
- nom clair, stable, aligné avec les conventions de la stack ;  
- ex. `useCampaignsDetail`, `useCampaignForm`, `useCampaignFiltersStore`, `CampaignsDetailController`…

5.4. Déduire `toStack.targetPath` depuis `project-structure.json` :  
- pour les hooks : dossier `hooks` si prévu ;  
- pour les stores : dossier `stores` ou `state` ;  
- pour les contrôleurs : dossier `controllers` ou équivalent.

5.5. Fixer `toStack.targetLayer` :  
- `"application"` pour les orchestrations et états partagés ;  
- `"presentation"` pour des logiques locales simples.

5.6. Optionnel :  
- `toStack.targetTechnology` (React hooks, Vue composables, Redux, Zustand…) ;  
- `toStack.targetPattern` (`viewHook`, `formHook`, `storeSlice`, etc.) ;  
- `toStack.hints[]` (bonnes pratiques, patterns recommandés).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-logic-${item.ucr}`).  
- `fromDsl` : concept `logic.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.logic.json",
    "domain": "logic",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.structureUcrs` : UCR de structure concernés (vue/section/composant) ;  
- `relations.layoutUcrs` : UCR de layout associés ;  
- `relations.eventUcrs`, `relations.dataflowUcrs`, `relations.serviceUcrs`, `relations.actionUcrs`, `relations.effectUcrs` : alimentés à partir des index si disponibles ;  
- `relations.configNames` : noms de features flags ou configs utilisés par cette logique ;  
- `metadata.isCritical = true` et `metadata.priority = "high"` pour les logiques centrales (chargement principal de page, flux métier clés, validations bloquantes).

5.8. Ajouter le `MappingItem` à `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Contrôles :  
- `mappingRoot.domain === "logic"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité de tous les `ucr` dans `mappingRoot.items[]` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.logic.json` ;  
- chaque `MappingItem` a un `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` renseignés.

6.2. Si un schéma JSON formel existe pour `mapping.logic.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas d’erreur bloquante :  
- ajouter une description claire dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.logic.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier du workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "35",
  "stageName": "mapping.logic",
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

> Continuer avec le prochain stage de mapping (par ex. `36-mapping-events`) uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
