# 🔧 Guide Inventaire — Styles (`inventory.styles`)

*(Domaine d’inventaire : **Styles** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Styles** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **sources de styles** utilisées (fichiers CSS, CSS-in-JS, thèmes, tokens, classes utilitaires…).
2. Les **groupes de styles** significatifs (globaux, par composant, par région).
3. Les **associations entre styles et vues** de la page (via les `ucr` de `inventory.structure.json`).
4. Les **thèmes / variantes** (light/dark, primary/secondary, etc.) lorsqu’ils sont présents.

L’inventaire répond à la question :

> **“Quels styles importants structurent l’apparence de cette page, et à quelles vues sont-ils rattachés ?”**

Ce domaine ne :

- **ne remplace pas** une feuille de style complète,
- **ne décrit pas** le layout spatial (→ `inventory.layout`),
- **ne détaille pas** toutes les règles CSS bas niveau, mais les regroupe en **unités de sens** exploitables pour la migration.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.styles.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"styles"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des groupes de styles (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

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

---

### 2.2. Schéma interne — `items[]`

Chaque élément de `items[]` représente un **groupe de styles** (StyleItem) appliqué à une ou plusieurs vues.

```text
items[] : StyleItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du groupe de styles, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Styles.

- `kind` : string  
  Type de groupe de styles, parmi par exemple :
  - `"global"` (styles globaux à la page),
  - `"component"` (styles associées à un composant/vue spécifique),
  - `"region"` (styles associées à une région de layout),
  - `"variant"` (thème/variantes comme `primary`, `secondary`),
  - `"state"` (hover, focus, disabled, etc. au sens visuel).

- `name` : string  
  Nom logique du groupe de styles, par exemple :
  - `"PageBackgroundMain"`,
  - `"ProductCardDefault"`,
  - `"ProductCardHoverVariant"`,
  - `"SidebarFilterRegionStyles"`.

- `sourcePath` : string  
  Chemin Legacy principal où est défini ce groupe de styles, par exemple :
  - feuille CSS,
  - fichier de CSS-in-JS,
  - composant utilisant un système de style (MUI `sx`, tailwind, etc.).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) auxquels ces styles s’appliquent directement.

- `styleSummary` : object  
  Résumé structuré du contenu des styles, par exemple :
  - `layout`: `"flex-row"`, `"grid-3-columns"`, `"inline-block"`, …
  - `colors`: liste de noms/logiques (`["bg-primary", "text-muted"]`),
  - `typography`: infos de base (`"heading"`, `"body"`, `"caption"`),
  - `effects`: `["shadow", "bordered", "rounded"]`,
  - etc.  
  Peut être `{}` au minimum, mais doit exister.

- `metadata` : object  
  Informations additionnelles :
  - `isThemeRoot`: booléen indiquant si ce groupe représente un pivot de thème,
  - `isOverride`: booléen indiquant s’il surcharge un autre groupe,
  - `notes`: string optionnel pour commentaires techniques.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedLayoutUcrs` : array de string  
  Liste des `ucr` de layout (s’ils existent, depuis `inventory.layout.json`) avec lesquels ce groupe de styles est étroitement lié (ex : styles d’une région spécifique).

- `variants` : array d’objets  
  Décrit des variantes du groupe de styles (ex : `primary`, `secondary`, `danger`, `success`), par exemple :
  ```json
  {
    "name": "primary",
    "styleSummary": { "colors": ["bg-primary", "text-on-primary"] }
  }
  ```

- `states` : array d’objets  
  Décrit des styles spécifiques à certains états visuels (`hover`, `focus`, `active`, `disabled`, etc.).

- `responsive` : object  
  Peut contenir :
  - `breakpoints`: array (ex : `["xs", "md", "lg"]`),
  - `behavior`: descriptions courtes (ex : `"stack-on-xs"`, `"collapse-sidebar-on-md"`).

Tout champ optionnel utilisé doit être **documenté** ici et respecté dans toute la pipeline.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de styles doivent être **uniques** dans `inventory.styles.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Tous les `relatedLayoutUcrs` (si utilisés) doivent référencer des `ucr` valides de `inventory.layout.json`.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts mobilisés

L’inventaire Styles ne définit pas (à ce stade) de nouveaux IDs DSL dédiés, mais s’appuie sur :

- la **Structure** (via les `ucr` de `inventory.structure.json`) pour ancrer les styles sur des vues,
- éventuellement le **Layout** (régions, grilles) pour contextualiser certains styles,
- les conventions de la stack cible (décrites dans les guides de stack) pour la granularité attendue.

Si le DSL interne est étendu à l’avenir avec un domaine `styles.*`, le guide pourra être mis à jour afin de référencer explicitement ces IDs dans `styleSummary` ou `dslTags`.

### 3.2. Règles d’analyse

L’inventaire Styles doit :

1. Identifier les **sources de styles** pertinentes pour la page :
   - imports CSS/SCSS,
   - CSS-in-JS,
   - classes utilitaires,
   - thèmes / design system.
2. Regrouper les styles en **unités logiques** (StyleItem) :
   - par composant,
   - par région de layout,
   - par thème ou variante.
3. Associer chaque StyleItem à une ou plusieurs vues (via `targetStructureUcrs`).
4. Résumer ces styles dans `styleSummary` avec un vocabulaire structuré, pas du texte brut.

### 3.3. Restrictions

L’inventaire Styles **ne doit pas** :

- réimplémenter la feuille de style complète,
- parser ou reproduire chaque propriété CSS au niveau le plus fin,
- décider de la manière exacte dont les styles seront représentés dans la stack cible (cela appartient à la Phase 2 & 3),
- introduire une dépendance directe à une technologie cible (ex. tailwind, MUI) dans le JSON contractuel.

---

## 4. 🔗 Relations avec les autres inventaires

- **Styles ← Structure**
  - Les `targetStructureUcrs` pointent vers des vues décrites dans `inventory.structure.json`.

- **Styles ← Layout**
  - Facultatif mais utile : `relatedLayoutUcrs` peut référencer des régions de layout pour contextualiser certains groupes de styles.

- **Styles → i18n / Actions / Tests**
  - Les styles peuvent être utiles pour certains scénarios (ex. éléments visuellement mis en avant), mais ils ne doivent pas dicter la logique.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de styles sont uniques.
- [ ] Tous les `targetStructureUcrs` pointent vers des `ucr` valides de `inventory.structure.json`.
- [ ] Tous les `relatedLayoutUcrs` (si présents) pointent vers des `ucr` de `inventory.layout.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `styleSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "styles",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "styles-global-page-bg-1",
      "kind": "global",
      "name": "PageBackgroundMain",
      "sourcePath": "src/legacy/styles/pages/ProductListPage.css",
      "targetStructureUcrs": ["view-root-1"],
      "styleSummary": {
        "layout": ["full-viewport"],
        "colors": ["bg-page", "text-default"]
      },
      "metadata": {
        "isThemeRoot": true
      }
    },
    {
      "ucr": "styles-component-product-card-default-1",
      "kind": "component",
      "name": "ProductCardDefault",
      "sourcePath": "src/legacy/components/ProductCard/ProductCard.css",
      "targetStructureUcrs": ["view-container-1"],
      "styleSummary": {
        "layout": ["card"],
        "colors": ["bg-surface", "text-primary"],
        "effects": ["shadow", "rounded"]
      },
      "metadata": {}
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

### 6.2. Exemple invalide (commenté)

```json
{
  "domain": "styles",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "styles-component-product-card-default-1",
      "kind": "component",
      "name": "ProductCardDefault",
      "sourcePath": "src/legacy/components/ProductCard/ProductCard.css",
      "targetStructureUcrs": ["view-unknown-99"],
      "styleSummary": {},
      "metadata": {}
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Problèmes :

- `targetStructureUcrs` contient `view-unknown-99` qui n’existe pas dans `inventory.structure.json`.
- `validation.status` ne devrait pas être `"valid"` dans ce cas, et une issue devrait être documentée.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"styles"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de styles sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Les `relatedLayoutUcrs` (si présents) sont valides vis-à-vis de `inventory.layout.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne pas chercher à “rejouer” tout le CSS ; se concentrer sur des groupes de styles significatifs pour la migration.
- S’appuyer systématiquement sur :
  - `inventory.structure.json` (UCR de vues),
  - `inventory.layout.json` (si présent),
  - les guides de stack pour comprendre la cible,
  - le bridge Legacy → DSL uniquement s’il fournit des indices utiles.
- Utiliser `validation.issues` pour documenter :
  - les styles non rattachés à des vues,
  - les styles très difficiles à projeter,
  - les ambiguïtés ou manques de contexte.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Styles*
