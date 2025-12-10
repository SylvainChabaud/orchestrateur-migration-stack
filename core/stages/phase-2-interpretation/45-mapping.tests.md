# 🧩 Stage 45 — mapping.tests

**Phase :** Phase 2 — Interprétation  
**Précédent :** 44 — mapping.actions  
**Suivant :** 46 — mapping.summary

---

## 🎯 Objectif

Construire le fichier `mapping.tests.json` pour la page `${project.pageName}` en projetant chaque UCR `tests.*` issu de `inventory.tests.json` vers :

- un fichier / scénario de test concret ;
- avec un niveau (`unit`, `component`, `integration`, `e2e`, `contract`) ;
- et une couverture traçable sur les UCR des autres domaines.

Ces informations guideront la Phase 3 pour générer / adapter la stratégie de tests (fichiers, dossiers, patterns) dans la stack cible.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tests.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.testing-strategy.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.tests.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.structure.json`
  - `inventory.layout.json`
  - `inventory.styles.json`
  - `inventory.i18n.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.async.json`
  - `inventory.routing.json`
  - `inventory.effects.json`
  - `inventory.actions.json`
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.config.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Tests**
  - `${paths.core}/guides-internals/mapping/guide.mapping.tests.md`
  - Fournit :
    - l'objectif du mapping des tests,
    - le schéma JSON contractuel de `mapping.tests.json`,
    - les règles de projection des UCR `tests.*` vers la stack cible,
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
- `mapping.effects.json`
- `mapping.actions.json`

Si `inventory.tests.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.tests.json`

Racine attendue :

```jsonc
{
  "domain": "tests",
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
1.2. Charger `project-structure.json` pour identifier la stratégie de dossiers de tests (`__tests__`, `tests`, `e2e`, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `tests.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux tests / stratégie de testing.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `tests`

2.1. Charger `inventory.tests.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `tests` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `tests` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.tests.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "tests",
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

4.1. Charger les inventaires des autres domaines (structure, layout, styles, i18n, hooks, logic, dataflows, services, async, routing, effects, actions, events, conditions, config) si disponibles.  
4.2. Charger tous les `mapping.*` déjà produits.  
4.3. Construire des index en mémoire pour relier rapidement :  
- UCR `tests.*` → UCR couverts dans les autres domaines.

### Étape 5 — Projeter chaque UCR `tests.*`

Pour chaque entrée de `inventory.tests.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `tests.unit`, `tests.component`, `tests.integration`, `tests.e2e`, etc.) ;  
- les métadonnées (niveau, criticité, cible de test).

5.2. Déterminer `toStack.testLevel`, `toStack.stackKind`, `toStack.targetPattern` :  
- `testLevel` : `"unit"`, `"component"`, `"integration"`, `"e2e"`, `"contract"`… ;  
- `stackKind` : `unitTest`, `componentTest`, `integrationTest`, `e2eTest`, `contractTest` ;  
- `targetPattern` : `"testFile"`, `"specFile"`, `"featureFile"`, etc.

5.3. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : identifiant de la suite / scénario (`CampaignsDetail.view.test`, `campaigns-e2e.spec`, etc.) ;  
- `targetPath` : chemin du fichier de test (dérivé de `project-structure.json`).

5.4. Fixer `toStack.targetLayer` :  
- `"presentation"`, `"application"`, `"infrastructure"`, `"e2e"` selon la nature du test.

5.5. Optionnel :  
- `toStack.targetTechnology` (jest, vitest, cypress, etc.) ;  
- `toStack.hints[]` (pratiques de test recommandées).

5.6. Renseigner la `coverage` :  
- remplir les tableaux `coverage.*Ucrs` avec les UCR principalement couverts par ce test ;  
- ne pas chercher l’exhaustivité parfaite, mais les UCR majeurs.

5.7. Renseigner la `scope` :  
- `scope.focus` : phrase courte décrivant l’objectif du test ;  
- `scope.criticalPath` : `true` si le test couvre un chemin métier critique.

5.8. Renseigner la `metadata` :  
- `metadata.isBlocking` si ce test doit bloquer un pipeline en cas d’échec (smoke / regression clé) ;  
- `metadata.priority` (`low`, `normal`, `high`) ;  
- `metadata.notes` pour les remarques importantes.

5.9. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-tests-${item.ucr}`).  
- `fromDsl` : concept `tests.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.tests.json",
    "domain": "tests",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- tous les champs `toStack`, `coverage`, `scope`, `metadata` renseignés comme décrit ci-dessus.

5.10. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "tests"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.tests.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `testLevel` renseignés.

6.2. Si un schéma JSON existe pour `mapping.tests.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.tests.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "45",
  "stageName": "mapping.tests",
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

> Continuer avec le **Stage 46** (domaine à confirmer dans la liste globale) uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
