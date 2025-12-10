# 🧭 Guide de Mapping — `mapping.styles`

*(Projection des concepts `styles.*` du DSL vers le système de design, les thèmes et styles de la stack cible)*

---

## 1. 🎯 Rôle du mapping `styles`

Le domaine `styles.*` du DSL décrit la **dimension visuelle** :

- couleurs, typographies, espacements ;
- variantes visuelles (états, tailles, importance) ;
- styles applicables aux composants, sections, layouts ;
- liens éventuels avec des design tokens ou un design system.

La **Phase 1 — Analyse** a produit :

- `inventory.styles.json` → inventaire des UCR `styles.*` ;
- des liens potentiels vers la structure et le layout (ex. styles attachés à des `structure.*` ou `layout.*`).

La **Phase 2 — Stage 32 — mapping.styles** doit :

> **Projeter chaque UCR `styles.*` vers un artefact de styling de la stack cible**,  
> en s’appuyant sur :
> - les guides de stack (design system, CSS / CSS-in-JS, thème global, etc.) ;
> - la structure cible du projet (`project-structure.json`) ;
> - les mappings structurels et de layout déjà produits (`mapping.structure.json`, `mapping.layout.json`).

L’objectif est de **relier le DSL des styles** à :

- des design tokens ;
- des thèmes globaux ;
- des modules de styles (CSS, modules, styled-components, Tailwind, etc.) ;
- des props ou variantes de composants du design system.

---

## 2. 📦 Format JSON racine (`mapping.styles.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.styles.json`

Structure racine attendue :

```json
{
  "domain": "styles",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

- `domain` : `"styles"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (uniquement informatif)  
- `items` : tableau de `MappingItem`  
- `validation` : statut global du mapping et éventuelles anomalies

---

## 3. 🔗 Schéma d’un `MappingItem` pour les styles

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `styles.*` issu de `inventory.styles.json` ;
- à un artefact de styling dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.styles.json",
    "domain": "styles",
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
    "layoutUcrs": [],
    "tokenNames": []
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
  - Identifiant du mapping, **unique** dans `mapping.styles.json`.  
  - Recommandation : préfixe `map-styles-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Doit être un concept de la famille `styles.*` du DSL (ex : `styles.color`, `styles.typography`, `styles.spacing`, `styles.variant`, `styles.state`, etc.).

- `sourceInventoryRef.file`  
  - Toujours `"inventory.styles.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"styles"`.

- `sourceInventoryRef.itemUcr`  
  - UCR d’origine dans l’inventaire.

- `toStack.stackKind`  
  - Type d’artefact de styling, par exemple :
    - `"styleToken"` (design token) ;
    - `"themeEntry"` (entrée dans le thème global) ;
    - `"componentStyle"` ;
    - `"variantProps"` (mapping vers une prop `variant` sur un composant) ;
    - `"utilityClass"` (classe utilitaire type Tailwind) ;
    - `"globalStyle"`.

- `toStack.targetId`  
  - Nom de la ressource de style dans la stack cible :  
    - nom de token, clé de thème, nom de fichier de style, etc.

- `toStack.targetPath`  
  - Chemin relatif, dérivé de `project-structure.json`, vers l’endroit où :
    - se trouve la définition de styles ;
    - ou sera généré le fichier de styles (par ex. `src/styles/tokens.ts` ou `src/pages/.../CampaignsDetail.styles.ts`).

- `toStack.targetLayer`  
  - Souvent `"presentation"` ou `"design"` selon les conventions internes.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Technologie de styling : `"css-modules"`, `"tailwind"`, `"styled-components"`, `"emotion"`, `"vanilla-css"`, etc.

- `toStack.targetPattern`  
  - Pattern utilisé : `"tokenFile"`, `"themeObject"`, `"componentStylesheet"`, `"utilityFirst"`, etc.

- `toStack.hints[]`  
  - Conseils : par ex. `"Utiliser la palette primary du design system"`, `"Exposer la variante 'danger'"`…

- `relations.structureUcrs`  
  - UCR de structure liés à ce style (composant, section, vue…).

- `relations.layoutUcrs`  
  - UCR de layout attachés (zone, colonne, grid…).

- `relations.tokenNames`  
  - Noms des tokens ou clés de thème associés (si déjà connus).

---

## 4. ⚙️ Entrées requises pour `mapping.styles`

### 4.1. Configuration

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

### 4.2. Artefacts Phase 0

- **Structure projet cible**  
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- **Bridge Legacy → DSL**  
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- **Guides de stack (design / styles)**  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.styles.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.design-system.md` (ou équivalent)

### 4.3. Inventaires Phase 1

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.styles.json` (inventaire primaire)  
- `inventories-summary.json`

### 4.4. Mappings déjà produits

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`  
- `mapping.layout.json` (si existant)

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Les décisions de mapping se basent sur l’inventaire `styles`, la structure cible, les stack-guides et les mappings précédents.

2. **Styles alignés sur le design system**  
   - Toujours privilégier tokens, variables de thème et composants du design system plutôt que des styles ad hoc.

3. **Respect de `project-structure.json`**  
   - `targetPath` ne doit jamais sortir de l’arborescence définie pour le projet.

4. **Regroupement logique**  
   - Plusieurs UCR `styles.*` peuvent être regroupés dans un même artefact (par ex. un fichier de styles ou une entrée de thème).

5. **Traçabilité**  
   - Chaque UCR de styles doit pouvoir être relié à ses cibles structurelles/layout via `relations.structureUcrs` / `relations.layoutUcrs` quand c’est pertinent.

---

## 6. Exemple simplifié de `mapping.styles.json`

```json
{
  "domain": "styles",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-styles-pageBackground-CampaignsDetail-1",
      "fromDsl": "styles.background",
      "sourceInventoryRef": {
        "file": "inventory.styles.json",
        "domain": "styles",
        "itemUcr": "styles-background-CampaignsDetail-1"
      },
      "toStack": {
        "stackKind": "themeEntry",
        "targetId": "campaignsDetail.background",
        "targetPath": "src/styles/theme/campaignsDetail.ts",
        "targetLayer": "design",
        "targetTechnology": "css-in-js",
        "targetPattern": "themeObject",
        "hints": [
          "Utiliser la couleur primary.100 comme fond",
          "Conserver le contraste AA avec le texte par défaut"
        ]
      },
      "relations": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "tokenNames": ["color.primary.100"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Fond principal de la page de détail campagne."
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

- [ ] `inventory.styles.json` présent et déclaré comme valide dans `inventories-summary.json`  
- [ ] `mapping.structure.json` accessible  
- [ ] `mapping.layout.json` accessible (si nécessaire)  
- [ ] Tous les UCR `styles.*` importants sont mappés  
- [ ] Chaque `MappingItem` possède un `stackKind`, `targetId`, `targetPath`, `targetLayer`  
- [ ] Tous les `targetPath` sont compatibles avec `project-structure.json`  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et `validation.issues` est cohérent  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.styles` (Stage 32 — Phase 2 : Interprétation)*
