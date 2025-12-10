# 🧩 Stage 34 — mapping.config

**Phase :** Phase 2 — Interprétation  
**Précédent :** 33 — mapping.i18n  
**Suivant :** 35 — mapping.logic  

---

## 🎯 Objectif

Construire le fichier `mapping.config.json` pour la page `${project.pageName}` en projetant chaque UCR `config.*` issu de `inventory.config.json` vers :

- des entrées de feature flags ;
- des entrées de configuration par tenant ;
- des paramètres par environnement ;
- des blocs de configuration applicative (permissions, seuils, constantes, etc.).

Le résultat doit fournir un plan de configuration exploitable par la Phase 3 pour générer ou structurer les fichiers de configuration concrets de la stack cible.

Aucune relecture du Legacy n’est effectuée dans ce stage.

---

## ⚙️ Entrées requises

> Toutes les entrées sont résolues à partir de `core/configs/project.config.yaml`.  
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
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.config.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tenants.md` (ou équivalent)

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.config.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)

### 4. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Config**
  - `${paths.core}/guides-internals/mapping/guide.mapping.config.md`
  - Fournit :
    - l'objectif du mapping de configuration,
    - le schéma JSON contractuel de `mapping.config.json`,
    - les règles de projection des UCR `config.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

### 5. Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.logic.json` (si existant, pour relier configs ↔ logique)  
- `mapping.structure.json` (pour relier configs ↔ composants / vues)  
- éventuellement `mapping.events.json`, `mapping.actions.json` pour les configs impactant des flows métier

Si `inventory.config.json`, `inventories-summary.json` ou `project-structure.json` sont manquants ou invalides → le stage doit se terminer en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.config.json`

Racine attendue :

```jsonc
{
  "domain": "config",
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

1.1. Charger `core/configs/project.config.yaml` et résoudre les variables `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître l’organisation des dossiers de configuration (`public/configuration`, `src/config`, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte de détection des concepts `config.*`.  
1.4. Charger les guides de stack relatifs à la configuration et aux tenants (s’ils existent).

### Étape 2 — Vérifier la présence et l’état de l’inventaire config

2.1. Charger `inventory.config.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `config` pour `${project.pageName}` est :  
- présent ;
- marqué comme valide ou exploitable.

2.3. Si l’inventaire `config` est manquant ou invalide :

- initialiser un objet `mappingRoot` minimal (voir Étape 3) ;
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;
- fixer `mappingRoot.validation.status = "rejected"` ;
- écrire le fichier `mapping.config.json` minimal ;
- conclure le stage en **Gate ❌**.

### Étape 3 — Initialiser l’objet racine `mappingRoot`

Construire en mémoire :

```jsonc
{
  "domain": "config",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

Nommer cet objet `mappingRoot` et l’utiliser comme buffer jusqu’à l’écriture du fichier.

### Étape 4 — Charger les mappings déjà produits (optionnel mais recommandé)

4.1. Charger `mapping.logic.json` si disponible pour relier les configs à la logique consommatrice.  
4.2. Charger `mapping.structure.json` pour relier les configs aux vues / sections concernées.  
4.3. Facultatif : charger d’autres mappings (`events`, `actions`) si des configs pilotent directement des flows métier.

### Étape 5 — Projeter chaque UCR `config.*`

Pour chaque entrée de `inventory.config.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (ex. `config.featureFlag`, `config.tenantSetting`, `config.envSetting`, etc.) ;  
- les métadonnées (tenant, environnement, portée, sensibilité, valeur par défaut…).

5.2. Déterminer `toStack.stackKind` en fonction du concept DSL et des guides de stack :  
- `featureFlagEntry` pour les flags ;  
- `tenantConfigEntry` pour les configs spécifiques à un tenant ;  
- `environmentConfigEntry` pour les configs variant par environnement ;  
- `permissionsConfigEntry` si la config pilote des droits ou capacités ;  
- `runtimeConfigEntry` pour des paramètres d’exécution.

5.3. Construire `toStack.targetId` :  
- nom canonique de la clé de config, en respectant les conventions (dot notation, prefixes, etc.),  
- ex. `feature.campaignsDetail.enabled`, `tenant.whitelabel.campaigns.maxCount`, `env.preprod.apiBaseUrl`…

5.4. Déduire `toStack.targetPath` à partir de `project-structure.json` :  
- localiser les fichiers de configuration prévus (par ex. `public/configuration`, `src/config`, etc.) ;  
- choisir le bon fichier / module cible selon tenant / environnement / domaine.

5.5. Fixer `toStack.targetLayer` :  
- `"configuration"` dans la majorité des cas ;  
- `"application"` si la config est encapsulée dans un module applicatif spécifique.

5.6. Optionnel :  
- `toStack.targetTechnology` (lib ou pattern de config utilisé) ;  
- `toStack.targetPattern` (`flatConfig`, `nestedConfig`, `byTenantByEnv`, etc.) ;  
- `toStack.hints[]` (consignes : ne pas exposer côté client, restreindre à certains tenants, etc.).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-config-${item.ucr}`).  
- `fromDsl` : concept `config.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.config.json",
    "domain": "config",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.logicUcrs` : UCR de logique consommant la config (si connus).  
- `relations.structureUcrs` : UCR de structure impactés (vue / composant conditionnel).  
- `relations.tenants` : liste des tenants concernés (WHITELABEL, EXITO, etc.).  
- `relations.environments` : environnements concernés (TEST, RECETTE, PREPROD, PROD, etc.).  
- `relations.permissionKeys` : clés de permission pour lesquelles cette config joue un rôle (si pertinent).  
- `metadata.isCritical` / `metadata.priority` :  
  - marquer `isCritical = true` pour les configs de sécurité, de droits, de disponibilité majeure ;  
  - régler `priority` sur `"high"` pour les configs qui impactent le parcours principal utilisateur.

5.8. Ajouter chaque `MappingItem` à `mappingRoot.items[]`.

### Étape 6 — Validation interne

6.1. Contrôles basiques :  
- `mappingRoot.domain === "config"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- tous les `ucr` dans `mappingRoot.items[]` sont uniques ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.config.json` ;  
- chaque `MappingItem` a un `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` renseignés.

6.2. Si un schéma JSON formel existe pour `mapping.config.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue descriptive dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.config.json`

7.2. Créer les répertoires manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier que `mapping.config.json` dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "34",
  "stageName": "mapping.config",
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

> Continuer avec le **Stage 35 — mapping.logic** uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
