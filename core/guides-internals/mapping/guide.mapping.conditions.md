# 🧭 Guide de Mapping — `mapping.conditions`

*(Projection des concepts `conditions.*` du DSL vers les systèmes de conditions / prédicats de la stack cible)*

---

## 1. 🎯 Rôle du mapping `conditions`

Le domaine `conditions.*` du DSL décrit toutes les **conditions logiques** qui contrôlent :

- la visibilité d’un composant / d’une section ;
- l’activation d’un bouton, d’un CTA, d’un champ ;
- le passage d’une étape à une autre dans un flow ;
- l’exécution d’une action, d’un effet, d’un appel service ;
- des comportements dépendants du contexte (tenant, environnement, permissions, config, données…).

La **Phase 1 — Analyse** a produit :

- `inventory.conditions.json` → inventaire des UCR `conditions.*` ;
- des liens éventuels avec :
  - `logic.*` (logique qui évalue ou combine ces conditions) ;
  - `config.*` (flags / paramètres utilisés dans les conditions) ;
  - `routing.*` (conditions de navigation / garde de routes) ;
  - `structure.*` / `layout.*` (affichage conditionnel).

La **Phase 2 — Stage 36 — mapping.conditions** doit :

> **Projeter chaque UCR `conditions.*` vers un artefact de conditions dans la stack cible**,  
> par exemple :
> - fonctions de prédicat ;
> - helpers de conditions ;
> - guards de routing ;
> - règles réutilisables (rule engine léger) ;
> - contrats de conditions attachés à des hooks / contrôleurs.

L’objectif est d’obtenir une cartographie claire des **conditions réutilisables**, avec un emplacement précis dans la stack cible et des liens forts avec la logique, la config et la structure.

---

## 2. 📦 Format JSON racine (`mapping.conditions.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.conditions.json`

Structure racine attendue :

```json
{
  "domain": "conditions",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs :

- `domain` : `"conditions"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les conditions

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `conditions.*` issu de `inventory.conditions.json` ;
- à un artefact de conditions dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.conditions.json",
    "domain": "conditions",
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
    "configUcrs": [],
    "structureUcrs": [],
    "layoutUcrs": [],
    "routingUcrs": [],
    "actionUcrs": [],
    "serviceUcrs": [],
    "effectUcrs": [],
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
  - Identifiant de mapping **unique** dans `mapping.conditions.json`.  
  - Préfixe recommandé : `map-conditions-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `conditions.*` du DSL, par ex. :
    - `conditions.visibility` ;
    - `conditions.enabled` ;
    - `conditions.transitionAllowed` ;
    - `conditions.routeGuard` ;
    - `conditions.canExecuteAction`, etc.

- `sourceInventoryRef.file`  
  - Toujours `"inventory.conditions.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"conditions"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des conditions.

- `toStack.stackKind`  
  - Type d’artefact de conditions dans la stack cible, par ex. :
    - `"predicateFunction"` (fonction `() => boolean`) ;
    - `"conditionsHelper"` (ensemble de prédicats) ;
    - `"routeGuard"` ;
    - `"ruleEntry"` (si on modélise une mini rule-engine).

- `toStack.targetId`  
  - Identifiant de la condition dans la stack cible, ex. :
    - `canViewCampaignDetails` ;
    - `isCampaignEditable` ;
    - `canAccessAdvancedBudget` ;
    - `canNavigateToStepReview`…

- `toStack.targetPath`  
  - Chemin relatif du module de conditions, dérivé de `project-structure.json` :  
    - ex. `src/pages/CampaignsDetail/conditions/canViewCampaignDetails.ts` ;  
    - ou `src/conditions/campaigns/canEditCampaign.ts`…

- `toStack.targetLayer`  
  - Couche ciblée, typiquement `"application"` (conditions métier) ou `"presentation"` (conditions purement UI).

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Par ex. `"typescript"`, `"javascript"`, `"vue"`, etc.

- `toStack.targetPattern`  
  - Par ex. `"predicateFunction"`, `"conditionsModule"`, `"routeGuard"`, `"ruleEntry"`…

- `toStack.hints[]`  
  - Conseils : `"Centraliser cette condition pour tous les écrans campagnes"`, `"Prévoir une version par tenant"`, etc.

