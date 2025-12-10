# 🧭 Guide de Mapping — `mapping.config`

*(Projection des concepts `config.*` du DSL vers le système de configuration / feature flags / tenants de la stack cible)*

---

## 1. 🎯 Rôle du mapping `config`

Le domaine `config.*` du DSL décrit tout ce qui relève de la **configuration applicative** :

- feature flags (activation / désactivation de fonctionnalités) ;
- paramètres par environnement (DEV / TEST / RECETTE / PREPROD / PROD) ;
- paramètres par tenant (GPA, MKP, 3WR, ITM, WHITELABEL, EXITO, etc.) ;
- constantes fonctionnelles (plafonds, seuils, limites) ;
- toggles liés aux permissions ou contextes d’exécution.

La **Phase 1 — Analyse** a produit :

- `inventory.config.json` → inventaire des UCR `config.*` (flags, paramètres, constantes, configs multi-tenants, etc.) ;
- des liens éventuels avec :
  - `logic.*` (logique conditionnée par des flags / configs) ;
  - `structure.*` / `layout.*` (affichage conditionnel) ;
  - `permissions.*` si modélisé dans le DSL.

La **Phase 2 — Stage 34 — mapping.config** doit :

> **Projeter chaque UCR `config.*` vers un artefact de configuration de la stack cible**,  
> par exemple :
> - fichiers de configuration (JSON, TS, YAML, .env, etc.) ;
> - modules de feature flags ;
> - fichiers de configuration par tenant ;
> - registres de permissions / capabilities.

L’objectif est de construire un **plan de configuration clair, multi-tenant et multi-environnement**, aligné avec la façon dont la stack cible gère ses configs.

---

## 2. 📦 Format JSON racine (`mapping.config.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.config.json`

Structure racine attendue :

```json
{
  "domain": "config",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs principaux :

- `domain` : `"config"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative uniquement)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping (`"valid"` / `"rejected"` + issues)

---

