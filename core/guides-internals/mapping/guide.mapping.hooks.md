# 🧭 Guide de Mapping — `mapping.hooks`

*(Projection des concepts `hooks.*` du DSL vers les hooks / composables de la stack cible)*

---

## 1. 🎯 Rôle du mapping `hooks`

Le domaine `hooks.*` du DSL décrit les **points d’ancrage de la logique de vue** :

- hooks de page (chargement, orchestration globale, lifecycle) ;
- hooks de formulaire (gestion des valeurs, validation, soumission) ;
- hooks de section / widget (états locaux, interactions spécifiques) ;
- hooks de data (data fetching, synchronisation, polling) ;
- hooks techniques (connexion à un store, à un routeur, à un contexte, etc.).

La **Phase 1 — Analyse** a produit :

- `inventory.hooks.json` → inventaire des UCR `hooks.*` ;  
- des liens éventuels avec :
  - `logic.*` (logique principale implémentée via ces hooks) ;
  - `conditions.*` (conditions d’activation, de visibilité) ;
  - `dataflows.*` (données consommées / exposées par les hooks) ;
  - `services.*` (APIs appelées depuis les hooks) ;
  - `routing.*` (navigation pilotée par les hooks).

La **Phase 2 — Stage 37 — mapping.hooks** doit :

> **Projeter chaque UCR `hooks.*` vers un hook concret de la stack cible**  
> (React hook, Vue composable, Svelte store + helper, etc.) en s’appuyant sur :
>
> - les guides de stack (patterns de hooks / composables) ;
> - la structure cible (`project-structure.json`) ;
> - les mappings `mapping.logic.json`, `mapping.conditions.json`, `mapping.dataflows.json`, etc.

L’objectif est de répondre à la question :  
**“Quel hook sera généré ou utilisé, où, et pour quelle logique ?”**

---

## 2. 📦 Format JSON racine (`mapping.hooks.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.hooks.json`

Structure racine attendue :

```json
{
  "domain": "hooks",
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

- `domain` : `"hooks"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les hooks

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `hooks.*` issu de `inventory.hooks.json` ;
- à un hook / composable concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.hooks.json",
    "domain": "hooks",
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
    "conditionsUcrs": [],
    "dataflowUcrs": [],
    "serviceUcrs": [],
    "routingUcrs": [],
    "structureUcrs": [],
    "layoutUcrs": [],
    "configNames": []
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
  - Identifiant de mapping **unique** dans `mapping.hooks.json`.  
  - Préfixe recommandé : `map-hooks-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `hooks.*` du DSL, par ex. :
    - `hooks.view` ;
    - `hooks.form` ;
    - `hooks.section` ;
    - `hooks.data` ;
    - `hooks.routing`, etc.

- `sourceInventoryRef.file`  
  - Toujours `"inventory.hooks.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"hooks"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des hooks.

- `toStack.stackKind`  
  - Type de hook dans la stack cible, par ex. :
    - `"viewHook"` ;
    - `"formHook"` ;
    - `"sectionHook"` ;
    - `"dataHook"` ;
    - `"routingHook"` ;
    - `"customHook"`.

- `toStack.targetId`  
  - Nom du hook / composable, par ex. :  
    - `useCampaignsDetail` ;  
    - `useCampaignForm` ;  
    - `useCampaignFilters` ;  
    - `useCampaignsRouting`…

- `toStack.targetPath`  
  - Chemin relatif dans la stack cible, dérivé de `project-structure.json`, par ex. :  
    - `src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts` ;  
    - `src/pages/CampaignsDetail/hooks/useCampaignForm.ts`…

