# 🧭 Guide de Mapping — `mapping.layout`

*(Projection des concepts `layout.*` du DSL vers les layouts, zones, templates d’affichage de la stack cible)*

---

## 1. 🎯 Rôle du mapping `layout`

Le domaine `layout.*` du DSL décrit l'organisation visuelle, les zones de page, les templates, les conteneurs spatiaux, les grilles, les colonnes, et toutes les structures d’affichage qui ne représentent pas du contenu fonctionnel mais du **positionnement**.

La **Phase 1** a produit l’inventaire `inventory.layout.json`, contenant :

- les zones de layout (`layout.zone`, `layout.header`, `layout.footer`, `layout.sidebar`, `layout.grid`, `layout.column`, etc.) ;
- les relations hiérarchiques entre zones ;
- les UCR de layout associés ;
- parfois des liens vers des `structure.*` (ex : une section UI attachée à une zone).

La **Phase 2 — Stage 31** doit :

> **Projeter chaque UCR `layout.*` vers un composant / template de layout de la stack cible**,  
> en respectant :
> - les conventions du design system ;
> - les règles de layout décrites dans les guides de stack ;
> - la structure cible (`project-structure.json`) ;
> - les artefacts générés par le mapping de structure (`mapping.structure.json`).

Ce mapping complète la projection structurelle en précisant *comment la page est organisée spatialement*.

---

## 2. 📦 Format JSON racine (`mapping.layout.json`)

Écrit dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.layout.json`

Structure attendue :

```json
{
  "domain": "layout",
  "pageName": "...",
  "sourceEntry": "...",
  "items": [],
  "validation": { "status": "pending", "issues": [] }
}
```

---

## 3. 🔗 Structure d’un `MappingItem`

```jsonc
{
  "ucr": "map-layout-...",
  "fromDsl": "layout.*",
  "sourceInventoryRef": {
    "file": "inventory.layout.json",
    "domain": "layout",
    "itemUcr": "layout-..."
  },
  "toStack": {
    "stackKind": "layoutComponent | layoutZone | layoutTemplate",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "presentation",
    "hints": []
  },
  "relations": {
    "structureUcrs": [],
    "childLayouts": [],
    "parentLayout": null
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### Champs obligatoires

- `ucr` → identifiant unique du mapping (préfixe recommandé : `map-layout-...`)
- `fromDsl` → un concept `layout.*` du DSL
- `sourceInventoryRef.file` = `"inventory.layout.json"`
- `toStack.stackKind` → parmi : `layoutComponent`, `layoutZone`, `layoutTemplate`
- `toStack.targetId` → nom du composant / template de layout
- `toStack.targetPath` → **dérivé de `project-structure.json`**
- `toStack.targetLayer` → `"presentation"`

### Champs optionnels

- `hints[]` → conseils de construction (ex: "Utiliser Grid du design system")
- `relations.*` → liens structurels / layout
- `metadata.*` → marquage de criticité

---

## 4. ⚙️ Entrées requises

### 4.1. Configuration

Depuis `${paths.core}/configs/project.config.yaml` :

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

### 4.2. Artefacts de Phase 0

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- guides de stack :  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.layout.md` (si présent)
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.ui-components.md`

### 4.3. Inventaires Phase 1

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.layout.json`
- `inventories-summary.json`

### 4.4. Mappings Phase 2 déjà produits

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.structure.json`

---

## 5. 🧠 Règles d’interprétation

1. **Jamais de relecture du Legacy**  
   La Phase 2 n’analyse plus le code d’origine.

2. **Le layout s'appuie sur la structure**  
   Une zone de layout doit être attachée à un nœud structurel (ex : section, vue).

3. **Respect total de `project-structure.json`**  
   Les chemins `targetPath` doivent provenir de cette structure.

4. **Un layout peut englober d'autres layouts**  
   → utiliser `relations.childLayouts` et `relations.parentLayout`.

5. **Un seul layout racine par page**  
   → celui dont `fromDsl = layout.root` si existant.

---

## 6. Exemple simplifié

```json
{
  "domain": "layout",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/...",
  "items": [
    {
      "ucr": "map-layout-main-1",
      "fromDsl": "layout.zone",
      "sourceInventoryRef": {
        "file": "inventory.layout.json",
        "domain": "layout",
        "itemUcr": "layout-zone-main-1"
      },
      "toStack": {
        "stackKind": "layoutZone",
        "targetId": "CampaignsDetailMainZone",
        "targetPath": "src/pages/CampaignsDetail/layout/MainZone.tsx",
        "targetLayer": "presentation",
        "hints": ["Utiliser Grid du design system"]
      },
      "relations": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "childLayouts": ["layout-zone-sidebar-1"],
        "parentLayout": null
      },
      "metadata": { "isCritical": true, "priority": "high" }
    }
  ],
  "validation": { "status": "valid", "issues": [] }
}
```

---

## 7. Checklist

- [ ] L’inventaire layout existe et est valide  
- [ ] Tous les UCR layout.* sont mappés  
- [ ] Tous les `targetPath` sont cohérents avec `project-structure.json`  
- [ ] Un layout racine identifié si nécessaire  
- [ ] `validation.status` = `"valid"` ou `"rejected"`  

---

© 2025 — ai-orchestrator-v4  
