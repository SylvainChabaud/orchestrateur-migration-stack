# 🧩 Stage 31 — mapping.layout

**Phase :** Phase 2 — Interprétation  
**Précédent :** 30 — mapping.structure  
**Suivant :** 32 — mapping.styles  

---

## 🎯 Objectif

Construire `mapping.layout.json` à partir de :

- `inventory.layout.json` (Phase 1) ;
- la structure cible (`project-structure.json`) ;
- les guides de stack ;
- les résultats de `mapping.structure.json`.

Aucune relecture du Legacy n’est effectuée.

---

## ⚙️ Entrées requises

Toutes les entrées proviennent de `core/configs/project.config.yaml`.

### Configuration

- `project.name`
- `project.pageName`
- `paths.root`, `paths.core`, `paths.workspace`
- `paths.legacySource` (référence uniquement)
- `stack.custom`
- `gates.*`  
- `stages.*`

### Artefacts Phase 0

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.layout.md` (si existant)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.ui-components.md`

### Inventaires Phase 1

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.layout.json`
- `inventories-summary.json`

### Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Layout**
  - `${paths.core}/guides-internals/mapping/guide.mapping.layout.md`
  - Fournit :
    - l'objectif du mapping de layout,
    - le schéma JSON contractuel de `mapping.layout.json`,
    - les règles de projection des UCR `layout.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

### Mappings Phase 2 déjà produits

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.structure.json`

Si un de ces inputs est manquant → **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.layout.json`

Structure attendue :

```jsonc
{
  "domain": "layout",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": { "status": "pending", "issues": [] }
}
```

---

## 🧠 Actions

### 1. Charger la configuration et les artefacts nécessaires

### 2. Vérifier la présence des inventaires layout

- Si `inventory.layout.json` ou `inventories-summary.json` est absent → **Gate ❌**.

### 3. Construire l’objet racine `mappingRoot`

```jsonc
{
  "domain": "layout",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": { "status": "pending", "issues": [] }
}
```

### 4. Pour chaque UCR `layout.*`

- Lire l’entrée dans l’inventaire ;  
- Déterminer :  
  - `stackKind` (`layoutZone`, `layoutTemplate`, etc.) ;  
  - `targetId` (nom du layout) ;  
  - `targetPath` (basé sur `project-structure.json`) ;  
- Construire le `MappingItem` ;  
- Ajouter aux `items[]`.

### 5. Renseigner les `relations`

- `parentLayout` ;
- `childLayouts` ;
- `structureUcrs` pour attacher les zones aux vues/sections.

### 6. Validation

- Un seul layout racine si le DSL l’exige ;  
- Tous les `targetPath` conformes ;
- Un mapping par UCR layout.*.

Si un problème bloquant → `validation.status = "rejected"`.

### 7. Écriture du fichier

Écrire dans :  

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.layout.json`

---

## 🧩 Gate

Afficher l’un des deux blocs :

```
## 🧩 Gate
Gate ✅
```

ou

```
## 🧩 Gate
Gate ❌
```

---

## Résumé JSON retourné par l’IA

```json
{
  "stageId": "31",
  "stageName": "mapping.layout",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

---

© 2025 — ai-orchestrator-v4