- `relations.logicUcrs`  
  - UCR `logic.*` qui évaluent ou consomment cette condition.

- `relations.configUcrs`  
  - UCR `config.*` utilisés dans la condition (flags, paramètres).

- `relations.structureUcrs` / `relations.layoutUcrs`  
  - UCR des parties d’UI affectées (composants / zones cachés ou désactivés).

- `relations.routingUcrs`  
  - UCR `routing.*` si la condition participe à une garde de route.

- `relations.actionUcrs` / `relations.serviceUcrs` / `relations.effectUcrs`  
  - UCR impactés si la condition bloque ou permet certaines actions / effets / appels services.

- `relations.tenants` / `relations.environments`  
  - Tenants / environnements pour lesquels la condition varie.

- `relations.permissionKeys`  
  - Clés de permissions / rôles nécessaires.

---

## 4. ⚙️ Entrées requises pour `mapping.conditions`

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

- Guides de stack (logique / conditions / tenants / permissions) :  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.logic.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.config.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tenants.md` (ou équivalent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.permissions.md` (ou équivalent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.conditions.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- optionnel pour enrichir les liens :
  - `inventory.logic.json`
  - `inventory.config.json`
  - `inventory.routing.json`
  - `inventory.actions.json`
  - `inventory.services.json`
  - `inventory.effects.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions se basent uniquement sur :
     - les inventaires DSL ;
     - les guides de stack ;
     - la structure cible ;
     - les mappings existants.

2. **Conditions centralisées et réutilisables**  
   - Éviter les conditions dupliquées dans plusieurs hooks / composants ;  
   - Privilégier des modules de conditions partagés (surtout si plusieurs pages les consomment).

3. **Articulation avec la config**  
   - Quand une condition utilise un flag / paramètre, le lier clairement via `relations.configUcrs` ;  
   - Ex : `canViewCampaignDetails` dépend de `feature.campaignsDetail.enabled` et de permissions.

4. **Respect de `project-structure.json`**  
   - Les `targetPath` doivent être compatibles avec l’arborescence réelle (dossier `conditions`, dossier `routing/guards`, etc.).

5. **Priorisation des conditions critiques**  
   - Marquer `metadata.isCritical = true` pour :
     - les conditions de sécurité ;
     - les conditions bloquant l’accès à une section sensible ;
     - les conditions légales / réglementaires.

---

## 6. Exemple simplifié de `mapping.conditions.json`

```json
{
  "domain": "conditions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-conditions-canViewCampaignDetails-1",
      "fromDsl": "conditions.visibility",
      "sourceInventoryRef": {
        "file": "inventory.conditions.json",
        "domain": "conditions",
        "itemUcr": "conditions-visibility-CampaignsDetail-main-1"
      },
      "toStack": {
        "stackKind": "predicateFunction",
        "targetId": "canViewCampaignDetails",
        "targetPath": "src/pages/CampaignsDetail/conditions/canViewCampaignDetails.ts",
        "targetLayer": "application",
        "targetTechnology": "typescript",
        "targetPattern": "predicateFunction",
        "hints": [
          "Vérifier la permission perm.campaigns.view",
          "Tenir compte du feature flag feature.campaignsDetail.enabled"
        ]
      },
      "relations": {
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "configUcrs": ["config-featureFlag-campaignsDetailEnabled-1"],
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "routingUcrs": [],
        "actionUcrs": ["action.userFlow-CampaignsDetail-open-1"],
        "serviceUcrs": [],
        "effectUcrs": [],
        "tenants": ["WHITELABEL", "EXITO"],
        "environments": ["TEST", "RECETTE", "PREPROD", "PROD"],
        "permissionKeys": ["perm.campaigns.view"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Condition principale pour afficher la page détail campagne."
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

- [ ] `inventory.conditions.json` présent et déclaré valide dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.logic.json` et `mapping.config.json` accessibles (fortement recommandé)  
- [ ] Chaque UCR `conditions.*` important a une projection dans `mapping.conditions.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets (`stackKind`, `targetId`, `targetPath`, `targetLayer`)  
- [ ] Les conditions critiques sont marquées (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.conditions` (Stage 36 — Phase 2 : Interprétation)*