- `toStack.targetLayer`  
  - Couche ciblée, typiquement `"application"` (hook principal) ou `"presentation"` (hook local UI).

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"react"`, `"vue"`, `"svelte"`, `"solid"`, etc.

- `toStack.targetPattern`  
  - `"viewHook"`, `"formHook"`, `"sectionHook"`, `"dataHook"`, `"routingHook"`, etc.

- `toStack.hints[]`  
  - Conseils : `"Expose un contrat simple pour la vue"`, `"Garder la logique de validation dans ce hook"`, etc.

- `relations.logicUcrs`  
  - UCR `logic.*` encapsulés dans ce hook.

- `relations.conditionsUcrs`  
  - UCR `conditions.*` évaluées par ce hook.

- `relations.dataflowUcrs`  
  - UCR `dataflows.*` consommés / fournis par ce hook.

- `relations.serviceUcrs`  
  - UCR `services.*` appelés depuis ce hook.

- `relations.routingUcrs`  
  - UCR `routing.*` manipulés (navigation, redirections).

- `relations.structureUcrs` / `relations.layoutUcrs`  
  - Zones de vue / composants qui utilisent ce hook.

- `relations.configNames`  
  - Noms de configs / flags impactant ce hook (si connus).

---

## 4. ⚙️ Entrées requises pour `mapping.hooks`

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

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.logic.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.state-management.md` (ou équivalent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.hooks.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels pour enrichir :
  - `inventory.logic.json`
  - `inventory.conditions.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.routing.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décision se basent sur les inventaires, les guides, la structure cible et les mappings existants.

2. **Séparation nette entre hook et logique brute**  
   - Le hook est un **point d’entrée** pour la vue ;  
   - La logique brute (`logic.*`) peut être encapsulée dans des services / helpers, mais exposée via le hook.

3. **Alignement fort avec la structure du projet**  
   - Les `targetPath` doivent respecter les conventions de la stack (dossier `hooks` par page, dossier `composables`, etc.).

4. **Traçabilité complète**  
   - Les relations (`logicUcrs`, `conditionsUcrs`, `dataflowUcrs`, etc.) doivent permettre à la Phase 3 :
     - de savoir quoi injecter dans le hook ;
     - quels services appeler ;
     - quelles conditions évaluer.

5. **Priorisation des hooks critiques**  
   - `metadata.isCritical = true` pour les hooks de page principale, de formulaire métier clé, ou ceux qui orchestrent un flow majeur.

---

## 6. Exemple simplifié de `mapping.hooks.json`

```json
{
  "domain": "hooks",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-hooks-view-CampaignsDetail-main-1",
      "fromDsl": "hooks.view",
      "sourceInventoryRef": {
        "file": "inventory.hooks.json",
        "domain": "hooks",
        "itemUcr": "hooks-view-CampaignsDetail-main-1"
      },
      "toStack": {
        "stackKind": "viewHook",
        "targetId": "useCampaignsDetail",
        "targetPath": "src/pages/CampaignsDetail/hooks/useCampaignsDetail.ts",
        "targetLayer": "application",
        "targetTechnology": "react",
        "targetPattern": "viewHook",
        "hints": [
          "Exposer loading/error/data + helpers d'actions",
          "Intégrer les conditions d'accès et de modification"
        ]
      },
      "relations": {
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "conditionsUcrs": [
          "conditions-visibility-CampaignsDetail-main-1",
          "conditions-enabled-CampaignsDetail-edit-1"
        ],
        "dataflowUcrs": ["dataflow.campaigns.byId-1"],
        "serviceUcrs": ["service.http.CampaignsService-1"],
        "routingUcrs": ["routing.CampaignsDetail.main-1"],
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "configNames": ["feature.campaignsDetail.enabled"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Hook principal de la page détail campagne."
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

- [ ] `inventory.hooks.json` présent et déclaré valide dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] `mapping.logic.json` et `mapping.conditions.json` accessibles  
- [ ] Chaque UCR `hooks.*` important a une projection dans `mapping.hooks.json`  
- [ ] Tous les `MappingItem` ont des `toStack.*` complets (`stackKind`, `targetId`, `targetPath`, `targetLayer`)  
- [ ] Les hooks critiques sont marqués (`metadata.isCritical = true` si besoin)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.hooks` (Stage 37 — Phase 2 : Interprétation)*