## 3. 🔗 Schéma d’un `MappingItem` de configuration

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `config.*` issu de `inventory.config.json` ;
- à une ressource de configuration dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.config.json",
    "domain": "config",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "hints": []
  },
  "relations": {
    "logicUcrs": [],
    "structureUcrs": [],
    "tenants": [],
    "environments": [],
    "permissionKeys": []
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping unique dans `mapping.config.json`.  
  - Préfixe recommandé : `map-config-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `config.*` du DSL, par ex. :
    - `config.featureFlag` ;
    - `config.tenantSetting` ;
    - `config.envSetting` ;
    - `config.permissionToggle` ;
    - `config.threshold`, etc.

- `sourceInventoryRef.file`  
  - Toujours `"inventory.config.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"config"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire de configuration.

- `toStack.stackKind`  
  - Type d’artefact configuration dans la stack cible, par ex. :
    - `"featureFlagEntry"` ;
    - `"tenantConfigEntry"` ;
    - `"environmentConfigEntry"` ;
    - `"permissionsConfigEntry"` ;
    - `"runtimeConfigEntry"`.

- `toStack.targetId`  
  - Identifiant final de la config dans la stack : nom de clé de config, de flag ou de bloc de configuration.

- `toStack.targetPath`  
  - Chemin relatif du fichier de configuration cible, dérivé de `project-structure.json`, par ex. :
    - `public/configuration/featureFlags.json` ;
    - `public/configuration/tenants/whitelabel.json` ;
    - `src/config/appConfig.ts` ;
    - ou toute structure définie par la stack.

- `toStack.targetLayer`  
  - Couche ciblée, par ex. `"configuration"` ou `"application"` selon la convention retenue.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Par ex. `"node-config"`, `"next-runtime-config"`, `"custom-json-config"`, etc.

- `toStack.targetPattern`  
  - Par ex. `"flatConfig"`, `"nestedConfig"`, `"byTenantByEnv"`, etc.

- `toStack.hints[]`  
  - Conseils concrets (ex. `"Regrouper ce flag avec les flags de campagnes"`, `"Ne pas exposer cette config au client"`).

- `relations.logicUcrs`  
  - UCR `logic.*` consommateurs de cette config.

- `relations.structureUcrs`  
  - UCR `structure.*` impactés (composants / sections rendues conditionnellement).

- `relations.tenants`  
  - Liste de tenants concernés (ex. `["WHITELABEL", "EXITO"]`).

- `relations.environments`  
  - Liste d’environnements où la config est utilisée ou varie (`"TEST"`, `"RECETTE"`, `"PREPROD"`, `"PROD"`, etc.).

- `relations.permissionKeys`  
  - Clés de permissions couplées (si la config pilote des capacités / droits).

---

## 4. ⚙️ Entrées requises pour `mapping.config`

### 4.1. Configuration (obligatoire)

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

### 4.2. Artefacts Phase 0 (lecture seule)

- Structure cible du projet :  
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- Bridge Legacy → DSL :  
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- Guides de stack (configuration / feature flags / tenants) :  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.config.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tenants.md` (ou équivalent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.config.json` (inventaire primaire)  
- `inventories-summary.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.logic.json` (si déjà présent)  
- `mapping.structure.json` (pour rattacher les configs à des composants / sections)  
- éventuellement `mapping.events.json`, `mapping.actions.json` si la config déclenche / impacte des flows métier

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent sur l’inventaire `config`, les guides de stack, la structure cible et les mappings existants.

2. **Stratégie multi-tenant et multi-environnement explicite**  
   - Les relations `tenants` et `environments` doivent rendre lisible :
     - quelles configs sont spécifiques à un tenant ;
     - quelles configs varient selon l’environnement.

3. **Respect de `project-structure.json`**  
   - Les `targetPath` doivent respecter l’organisation réelle des fichiers de configuration.

4. **Sensibilité de la config**  
   - Les configs sensibles (auth, secrets, endpoints critiques) doivent être marquées (`metadata.isCritical = true`) et, si pertinent, accompagnées d’un hint indiquant qu’elles ne doivent pas être exposées côté client.

5. **Traçabilité vers la logique**  
   - Quand possible, les `logicUcrs` doivent pointer vers les UCR `logic.*` qui consomment la config, pour faciliter la génération de code conditionnel.

---

## 6. Exemple simplifié de `mapping.config.json`

```json
{
  "domain": "config",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-config-feature-campaignsDetailEnabled-1",
      "fromDsl": "config.featureFlag",
      "sourceInventoryRef": {
        "file": "inventory.config.json",
        "domain": "config",
        "itemUcr": "config-featureFlag-campaignsDetailEnabled-1"
      },
      "toStack": {
        "stackKind": "featureFlagEntry",
        "targetId": "feature.campaignsDetail.enabled",
        "targetPath": "public/configuration/featureFlags.json",
        "targetLayer": "configuration",
        "targetTechnology": "custom-json-config",
        "targetPattern": "flatConfig",
        "hints": [
          "Flag partagé par les tenants WHITELABEL et EXITO",
          "Désactivé par défaut en TEST / RECETTE"
        ]
      },
      "relations": {
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "tenants": ["WHITELABEL", "EXITO"],
        "environments": ["TEST", "RECETTE", "PREPROD", "PROD"],
        "permissionKeys": ["perm.campaigns.view"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Contrôle l'accès à la page détail campagne."
      }
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 7. ✅ Checklist de validation

- [ ] `inventory.config.json` est présent et déclaré valide dans `inventories-summary.json`  
- [ ] `project-structure.json` est accessible  
- [ ] Les conventions multi-tenant / multi-env sont respectées selon les guides de stack  
- [ ] Chaque UCR `config.*` important a une projection dans `mapping.config.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets (`stackKind`, `targetId`, `targetPath`, `targetLayer`)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.config` (Stage 34 — Phase 2 : Interprétation)*
