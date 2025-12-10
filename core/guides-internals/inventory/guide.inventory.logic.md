# 🔧 Guide Inventaire — Logic (`inventory.logic`)

*(Domaine d’inventaire : **Logique** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Logic** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **états locaux** (state propre au composant ou au module).
2. Les **états dérivés** (valeurs calculées à partir d’autres états ou props).
3. Les **règles métier** (business rules) exprimées dans le code.
4. Les **fonctions de calcul / transformation** purement logiques.
5. Les **cycles de vie de la logique** (initialisation/reset de la logique, pas des effets).

Il répond à la question :

> **“Quelle est la logique pure qui pilote cette page (hors conditions, événements, effets et dataflows) ?”**

Ce domaine ne couvre pas :

- les conditions (if/else, guards → `inventory.conditions`),
- les hooks (React / custom → `inventory.hooks`),
- les événements (`inventory.events`),
- les effets / side-effects (`inventory.effects`),
- les appels aux APIs, queries, mutations (`inventory.dataflows`, `inventory.services`).  

Il se concentre sur le **cœur logique**, c’est-à-dire ce qui pourrait être, en théorie, testé en pur code sans IO.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.logic.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"logic"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des unités de logique (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "logic",
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

Chaque élément de `items[]` représente une **unité logique** (*LogicItem*).

```text
items[] : LogicItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’unité logique, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Logic.

- `kind` : string  
  Catégorie de logique parmi un ensemble contrôlé, par exemple :
  - `"localState"`,
  - `"derivedState"`,
  - `"businessRule"`,
  - `"formValidation"`,
  - `"computation"` (calcul pur),
  - `"lifecycleLogic"` (initialisation/reset de la logique).  
  *(Les effets / hooks / conditions sont EXCLUS de ce domaine.)*

- `name` : string  
  Nom logique de l’unité, par exemple :
  - `"selectedRowsState"`,
  - `"filteredCampaignsDerivedState"`,
  - `"campaignBudgetValidationRule"`,
  - `"computeKpis"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où se trouve la logique (composant, hook, module utilitaire…).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) impactés directement par cette logique, par exemple :
  - vues qui consomment l’état,
  - conteneurs qui portent la logique métier.

- `logicSummary` : object  
  Résumé structuré du rôle de l’unité logique, par exemple :
  - `inputs`: description des données/états en entrée,
  - `outputs`: description des données/états en sortie,
  - `description`: courte phrase fonctionnelle (“filtre les campagnes selon les filtres actifs et le statut”).  
  Ce champ doit exister (même si certains sous-champs sont vides).

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isPure`: booléen (true si la fonction est pure, sans effets),
  - `isSharedAcrossPages`: booléen,
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui influencent directement cette logique.

- `relatedDataflowIds` : array de string  
  Identifiants (logiques) pour des dataflows / services associés à cette logique (sans les détailler).

- `dependsOnLogicUcrs` : array de string  
  Liste des `ucr` d’autres `LogicItem` dont dépend celui-ci (chaîne de calcul).

- `complexity` : string  
  Indication qualitative de complexité (`"low"`, `"medium"`, `"high"`).

- `riskLevel` : string  
  Impact potentiel d’une régression sur cette logique (`"low"`, `"medium"`, `"high"`).

Tout champ optionnel utilisé doit être **documenté** ici.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de logique doivent être **uniques** dans `inventory.logic.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- `dependsOnLogicUcrs` (si présent) ne doit contenir que des `ucr` existant dans `items[]`.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `logic.*` du DSL peut inclure par exemple :

- `logic.localState`
- `logic.derivedState`
- `logic.businessRule`
- `logic.formValidation`
- `logic.computation`
- `logic.domainModel`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître ces concepts dans le code Legacy.  
Si certaines entrées sont manquantes dans le bridge, l’IA doit :

- s’appuyer sur des heuristiques génériques (hooks de state, fonctions utilitaires, etc.),
- documenter la limitation dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Logic doit :

1. Parcourir le code à partir de `${paths.legacySource}` afin de :
   - identifier les **états locaux** (ex. `useState`, `useReducer`, `this.state`),
   - identifier les **états dérivés** (ex. `useMemo`, fonctions de mapping/filtrage),
   - isoler les **règles métier** (fonctions qui prennent des données métier en entrée et retournent des décisions/résultats),
   - repérer les **fonctions de validation** de formulaires orientées métier,
   - repérer les **fonctions de calcul** répétables (score, agrégations, KPIs…).
2. Éviter d’absorber :
   - les **conditions** (if/else) — uniquement référencées via un résumé éventuel si indispensable,
   - les **effets** (APIs, timers, manipulations DOM) — qui relèvent d’`inventory.effects`,
   - la logique d’**attachement aux événements** (click handlers, submit, etc.).
3. Regrouper les éléments en unités cohérentes (`LogicItem`), avec :
   - un `kind` clair,
   - un lien vers les vues via `targetStructureUcrs`,
   - un `logicSummary` intelligible.

### 3.3. Restrictions

L’inventaire Logic **ne doit pas** :

- décrire les flux de données asynchrones (requêtes, subscriptions → Dataflows/Services),
- détailler la structure des hooks ou des effets,
- mélanger les responsabilités : un item doit rester concentré sur une logique.

En cas de mélange inévitable dans le Legacy (logique + effets), l’IA doit :

- extraire autant que possible la partie logique en la décrivant ici,
- noter dans `metadata.notes` ou `validation.issues` que la logique est couplée à des effets.

---

## 4. 🔗 Relations avec les autres inventaires

- **Logic ← Structure**
  - Utilise les `ucr` de Structure pour indiquer quelles vues sont impactées par la logique.

- **Logic ↔ Config**
  - De nombreuses règles métier sont paramétrées par des flags / settings.  
    Référence possible via `relatedConfigNames`.

- **Logic ↔ Dataflows / Services**
  - La logique peut préparer des données pour des appels API ou exploiter des résultats de dataflows, sans les décrire en détail ici.

- **Logic ↔ Conditions / Events / Effects**
  - Ces domaines décrivent comment la logique est déclenchée, conditionnée ou produisant des effets.  
    Ici, on ne documente que le **contenu logique**.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de logique sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `logicSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "logic",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "logic-localState-selectedRows-1",
      "kind": "localState",
      "name": "selectedRowsState",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-table-campaigns-1"],
      "logicSummary": {
        "inputs": ["liste de campagnes affichées"],
        "outputs": ["liste des identifiants de campagnes sélectionnées"],
        "description": "Stocke et met à jour la sélection de lignes dans le tableau des campagnes."
      },
      "metadata": {
        "isPure": false,
        "isSharedAcrossPages": false
      }
    },
    {
      "ucr": "logic-derivedState-filteredCampaigns-1",
      "kind": "derivedState",
      "name": "filteredCampaignsDerivedState",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/hooks/useFilteredCampaigns.js",
      "targetStructureUcrs": ["view-table-campaigns-1"],
      "logicSummary": {
        "inputs": ["liste de campagnes", "filtres actifs"],
        "outputs": ["liste de campagnes filtrées"],
        "description": "Applique les filtres actifs à la liste de campagnes pour produire l’ensemble filtré."
      },
      "metadata": {
        "isPure": true,
        "isSharedAcrossPages": true
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
  "domain": "logic",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "logic-localState-selectedRows-1",
      "kind": "localState",
      "name": "selectedRowsState",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-unknown-99"],
      "logicSummary": {
        "inputs": ["liste de campagnes affichées"],
        "outputs": ["liste des identifiants de campagnes sélectionnées"],
        "description": "Stocke et met à jour la sélection de lignes dans le tableau des campagnes."
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

Problèmes :

- `targetStructureUcrs` contient `view-unknown-99` qui n’existe pas dans `inventory.structure.json`.
- `validation.status` ne devrait pas être `"valid"`.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"logic"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de logique sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Isoler autant que possible la logique pure, même si le Legacy mélange effets/événements et logique.
- Utiliser :
  - `inventory.structure.json` pour ancrer la logique aux vues,
  - le bridge `logic.*` pour reconnaître les patterns typiques,
  - les autres inventaires (config, dataflows, etc.) comme contexte, sans les dupliquer.
- En cas de doute sur la catégorisation (logic vs condition vs effet), documenter l’ambiguïté dans `metadata.notes`.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Logic*
