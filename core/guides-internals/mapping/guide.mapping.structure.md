# 🧭 Guide de Mapping — `mapping.structure`

*(Projection des concepts `structure.*` du DSL vers la structure de la stack cible)*

---

## 1. 🎯 Rôle du mapping `structure`

La **Phase 1** a produit l’inventaire canonique `inventory.structure.json`, qui décrit :

- l’arborescence UI de la page (`structure.page`, `structure.viewNode`, `structure.component`, `structure.container`, `structure.section`, etc.),
- les éléments interactifs de base (`structure.button`, `structure.input`, `structure.form`, `structure.list`, `structure.item`, `structure.text`, `structure.layoutZone`),
- pour chaque élément, un **UCR** stable (`view-*`, `component-*`, `form-*`, etc.) et un concept DSL `structure.*`.fileciteturn0file2turn0file6

La **Phase 2**, et en particulier le stage **30 — mapping.structure**, ne lit plus le Legacy.  
Elle doit :

> **Projeter chaque élément `structure.*` inventorié vers un artefact de structure dans la stack cible**,  
> en utilisant :
> - la configuration de stack (`*.stack.yaml`),
> - les guides de stack générés en Phase 0 (structure, UI, routing),
> - la structure cible du projet (`project-structure.json`).fileciteturn0file3turn0file4

Le mapping `structure` fournit le **socle** sur lequel les autres mappings (layout, logic, events, routing, actions, tests, etc.) vont venir se raccrocher via les UCR.

---

## 2. 📦 Format JSON racine (`mapping.structure.json`)

