# 🧩 Stage 44 — mapping.actions

**Phase :** Phase 2 — Interprétation  
**Précédent :** 43 — mapping.effects  
**Suivant :** 45 — mappings.tests

---

## 🎯 Objectif

Construire le fichier `mapping.actions.json` pour la page `${project.pageName}` en projetant chaque UCR `actions.*` issu de `inventory.actions.json` vers :

- des use-cases / fonctions d’actions métier ;
- des action handlers / action creators ;
- des commandes front qui orchestrent services, dataflows, async, effets, routing.

Ces informations guideront la Phase 3 pour générer / adapter les modules d’actions et leur intégration propre dans la stack cible.

Aucune relecture du Legacy n’est autorisée dans ce stage.

---

## ⚙️ Entrées requises

> Toutes les entrées sont dérivées de `core/configs.project.config.yaml`.  
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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.actions.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.application-layer.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.actions.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.async.json`
  - `inventory.effects.json`
  - `inventory.logic.json`
  - `inventory.routing.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Actions**
  - `${paths.core}/guides-internals/mapping/guide.mapping.actions.md`
  - Fournit :
    - l'objectif du mapping des actions,
    - le schéma JSON contractuel de `mapping.actions.json`,
    - les règles de projection des UCR `actions.*` vers la stack cible,
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

Si `inventory.actions.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.actions.json`

Racine attendue :

```jsonc
{
  "domain": "actions",
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
1.2. Charger `project-structure.json` pour identifier les emplacements possibles des use-cases / actions (dossier `application`, `useCases`, `actions`, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `actions.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés à la couche applicative et aux actions.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `actions`

2.1. Charger `inventory.actions.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `actions` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `actions` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.actions.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "actions",
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

4.1. Charger les inventaires `events`, `conditions`, `dataflows`, `services`, `async`, `effects`, `logic`, `routing` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.dataflows.json`, `mapping.async.json`, `mapping.services.json`, `mapping.routing.json`, `mapping.effects.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `actions.*` → UCR `events.*` ;  
- UCR `actions.*` → UCR `conditions.*` ;  
- UCR `actions.*` → UCR `dataflows.*` / `services.*` / `async.*` ;  
- UCR `actions.*` → UCR `effects.*` / `logic.*` / `routing.*`.

### Étape 5 — Projeter chaque UCR `actions.*`

Pour chaque entrée de `inventory.actions.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `actions.command`, `actions.useCase`, `actions.workflow`, etc.) ;  
- les métadonnées (criticité, liens d’inventaire, nature de l’action).

5.2. Déterminer `toStack.stackKind` et `toStack.targetPattern` :  
- `stackKind` : `useCaseFunction`, `actionHandler`, `actionCreator`, `commandService`, etc. ;  
- `targetPattern` : `"useCaseFunction"`, `"actionCreator"`, `"thunk"`, `"saga"`, `"listener"`, etc.

5.3. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom d’action (`saveCampaign`, `duplicateCampaign`, etc.) ;  
- `targetPath` : module de use-cases / actions, dérivé de `project-structure.json`.

5.4. Fixer `toStack.targetLayer` :  
- `"application"` pour les use-cases front ;  
- éventuellement `"domain"` si le projet distingue clairement la couche domaine côté front.

5.5. Renseigner `payload` :  
- `inputShape` : structure synthétique des entrées (payload d’action) ;  
- `outputShape` : structure synthétique des sorties (résultat d’action) ;  
- `canBePartial` : `true` si l’action accepte des payloads partiels.

5.6. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-actions-${item.ucr}`).  
- `fromDsl` : concept `actions.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.actions.json",
    "domain": "actions",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.eventsUcrs` : `events.*` déclencheurs ;  
- `relations.conditionsUcrs` : `conditions.*` préalables ;  
- `relations.dataflowsUcrs`, `relations.servicesUcrs`, `relations.asyncUcrs` : flux, services, comportements async orchestrés ;  
- `relations.effectsUcrs` : `effects.*` déclenchés ;  
- `relations.logicUcrs` : logique associée ;  
- `relations.routingUcrs` : routes impliquées (redirections, transitions) ;  
- `relations.configNames` : configs qui activent / désactivent ou modulent l’action ;  
- `metadata.isCritical` / `metadata.priority` / `metadata.isIdempotent` : marquer les caractéristiques importantes de l’action.

5.7. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "actions"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.actions.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` renseignés.

6.2. Si un schéma JSON existe pour `mapping.actions.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.actions.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "44",
  "stageName": "mapping.actions",
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

> Continuer avec le **Stage 45** (domaine à confirmer dans la liste globale) uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
