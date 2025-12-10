# 🧩 Stage 32 — mapping.styles

**Phase :** Phase 2 — Interprétation  
**Précédent :** 31 — mapping.layout  
**Suivant :** 33 — mapping.i18n  

---

## 🎯 Objectif

Construire le fichier `mapping.styles.json` pour la page `${project.pageName}` en projetant chaque UCR `styles.*` de `inventory.styles.json` vers des artefacts de styling de la stack cible (design tokens, thème, fichiers de styles, variantes de composants, etc.), en s’alignant sur :

- la structure cible du projet (`project-structure.json`) ;
- les guides de stack (design system, styles) ;
- les mappings déjà existants (`mapping.structure.json`, `mapping.layout.json`).

Aucune relecture du Legacy n’est effectuée à ce stage.

---

## ⚙️ Entrées requises

> Toutes les entrées proviennent directement ou indirectement de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### Configuration

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

### Artefacts de la Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.styles.md` (si existant)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.design-system.md` (ou équivalent)

### Inventaires de la Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.styles.json`
- `inventories-summary.json`

### Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Styles**
  - `${paths.core}/guides-internals/mapping/guide.mapping.styles.md`
  - Fournit :
    - l'objectif du mapping de styles,
    - le schéma JSON contractuel de `mapping.styles.json`,
    - les règles de projection des UCR `styles.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

### Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json` (si existant)

Si un des fichiers obligatoires est absent ou invalide → **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.styles.json`

Racine attendue :

```jsonc
{
  "domain": "styles",
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

### 1. Charger configuration et contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre toutes les interpolations `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître l’arborescence du projet cible.  
1.3. Charger `bridge-legacy-to-dsl.json` (contexte, sans ré-analyse du Legacy).  
1.4. Charger les guides de stack liés aux styles / design system.

### 2. Vérifier la présence et l’état des inventaires

2.1. Charger `inventory.styles.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `styles` pour `${project.pageName}` est présent et valide.  
2.3. Si l’inventaire `styles` est manquant ou invalide :
- initialiser un `mapping.styles` minimal ;
- positionner `validation.status = "rejected"` et ajouter une issue explicite ;
- sortir avec **Gate ❌**.

### 3. Charger les mappings précédents

3.1. Charger `mapping.structure.json` (obligatoire).  
3.2. Charger `mapping.layout.json` si disponible.  
3.3. Construire en mémoire des helpers pour relier :  
- UCR de styles → UCR de structure ;  
- UCR de styles → UCR de layout (si l’inventaire les expose).

### 4. Initialiser l’objet racine du mapping

Construire en mémoire :

```jsonc
{
  "domain": "styles",
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

### 5. Projeter chaque UCR `styles.*`

Pour chaque entrée de `inventory.styles.json` :

5.1. Lire :  
- `item.ucr` ;  
- `item.dsl` (concept `styles.*`) ;  
- les métadonnées (type de style, intensité, contexte d’utilisation, liens vers structure/layout…).

5.2. Déterminer `toStack.stackKind` en fonction :  
- du concept DSL (`styles.color`, `styles.variant`, `styles.spacing`, etc.) ;  
- des conventions du design system ;  
- de la structure / layout associés (component vs global).

Exemples de `stackKind` :  
- `"styleToken"` ;  
- `"themeEntry"` ;  
- `"componentStyle"` ;  
- `"variantProps"` ;  
- `"utilityClass"` ;  
- `"globalStyle"`.

5.3. Déterminer `toStack.targetId` :  
- nom du token, de la clé de thème, du module de styles, etc.  
- respecter les conventions définies dans les guides.

5.4. Déduire `toStack.targetPath` à partir de `project-structure.json` :  
- pour les tokens/thèmes globaux : fichier de thème ou de tokens ;  
- pour les styles de composant : fichier de styles situé à côté du composant cible.

5.5. Définir `toStack.targetLayer` :  
- `"design"` ou `"presentation"` selon la convention retenue.

5.6. Optionnel :  
- `toStack.targetTechnology` (`css-modules`, `tailwind`, etc.) ;  
- `toStack.targetPattern` (`tokenFile`, `themeObject`, `componentStylesheet`, etc.) ;  
- `toStack.hints[]` (recommandations de mise en œuvre).

5.7. Construire un `MappingItem` :

- `ucr` : identifiant de mapping unique (souvent `map-styles-${item.ucr}`).  
- `fromDsl` : concept `styles.*`.  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.styles.json",
    "domain": "styles",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.structureUcrs` : UCR de structure liés (s’il y en a).  
- `relations.layoutUcrs` : UCR de layout liés (s’il y en a).  
- `relations.tokenNames` : noms de tokens / clés de thème (si connus).  
- `metadata` : marquer `isCritical = true` et `priority = "high"` pour :  
  - les styles d’éléments clés (ex : fond principal de page, CTA, sections critiques).

5.8. Ajouter le `MappingItem` à `mappingRoot.items[]`.

### 6. Validation interne

6.1. Vérifier :  
- `mappingRoot.domain === "styles"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- tous les `ucr` de mapping sont uniques ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.styles.json` ;  
- chaque `MappingItem.toStack` possède un `stackKind`, `targetId`, `targetPath`, `targetLayer`.

6.2. Si un schéma JSON formel existe pour `mapping.styles.json`, valider `mappingRoot` contre ce schéma.

6.3. En cas de problème bloquant :  
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :  
- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `mappingRoot.validation.issues` est un tableau (éventuellement vide).

### 7. Écriture de la sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.styles.json`

7.2. Créer les dossiers manquants si besoin.  
7.3. Ne modifier aucun autre fichier.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer ce résumé (non écrit sur disque) :

```json
{
  "stageId": "32",
  "stageName": "mapping.styles",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

- `inputsAvailable` = `false` si une entrée obligatoire est manquante.  
- `schemaValidated` = `false` si la validation de schéma échoue ou n’a pas été faite.  
- `outputsWritten` = `false` si l’écriture du fichier de sortie échoue.

---

## 🧩 Gate

Le stage doit se terminer par **exactement l’un** des blocs suivants :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` en cas de problème bloquant (inputs manquants, inventaire invalide, schéma non respecté, sortie non écrite, etc.).

---

## 📦 Stage suivant

> Continuer avec `33-mapping-i18n.md` uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
