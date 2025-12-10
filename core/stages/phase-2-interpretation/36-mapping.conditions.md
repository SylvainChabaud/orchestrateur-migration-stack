# 🧩 Stage 36 — mapping.conditions

**Phase :** Phase 2 — Interprétation  
**Précédent :** 35 — mapping.logic  
**Suivant :** 37 — mapping.hooks  

---

## 🎯 Objectif

Construire le fichier `mapping.conditions.json` pour la page `${project.pageName}` en projetant chaque UCR `conditions.*` issu de `inventory.conditions.json` vers :

- des fonctions de prédicat ;
- des helpers de conditions ;
- des guards de routing ;
- des règles réutilisables de type « rule engine léger ».  

Ces conditions seront ensuite exploitées par la Phase 3 pour générer le code des conditions partagées et les intégrer dans les hooks, contrôleurs, routes, actions, etc.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.config.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tenants.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.permissions.md` (ou équivalent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.conditions.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels pour enrichir les liens :
  - `inventory.logic.json`
  - `inventory.config.json`
  - `inventory.routing.json`
  - `inventory.actions.json`
  - `inventory.services.json`
  - `inventory.effects.json`

### 4. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`

### 5. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Conditions**
  - `${paths.core}/guides-internals/mapping/guide.mapping.conditions.md`
  - Fournit :
    - l'objectif du mapping de conditions,
    - le schéma JSON contractuel de `mapping.conditions.json`,
    - les règles de projection des UCR `conditions.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

Si `inventory.conditions.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit se terminer en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.conditions.json`

Racine attendue :

```jsonc
{
  "domain": "conditions",
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
1.2. Charger `project-structure.json` pour connaître l’organisation des modules de conditions / routing / logique.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `conditions.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux conditions, à la logique et à la configuration.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `conditions`

2.1. Charger `inventory.conditions.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `conditions` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `conditions` est manquant ou invalide :

- initialiser un `mapping.conditions` minimal en mémoire (cf. racine plus haut) ;  
- ajouter une issue explicite dans `validation.issues` ;  
- fixer `validation.status = "rejected"` ;  
- écrire malgré tout le fichier `mapping.conditions.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "conditions",
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

4.1. Charger les inventaires optionnels (logic, config, routing, actions, services, effects) si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `conditions.*` → UCR `logic.*` ;  
- UCR `conditions.*` → UCR `config.*` ;  
- UCR `conditions.*` → UCR `structure.*` / `layout.*` ;  
- UCR `conditions.*` → UCR `routing.*` / `actions.*` / `services.*` / `effects.*`.

### Étape 5 — Projeter chaque UCR `conditions.*`

Pour chaque entrée de `inventory.conditions.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `conditions.visibility`, `conditions.enabled`, `conditions.routeGuard`, `conditions.canExecuteAction`, etc.) ;  
- les métadonnées (tenant, environnement, permissions, criticité, contexte d’usage…).

5.2. Décider de `toStack.stackKind` en fonction des guides de stack :  
- `predicateFunction` pour une condition réutilisable (`() => boolean`) ;  
- `conditionsHelper` pour un module regroupant plusieurs prédicats ;  
- `routeGuard` pour des conditions de navigation ;  
- `ruleEntry` si l’on modélise des règles dans un registre dédié.

5.3. Construire `toStack.targetId` :  
- nom explicite, stable, ex. `canViewCampaignDetails`, `isCampaignEditable`, `canUseAdvancedBudget`, etc.

5.4. Déduire `toStack.targetPath` depuis `project-structure.json` :  
- dossier `conditions` prévu pour la page ou le domaine ;  
- ou module de guards de routing si la condition concerne des routes.

5.5. Fixer `toStack.targetLayer` :  
- `"application"` pour les conditions métier majeures ;  
- `"presentation"` pour les conditions purement UI (ex. `hasErrorToDisplay`).

5.6. Optionnel :  
- `toStack.targetTechnology` (typescript, javascript, etc.) ;  
- `toStack.targetPattern` (`predicateFunction`, `conditionsModule`, `routeGuard`, etc.) ;  
- `toStack.hints[]` (notes sur la factorisation, la mutualisation multi-tenant, etc.).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-conditions-${item.ucr}`).  
- `fromDsl` : concept `conditions.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.conditions.json",
    "domain": "conditions",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.logicUcrs` : UCR de logique qui évaluent cette condition ;  
- `relations.configUcrs` : UCR de config utilisés dans la condition (flags, params) ;  
- `relations.structureUcrs` / `relations.layoutUcrs` : cibles UI impactées ;  
- `relations.routingUcrs`, `relations.actionUcrs`, `relations.serviceUcrs`, `relations.effectUcrs` : si la condition intervient dans ces domaines ;  
- `relations.tenants` / `relations.environments` : lorsque la condition varie selon tenant ou environnement ;  
- `relations.permissionKeys` : clés de permissions nécessaires ;  
- `metadata.isCritical` : mettre à `true` pour les conditions de sécurité, d’accès, de droits, etc.  
- `metadata.priority` : `"high"` pour les conditions qui bloquent un flow majeur.

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "conditions"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité de tous les `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.conditions.json` ;  
- chaque `MappingItem` a un `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` renseignés.

6.2. Si un schéma JSON formel existe pour `mapping.conditions.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas d’erreur bloquante :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.conditions.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier du workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "36",
  "stageName": "mapping.conditions",
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

> Continuer avec le **Stage 37 — mapping.hooks** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
