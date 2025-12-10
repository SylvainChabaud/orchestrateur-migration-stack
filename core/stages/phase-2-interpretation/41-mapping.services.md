# 🧩 Stage 41 — mapping.services

**Phase :** Phase 2 — Interprétation  
**Précédent :** 40 — mapping.async  
**Suivant :** 42 — mapping.routing  

---

## 🎯 Objectif

Construire le fichier `mapping.services.json` pour la page `${project.pageName}` en projetant chaque UCR `services.*` issu de `inventory.services.json` vers :

- des services HTTP / clients backend ;
- des services métier (façades) côté front ;
- des gateways ou adaptateurs vers des systèmes tiers.

Ces informations guideront la Phase 3 pour générer / adapter les modules de services, leurs contrats et leur intégration dans les dataflows.

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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.services.md` (ou équivalent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.data.md` (si présent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.services.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels :
  - `inventory.dataflows.json`
  - `inventory.async.json`
  - `inventory.logic.json`
  - `inventory.actions.json`
  - `inventory.events.json`

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Services**
  - `${paths.core}/guides-internals/mapping/guide.mapping.services.md`
  - Fournit :
    - l'objectif du mapping de services,
    - le schéma JSON contractuel de `mapping.services.json`,
    - les règles de projection des UCR `services.*` vers la stack cible,
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

Si `inventory.services.json`, `inventories-summary.json`, `project-structure.json` ou `mapping.structure.json` sont manquants ou invalides → le stage doit conclure en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.services.json`

Racine attendue :

```jsonc
{
  "domain": "services",
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
1.2. Charger `project-structure.json` pour identifier les dossiers `services`, `api`, `clients`, etc.  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte sur la détection des `services.*` (sans relire le Legacy).  
1.4. Charger les guides de stack liés aux services, aux données et au transport.

### Étape 2 — Vérifier la présence et l’état de l’inventaire `services`

2.1. Charger `inventory.services.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `services` pour `${project.pageName}` est présent et déclaré comme valide ou exploitable.

2.3. Si l’inventaire `services` est manquant ou invalide :

- initialiser un `mappingRoot` minimal (cf. racine plus haut) ;  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"` ;  
- écrire le fichier `mapping.services.json` minimal ;  
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "services",
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

4.1. Charger les inventaires `dataflows`, `async`, `logic`, `actions`, `events` si disponibles.  
4.2. Charger `mapping.structure.json` (obligatoire).  
4.3. Charger `mapping.layout.json`, `mapping.styles.json`, `mapping.i18n.json`, `mapping.config.json`, `mapping.logic.json`, `mapping.conditions.json`, `mapping.hooks.json`, `mapping.events.json`, `mapping.dataflows.json`, `mapping.async.json` si disponibles.  
4.4. Construire des index en mémoire pour relier rapidement :  
- UCR `services.*` → UCR `dataflows.*` ;  
- UCR `services.*` → UCR `async.*` ;  
- UCR `services.*` → UCR `logic.*` / `actions.*` / `events.*`.

### Étape 5 — Projeter chaque UCR `services.*`

Pour chaque entrée de `inventory.services.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `services.http`, `services.domain`, `services.gateway`, etc.) ;  
- les métadonnées (endpoint logique, verbe, criticité, liens d’inventaire).

5.2. Déterminer `toStack.stackKind` :  
- `httpService`, `domainService`, `gateway`, `thirdPartyClient`, etc.

5.3. Construire `toStack.targetId` et `toStack.targetPath` :  
- `targetId` : nom stable (`CampaignsService`, `BudgetService`, etc.) ;  
- `targetPath` : module de service dans la structure cible (dossier `services` ou `api`).

5.4. Fixer `toStack.targetLayer` :  
- `"infrastructure"` pour les clients / gateways ;  
- `"application"` pour des services métier front.

5.5. Renseigner `contract` :  
- `endpoint` : forme logique de l’endpoint (ex. `/campaigns/{id}`) ;  
- `method` : `"GET"`, `"POST"`, etc. ou type de call ;  
- `inputShape` / `outputShape` : formes synthétiques des payloads (clés importantes).

5.6. Optionnel :  
- `toStack.targetTechnology`, `toStack.targetPattern`, `toStack.transport`, `toStack.hints[]` ;  
- mapping à des configs globales (base URL, timeouts, headers).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-services-${item.ucr}`).  
- `fromDsl` : concept `services.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.services.json",
    "domain": "services",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.dataflowsUcrs` : UCR `dataflows.*` qui consomment ce service ;  
- `relations.asyncUcrs` : UCR `async.*` décrivant retries / timeouts / polling ;  
- `relations.logicUcrs`, `relations.actionsUcrs`, `relations.eventsUcrs` : logique / use-cases / événements qui invoquent ce service ;  
- `relations.configNames` : noms de configs (base URL, timeouts, etc.) ;  
- `metadata.isCritical` / `metadata.priority` : marquer les services critiques.

5.8. Ajouter le `MappingItem` dans `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "services"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- unicité des `ucr` ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.services.json` ;  
- chaque `MappingItem` possède `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer`, `contract.endpoint`, `contract.method` renseignés.

6.2. Si un schéma JSON existe pour `mapping.services.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.services.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "41",
  "stageName": "mapping.services",
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

> Continuer avec le **Stage 42 — mapping.actions** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