Le fichier est écrit dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.structure.json`fileciteturn0file3turn0file4

Schéma racine :

- `domain` : `"structure"`
- `pageName` : `${project.pageName}`
- `sourceEntry` : `${paths.legacySource}` (référence informative, jamais re‑scannée en Phase 2)
- `items` : tableau de `MappingItem` (un ou plusieurs par UCR issu de `inventory.structure.json`)
- `validation` : objet décrivant l’état global du mapping

Exemple minimal :

```json
{
  "domain": "structure",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Le contrat racine est identique à celui des autres mappings, seule la valeur de `domain` change.fileciteturn1file2

---

## 3. 🔗 Schéma d’un `MappingItem` pour la structure

### 3.1. Rappel du schéma générique

Un `MappingItem` relie :

- un **UCR** `structure.*` issu de `inventory.structure.json`,
- à une **projection de structure dans la stack cible** (vue, composant, conteneur, section, formulaire, etc.).fileciteturn0file3turn0file6

Schéma de base (hérité du template générique) :

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.structure.json",
    "domain": "structure",
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
    "structureUcrs": [],
    "logicUcrs": [],
    "eventUcrs": [],
    "dataflowUcrs": [],
    "asyncUcrs": [],
    "serviceUcrs": [],
    "routingUcrs": [],
    "effectUcrs": [],
    "actionUcrs": [],
    "testUcrs": [],
    "configNames": []
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires (spécifiques au domaine `structure`)

Pour `mapping.structure`, les champs suivants sont **obligatoires** :

- `ucr`  
  - Identifiant du mapping, unique dans `mapping.structure.json`.
  - Recommandation : préfixer avec `map-structure-…` et dériver l’UCR inventaire, par ex.  
    `view-page-CampaignsDetail-1` → `map-structure-viewPage-CampaignsDetail-1`.fileciteturn0file7

- `fromDsl`  
  - Doit être un concept de la famille `structure.*` du DSL :  
    `structure.page`, `structure.viewNode`, `structure.component`, `structure.container`, `structure.section`, `structure.form`, `structure.input`, `structure.button`, `structure.list`, `structure.item`, `structure.text`, `structure.layoutZone`, etc.fileciteturn0file6

- `sourceInventoryRef.file`  
  - Toujours `"inventory.structure.json"` pour ce domaine.

- `sourceInventoryRef.domain`  
  - Toujours `"structure"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact de l’élément issu de l’inventaire (`view-*`, `component-*`, `form-*`, etc.).

- `toStack.stackKind`  
  - Type d’artefact de structure dans la stack cible, parmi par exemple :
    - `"uiView"` (vue/page principale),
    - `"uiComponent"` (composant réutilisable),
    - `"uiContainer"` (wrapper/section logique),
    - `"uiForm"`,
    - `"uiFormField"`,
    - `"uiList"`,
    - `"uiListItem"`,
    - `"uiLayoutZone"`,
    - `"uiFragment"`.

- `toStack.targetId`  
  - Nom logique de l’artefact de structure dans la stack cible.  
  - Exemples : `CampaignsDetailView`, `CampaignSummaryCard`, `CampaignForm`, `CampaignsList`, `CampaignsFiltersSection`…

- `toStack.targetPath`  
  - Chemin relatif dans le projet cible, **tiré de `project-structure.json`**, jamais inventé hors de cette structure.  
  - Exemple : `src/pages/CampaignsDetail/components/CampaignsDetailView.tsx`.fileciteturn0file1turn0file3

- `toStack.targetLayer`  
  - Couche d’architecture dans la stack cible, typiquement :
    - `"presentation"` pour les vues, composants UI, containers visuels,
    - `"application"` pour certains conteneurs d’orchestration si la stack l’impose.

- `metadata.isCritical` / `metadata.priority`  
  - Doivent être renseignés pour toutes les vues principales, formulaires globaux et sections majeures.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Technologie concrète utilisée côté structure (ex. `"react"`, `"vue"`, `"solid"`, `"web-components"`…).

- `toStack.targetPattern`  
  - Pattern d’architecture UI, par ex. :
    - `"pageComponent"`,
    - `"presentationalComponent"`,
    - `"smartContainer"`,
    - `"layoutComponent"`,
    - `"formComponent"`,
    - `"listComponent"`.

- `toStack.hints[]`  
  - Conseils pour la génération et pour les humains : layout recommandé, design system à utiliser, conventions à respecter…

- `relations.structureUcrs`  
  - Permet de relier plusieurs UCR simples à un même artefact stack, par ex. regrouper plusieurs `structure.section` sous une même vue.  

- `relations.routingUcrs`, `relations.actionUcrs`, `relations.eventUcrs`  
  - Utiles pour relier une vue/section à :
    - la route qui y mène (`routing.*`),
    - les actions métier majeures qui s’y déclenchent (`action.*`),
    - les événements dominants (`event.*`).

---

## 4. ⚙️ Entrées attendues pour le mapping `structure`

Le stage 30 lit au minimum :fileciteturn0file3turn0file4

### 4.1. Configuration projet

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource` (pour `sourceEntry` uniquement)
- `paths.stages`
- `stack.custom`
- `gates.*`, `stages.*`

### 4.2. Artefacts Phase 0

- **Structure projet cible**  
  `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- **Bridge Legacy → DSL** (lecture, contexte)  
  `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- **Guides de stack (structure/UI)**  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.structure.md` (ou équivalent)  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.ui-components.md`

### 4.3. Inventaires Phase 1

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.structure.json` (inventaire primaire)
- `inventories-summary.json` (état global de la Phase 1)

### 4.4. Guides internes (core)

Dans `${paths.core}/guides-internals/` :

- `globals/guide.ucr.md`
- `dsl/guide.dsl.structure.md` (ou équivalent)
- `mapping/guide.mapping.structure.md` (ce guide)

---

## 5. 🧠 Règles d’interprétation pour `mapping.structure`

1. **Ne jamais re‑scanner le Legacy en Phase 2**  
   - Toute compréhension structurelle provient de `inventory.structure.json` + `project-structure.json` + stack‑guides.  
   - Le chemin Legacy n’est utilisé que comme référence dans `sourceEntry`.

2. **Respect absolu de `project-structure.json`**  
   - Aucun `targetPath` ne doit sortir de l’arborescence définie.  
   - Les dossiers/components doivent être choisis parmi ceux décrits dans `project-structure.json` (ou selon ses règles de génération).fileciteturn0file1turn0file3

3. **Pas de “sur‑invention” d’artefacts**  
   - Le mapping peut regrouper plusieurs éléments DSL vers un même artefact stack (ex. plusieurs sections vers un même composant), mais ne doit pas multiplier artificiellement les composants sans raison issue des guides ou de la structure cible.

4. **Traçabilité UCR complète**  
   - Chaque vue/composant mappé doit être traçable :
     - à son élément `structure.*` d’origine (via `sourceInventoryRef.itemUcr`),
     - aux actions/routes majeures qui l’utilisent (via `relations.*`).fileciteturn0file7

5. **Compatibilité avec les autres mappings**  
   - Les `targetId`/`targetPath` définis ici seront réutilisés par :
     - `mapping.layout` (placement des zones et templates),
     - `mapping.logic` / `mapping.events` / `mapping.effects` (logique attachée aux vues),
     - `mapping.routing` (entrypoints, routes),
     - `mapping.actions` (use cases complets).fileciteturn0file4

---

## 6. 📘 Exemple complet de `mapping.structure.json`

Exemple illustratif pour une page `CampaignsDetail` :

```json
{
  "domain": "structure",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-structure-viewPage-CampaignsDetail-1",
      "fromDsl": "structure.page",
      "sourceInventoryRef": {
        "file": "inventory.structure.json",
        "domain": "structure",
        "itemUcr": "view-page-CampaignsDetail-1"
      },
      "toStack": {
        "stackKind": "uiView",
        "targetId": "CampaignsDetailView",
        "targetPath": "src/pages/CampaignsDetail/CampaignsDetailView.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "react",
        "targetPattern": "pageComponent",
        "hints": [
          "Use MainLayout from stack-guides as outer layout",
          "Expose this view as main detail page for campaigns"
        ]
      },
      "relations": {
        "routingUcrs": ["routing-entry-CampaignsDetail-1"],
        "actionUcrs": [
          "action.userFlow-CampaignsDetail-saveCampaignFlow-1",
          "action.userFlow-CampaignsDetail-duplicateCampaignFlow-1"
        ]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Main detail view for campaigns."
      }
    },
    {
      "ucr": "map-structure-form-CampaignsDetail-main-1",
      "fromDsl": "structure.form",
      "sourceInventoryRef": {
        "file": "inventory.structure.json",
        "domain": "structure",
        "itemUcr": "form-CampaignsDetail-main-1"
      },
      "toStack": {
        "stackKind": "uiForm",
        "targetId": "CampaignForm",
        "targetPath": "src/pages/CampaignsDetail/components/CampaignForm.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "react",
        "targetPattern": "formComponent",
        "hints": [
          "Use FormProvider from stack-guides if available",
          "Group budget-related fields under dedicated section"
        ]
      },
      "relations": {
        "structureUcrs": [
          "input-CampaignsDetail-name-1",
          "input-CampaignsDetail-budget-1"
        ],
        "actionUcrs": ["action.formSubmit-CampaignsDetail-saveCampaign-1"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Main business form of the page."
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

## 7. ✅ Checklist de validation pour `mapping.structure`

Avant de considérer le mapping comme valide :

- [ ] `domain === "structure"`  
- [ ] `pageName === project.pageName`  
- [ ] `sourceEntry === paths.legacySource`  
- [ ] Tous les `ucr` sont uniques dans `items[]` et conformes aux règles UCR  
- [ ] Tous les `fromDsl` appartiennent à la famille `structure.*` du DSL  
- [ ] Chaque `sourceInventoryRef.itemUcr` existe dans `inventory.structure.json`  
- [ ] Chaque `toStack.stackKind` est compatible avec les patterns de la stack cible  
- [ ] Chaque `toStack.targetPath` respecte `project-structure.json`  
- [ ] Les éléments critiques (vue principale, formulaires globaux) ont `metadata.isCritical = true`  
- [ ] Le JSON complet valide le schéma formel (si disponible)  
- [ ] `validation.status` vaut `"valid"` ou `"rejected"` et `validation.issues` est cohérent

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.structure` (Stage 30 — Phase 2 : Interprétation)*
