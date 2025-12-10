# 🔧 Guide Inventaire — Structure (`inventory.structure`)

*(Domaine d’inventaire : **Structure** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Structure** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. La **hiérarchie des vues** (root, conteneurs, composants de présentation, fragments, slots, portails…).
2. Les **relations parent → enfant** entre ces vues.
3. Les **régions de layout** associées (si connues au niveau structurel).
4. Les **points d’accroche** vers la logique, les événements, les données (sans les détailler).

Il répond à la question :

> **“Quelle est l’ossature visuelle et structurelle de cette page, indépendamment de la logique métier et de la stack cible ?”**

Le domaine Structure ne :

- **ne décrit pas** la logique métier (cela appartient à `inventory.logic`),
- **ne détaille pas** les conditions ni les règles de visibilité (cela appartient à `inventory.conditions`),
- **ne décrit pas** les flux de données (cela appartient à `inventory.dataflows`, `inventory.services`),
- **ne génère pas** de code cible (c’est la responsabilité de la Phase 3 – Génération).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.structure.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"structure"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des nœuds de structure (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

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

> L’ordre et les noms des clés racines doivent rester **stables** dans toute la pipeline.

---

### 2.2. Schéma interne — `items[]`

Chaque élément de `items[]` représente un **nœud de vue** (view node) dans la hiérarchie de la page.

```text
items[] : StructureNode
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du nœud, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire.
  - Servira de clé de référence pour les autres inventaires (logic, layout, events, dataflows…).

- `name` : string  
  Nom logique de la vue (ex : `ProductCard`, `FilterPanel`, `RootPageLayout`).  
  - Peut être dérivé du nom du composant, du template, ou d’une convention.

- `type` : string  
  Type de nœud parmi un ensemble contrôlé, par exemple :
  - `"root"` (racine de la page),
  - `"container"`,
  - `"presentational"`,
  - `"fragment"`,
  - `"layoutWrapper"`,
  - `"slot"`,
  - `"portal"`.

- `sourcePath` : string  
  Chemin Legacy du fichier où est défini le nœud (ex : `src/legacy/components/ProductCard/index.jsx`).

- `dslTags` : array de string  
  Liste des IDs DSL pertinents pour ce nœud, typiquement dans le domaine `structure.*`, par exemple :
  - `["structure.viewNode", "structure.containerView"]`
  - `["structure.viewNode", "structure.rootView"]`

- `parentUcr` : string | null  
  - `null` pour le nœud racine de la page.
  - `ucr` du parent direct sinon.

- `childrenUcrs` : array de string  
  - Liste ordonnée des `ucr` des enfants directs dans la hiérarchie.

- `metadata` : object  
  Objet libre, mais **clé** pour transporter des informations dérivées, par exemple :
  - `displayName`,
  - `legacyComponentName`,
  - `isRoutedEntry` (booléen),
  - `hasSuspenseBoundary`, etc.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels

Selon les besoins, on peut enrichir le schéma avec des champs optionnels (documentés) :

- `layoutRegion` : string | null  
  Nom logique de la région de layout si déjà identifiable (ex : `"header"`, `"sidebar"`, `"main"`, `"footer"`).

- `hasDynamicChildren` : boolean  
  Indique si les enfants sont dynamiquement générés (ex : map sur un tableau).

- `boundEvents` : array de string  
  Liste des types d’événements principaux attachés au nœud (ex : `["click", "change"]`).  
  > Les détails seront décrits dans `inventory.events`, mais cette information peut aider à l’interprétation.

- `dataBindings` : array de string  
  Nom ou description courte des bindings de données visibles au niveau structurel (`props` clés, etc.).  
  > Les flux détaillés appartiennent aux inventaires Data.

- Tout autre champ optionnel doit être **documenté ici** avant utilisation.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` doivent être **uniques**.
- Tous les `parentUcr` doivent être :
  - soit `null` pour la racine,
  - soit un `ucr` existant dans `items[]`.
- Les `childrenUcrs[]` doivent **uniquement** contenir des `ucr` existants.
- La hiérarchie ne doit pas former de **cycle**.
- Aucune clé inconnue ne doit apparaître au niveau racine ou dans `items[]`.
- Tous les champs obligatoires doivent être présents pour chaque item.
- Le JSON doit être **strictement sérialisable** (aucun commentaire, aucune structure non JSON).

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Principalement :

- `structure.viewNode`
- `structure.rootView`
- `structure.childView`
- `structure.containerView`
- `structure.presentationalView`
- `structure.fragment`
- `structure.slot`
- `structure.portal`
- `structure.viewHierarchy`

Éventuellement, certains concepts de layout peuvent être effleurés (sans les détailler) :

- `layout.region`
- `layout.section`
- `layout.panel`

> Le mapping concret vers le Legacy est fourni par le **bridge** `bridge-legacy-to-dsl.json`.

### 3.2. Règles d’analyse

L’inventaire Structure doit :

1. Identifier la **vue racine** de la page (root view).
2. Identifier toutes les **vues significatives** de la page :
   - composants principaux,
   - blocs de layout,
   - sections structurantes.
3. Construire un **arbre parent → enfants** complet et cohérent.
4. Associer à chaque nœud :
   - son `ucr`,
   - son `name`,
   - son `type`,
   - son `sourcePath`,
   - ses `dslTags`,
   - son `parentUcr` et ses `childrenUcrs`,
   - son `metadata` minimal.

### 3.3. Restrictions

L’inventaire Structure **ne doit pas** :

- interpréter la logique métier (pas de détails sur les règles business),
- résoudre les flux de données détaillés (dataflows, services),
- détailler les conditions complexes (cela appartient à `inventory.conditions`),
- générer du code cible,
- s’encombrer de détails d’implémentation spécifiques à la stack cible.

---

## 4. 🔗 Relations avec les autres inventaires

- **Structure → Logic**
  - Fournit les `ucr` des vues auxquelles la logique est attachée (hooks, handlers…).
- **Structure → Layout**
  - Sert de base à la description plus fine des régions et grilles de layout.
- **Structure → Events**
  - Offre les points d’accroche pour les handlers d’événements.
- **Structure → Dataflows / Services**
  - Permet de relier les vues aux sources de données qui les alimentent.
- **Structure → Tests**
  - Sert de support pour cibler des vues dans les scénarios de test.

L’inventaire Structure est donc un **socle** : ses `ucr` sont réutilisés dans les autres inventaires pour référencer les mêmes entités de façon cohérente.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` sont uniques dans `items[]`.
- [ ] Tous les `parentUcr` sont soit `null`, soit un `ucr` existant.
- [ ] Tous les `childrenUcrs` référencent des `ucr` existants.
- [ ] Aucun cycle n’est détecté dans la hiérarchie.
- [ ] Tous les champs obligatoires sont présents pour chaque item.
- [ ] `validation.status` est cohérent avec `validation.issues`.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "structure",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "view-root-1",
      "name": "ProductListPageRoot",
      "type": "root",
      "sourcePath": "src/legacy/pages/ProductListPage/index.jsx",
      "dslTags": ["structure.viewNode", "structure.rootView"],
      "parentUcr": null,
      "childrenUcrs": ["view-container-1"],
      "metadata": {}
    },
    {
      "ucr": "view-container-1",
      "name": "ProductListLayout",
      "type": "container",
      "sourcePath": "src/legacy/components/ProductListLayout/index.jsx",
      "dslTags": ["structure.viewNode", "structure.containerView"],
      "parentUcr": "view-root-1",
      "childrenUcrs": [],
      "metadata": {
        "layoutRegion": "main"
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
  "domain": "structure",
  "pageName": "ProductListPage",
  "sourceEntry": "src/legacy/pages/ProductListPage/index.jsx",
  "items": [
    {
      "ucr": "view-root-1",
      "name": "Root",
      "type": "root",
      "sourcePath": "src/legacy/pages/ProductListPage/index.jsx",
      "dslTags": ["structure.viewNode", "structure.rootView"],
      "parentUcr": null,
      "childrenUcrs": ["view-unknown-99"],
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

- `childrenUcrs` contient `view-unknown-99` qui n’existe pas dans `items[]`.
- `validation.status` ne devrait pas être `"valid"` dans ce cas.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"structure"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` sont uniques  
- [ ] Tous les `parentUcr` sont valides ou `null`  
- [ ] Tous les `childrenUcrs` sont valides  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Produire un JSON strict, sans texte libre en dehors des champs prévus.
- Ne jamais inventer de vues ou de hiérarchies qui n’existent pas réellement dans le Legacy.
- S’appuyer sur :
  - le **DSL interne** (`structure.*`),
  - le **bridge Legacy → DSL** (`bridge-legacy-to-dsl.json`),
  - le **guide UCR** pour les identifiants,
  - la **spécification de structure cible** pour vérifier la cohérence globale.
- En cas d’ambiguïté ou d’erreur non bloquante, documenter dans `validation.issues` sans casser la structure JSON.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Structure*
