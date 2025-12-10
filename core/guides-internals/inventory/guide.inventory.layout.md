# 🔧 Guide Inventaire — Layout (`inventory.layout`)

*(Domaine d’inventaire : **Layout** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Layout** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **régions de layout** (ex : header, sidebar, main, footer, panels, drawers, overlays…).
2. Les **structures de disposition** (grilles, stacks, listes, tables…).
3. Les **relations** entre ces éléments de layout et les vues de structure (référencées par leurs `ucr`).
4. Les **comportements responsives majeurs** (breakpoints principaux, bascules de layout).

Il répond à la question :

> **“Comment les vues de la page sont organisées visuellement et spatialement, indépendamment de la logique métier et de la stack cible ?”**

Le domaine Layout ne :

- **ne décrit pas** la logique métier (`inventory.logic`),
- **ne détaille pas** les styles fins (couleurs, typographies — `inventory.styles`),
- **ne gère pas** les flux de données (`inventory.dataflows`, `inventory.services`),
- **ne génère pas** de code cible (Phase 3 – Génération).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.layout.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"layout"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des éléments de layout (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "layout",
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

Chaque élément de `items[]` représente un **élément de layout** (région ou structure de disposition).

```text
items[] : LayoutItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du layout item, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Layout.
  - Peut suivre une convention propre au domaine (ex : `layout-region-main-1`), tant qu’elle respecte les règles générales UCR.

- `type` : string  
  Type de layout parmi un ensemble contrôlé, par exemple :
  - `"region"`,
  - `"grid"`,
  - `"stack"`,
  - `"list"`,
  - `"table"`,
  - `"modal"`,
  - `"drawer"`,
  - `"overlay"`,
  - `"panel"`.

- `name` : string  
  Nom logique du layout item, par exemple :
  - `"MainContentRegion"`,
  - `"SidebarFilters"`,
  - `"ProductGrid"`,
  - `"FooterLinks"`.

- `sourcePath` : string  
  Chemin Legacy du fichier où se trouve la structure de layout principale (ex : `src/legacy/components/Layout/MainLayout.jsx`).

- `dslTags` : array de string  
  Liste des IDs DSL pertinents pour ce layout item, typiquement dans le domaine `layout.*`, par exemple :
  - `["layout.region"]`,
  - `["layout.grid"]`,
  - `["layout.region", "layout.responsiveBreakpoint"]`.

- `structureUcrs` : array de string  
  Liste des `ucr` de **structure** (provenant de `inventory.structure.json`) associés à ce layout item, par exemple :
  - la vue racine de la région,
  - les vues principales composant la grille ou la liste.

- `metadata` : object  
  Objet libre permettant de transporter des informations dérivées, par exemple :
  - `role`: `"header" | "main" | "sidebar" | "footer" | "overlay" | ...`
  - `order`: numéro d’ordre dans la page ou dans le parent,
  - `isSticky`: booléen,
  - `isScrollable`: booléen.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `parentLayoutUcr` : string | null  
  - `null` si l’item est en haut de la hiérarchie de layout,
  - `ucr` d’un autre `LayoutItem` s’il est imbriqué (ex : une grille dans une région).

- `childrenLayoutUcrs` : array de string  
  - liste ordonnée des `ucr` des layout items enfants (sous-régions, sous-grilles, etc.).

- `responsive` : object  
  Peut contenir des informations comme :
  - `breakpoints`: array (ex : `["xs", "sm", "md", "lg", "xl"]`),
  - `behavior`: descriptions courtes (ex : `"stack-to-column"`, `"sidebar-collapses"`, `"grid-2-to-1"`).

- `gridConfig` : object  
  Pour les items de type `"grid"` :
  - `columns`: nombre de colonnes ou description,
  - `gap`: indication générique (ex : `"small"`, `"medium"`, `"large"`),
  - `hasMasonry`: booléen.

- `listConfig` : object  
  Pour les listes / tables :
  - `isVirtualized`: booléen,
  - `hasStickyHeader`: booléen,
  - `hasPagination`: booléen.

Tout champ optionnel utilisé doit être **documenté** ici.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de layout doivent être **uniques** dans `inventory.layout.json`.
- Tous les `structureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- `parentLayoutUcr` (s’il est présent) doit référencer un `ucr` existant de `inventory.layout.json`.
- `childrenLayoutUcrs` (s’il est présent) doit contenir uniquement des `ucr` valides de layout.
- La hiérarchie de layout ne doit pas introduire de **cycle**.
- Aucune clé inconnue ne doit apparaître au niveau racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Principalement dans le domaine `layout.*` :

- `layout.region`
- `layout.section`
- `layout.grid`
- `layout.stack`
- `layout.card`
- `layout.list`
- `layout.table`
- `layout.modal`
- `layout.drawer`
- `layout.panel`
- `layout.overlay`
- `layout.responsiveBreakpoint`

Le mapping concret vers le Legacy est fourni par le **bridge** `bridge-legacy-to-dsl.json`.

### 3.2. Règles d’analyse

L’inventaire Layout doit :

1. S’appuyer sur `inventory.structure.json` pour :
   - obtenir la liste des vues et leurs `ucr`,
   - naviguer dans la hiérarchie parent → enfants.
2. Analyser le code Legacy (via le bridge) pour identifier :
   - les régions de layout globales (header, main, sidebar, footer, overlays…),
   - les structures de grilles, stacks, listes, tableaux,
   - les patterns évidents de responsive (breakpoints, bascule mobile/desktop…).
3. Créer un `LayoutItem` pour chaque région/structure significative, en :
   - liant l’item à un ou plusieurs `structureUcrs`,
   - assignant le `type`, le `name`, les `dslTags`,
   - renseignant les métadonnées utiles (`role`, `order`, etc.).

### 3.3. Restrictions

L’inventaire Layout **ne doit pas** :

- détailler les styles graphiques (couleurs, typos → `inventory.styles`),
- gérer la logique métier, les conditions d’affichage complexes (→ `inventory.logic`, `inventory.conditions`),
- décrire les flux de données (→ `inventory.dataflows`, `inventory.services`),
- se lier directement à la stack cible (React 19) : il reste **framework-agnostique**.

---

## 4. 🔗 Relations avec les autres inventaires

- **Layout ← Structure**
  - Layout réutilise les `ucr` de Structure via `structureUcrs`.
  - Il ne peut pas exister sans un `inventory.structure.json` valide.

- **Layout → Styles**
  - Layout donne un contexte (régions, structures) qui sera stylisé par `inventory.styles` et ensuite en Phase 3 (génération).

- **Layout → Actions / Effects / Tests**
  - Les régions et structures de layout servent de base pour :
    - cibler des zones fonctionnelles (ex : où se trouvent les CTA),
    - définir des scénarios de tests (zones à vérifier).

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de layout sont uniques.
- [ ] Tous les `structureUcrs` référencent des `ucr` valides dans `inventory.structure.json`.
- [ ] Si `parentLayoutUcr` est utilisé, il référence un `ucr` valide de layout.
- [ ] Si `childrenLayoutUcrs` est utilisé, il ne référence que des `ucr` valides de layout.
- [ ] Aucun cycle n’est détecté dans la hiérarchie de layout.
- [ ] Tous les champs obligatoires sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "layout",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "layout-region-main-1",
      "type": "region",
      "name": "MainContentRegion",
      "sourcePath": "src/legacy/pages/ProductListPage/index.jsx",
      "dslTags": ["layout.region"],
      "structureUcrs": ["view-root-1"],
      "metadata": {
        "role": "main",
        "order": 1
      }
    },
    {
      "ucr": "layout-grid-products-1",
      "type": "grid",
      "name": "ProductGrid",
      "sourcePath": "src/legacy/components/ProductGrid/index.jsx",
      "dslTags": ["layout.grid"],
      "structureUcrs": ["view-container-1"],
      "metadata": {
        "role": "content",
        "order": 1
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

### 6.2. Exemple invalide (commenté)

```json
{
  "domain": "layout",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "layout-region-main-1",
      "type": "region",
      "name": "MainContentRegion",
      "sourcePath": "src/legacy/pages/ProductListPage/index.jsx",
      "dslTags": ["layout.region"],
      "structureUcrs": ["view-unknown-99"],
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

- `structureUcrs` contient `view-unknown-99` qui n’existe pas dans `inventory.structure.json`.
- `validation.status` ne devrait pas être `"valid"` dans ce cas.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"layout"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de layout sont uniques  
- [ ] Tous les `structureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] La hiérarchie de layout (si présente) est sans cycle  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne jamais inventer de régions ou de structures de layout qui n’existent pas réellement dans le Legacy.
- Toujours s’appuyer sur :
  - le **DSL interne** (`layout.*`),
  - le **bridge Legacy → DSL** (`bridge-legacy-to-dsl.json`),
  - l’**inventaire de structure** (`inventory.structure.json`),
  - le **guide UCR** pour les identifiants,
  - les **guides de stack** et `project-structure.json` comme contexte de référence.
- En cas d’ambiguïté, utiliser `validation.issues` pour documenter sans casser la structure JSON.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Layout*
