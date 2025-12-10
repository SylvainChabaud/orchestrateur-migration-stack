# 🔧 Guide Inventaire — Conditions (`inventory.conditions`)

*(Domaine d’inventaire : **Conditions** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Conditions** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **conditions** qui pilotent l’affichage ou le comportement (if/else, ternaires, guards…).
2. Les **conditions basées sur la configuration** (feature flags, settings),
3. Les **conditions basées sur les données** (présence, nombre, statut),
4. Les **conditions de garde** (guards) avant exécution d’une logique ou d’un effet,
5. Les **conditions liées aux permissions / rôles**.

Il répond à la question :

> **“Quels sont les points de décision (conditions) importants dans cette page, sur quoi reposent-ils et quelles vues/logiques impactent-ils ?”**

Ce domaine ne :

- ne décrit pas la logique métier elle-même (→ `inventory.logic`),
- ne décrit pas les événements (→ `inventory.events`),
- ne décrit pas les effets / side-effects (→ `inventory.effects`),
- ne modélise pas les flux de données (→ `inventory.dataflows`, `inventory.services`).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.conditions.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"conditions"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des conditions significatives (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "conditions",
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

Chaque élément de `items[]` représente une **condition significative** (*ConditionItem*).

```text
items[] : ConditionItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de la condition, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Conditions.

- `kind` : string  
  Type de condition parmi un ensemble contrôlé, par exemple :
  - `"branch"` (condition générale if/else),
  - `"guard"` (precondition avant logique/effet),
  - `"featureFlagCheck"`,
  - `"configCheck"`,
  - `"dataPresence"`,
  - `"permissionCheck"`,
  - `"boundaryCheck"` (plage, limite, etc.).

- `name` : string  
  Nom logique de la condition, par exemple :
  - `"showAdvancedBlockIfFlagEnabled"`,
  - `"redirectIfUserNotAuthorized"`,
  - `"showEmptyStateIfNoCampaigns"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où se trouve la condition.

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) sur lesquels cette condition a un impact direct (affichage, comportement).

- `conditionSummary` : object  
  Résumé structuré de la condition, par exemple :
  - `expressionType`: `"if" | "ternary" | "guard" | "switch" | "logicalAnd" | "coalesce"`,  
  - `inputs`: description des sources utilisées dans la condition (états, props, données, config),
  - `outcomes`: description des principaux cas (`"true"`, `"false"`, cas de switch),
  - `description`: phrase courte (“Affiche la section X seulement si le flag Y est activé et qu’il y a au moins une campagne.”).

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen (true si erreur grave potentielle en cas de bug),
  - `isGuard`: booléen (true si c’est un guard au sens strict),
  - `isDuplicated`: booléen (true si condition dupliquée dans plusieurs endroits),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) utilisés dans la condition.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de `LogicItem` (issus de `inventory.logic.json`) auxquels la condition est liée (par ex. guards avant une logique).

- `relatedDataflowIds` : array de string  
  Identifiants logiques d’appels data/services conditionnés par cette condition.

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) qui déclenchent cette condition ou sont conditionnés par elle.

- `severity` : string  
  Impact potentiel d’une mauvaise évaluation de cette condition (`"low"`, `"medium"`, `"high"`).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de condition doivent être **uniques** dans `inventory.conditions.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- `relatedLogicUcrs` (si présent) ne doit contenir que des `ucr` existant dans `inventory.logic.json`.
- `relatedConfigNames` (si présent) doit référencer des `configName` définis dans `inventory.config.json`.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `condition.*` du DSL peut inclure par exemple :

- `condition.featureFlag`
- `condition.branchIf`
- `condition.guard`
- `condition.dataPresence`
- `condition.permission`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître ces concepts dans le code.  
Si certaines entrées sont manquantes, l’IA doit :

- s’appuyer sur des patterns procéduraux génériques (if, ternaires, `&&` dans le JSX, etc.),
- documenter la limitation dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Conditions doit :

1. Parcourir le code à partir de `${paths.legacySource}` afin de :
   - identifier les **conditions structurantes** (affichage/masquage de blocs, guards critiques),
   - distinguer les conditions mineures (cosmétiques) des conditions majeures,
   - repérer les conditions récurrentes (même pattern/flag utilisé partout).
2. Pour chaque condition significative :
   - déterminer le `kind` adéquat (branch, guard, featureFlagCheck, etc.),
   - identifier les **inputs** principaux (états, props, config, données),
   - décrire les **outcomes** logiques importants (true/false, cas d’un switch),
   - associer les `targetStructureUcrs` correspondants,
   - lier éventuellement la condition à des paramètres de config ou à de la logique en amont/aval.

### 3.3. Restrictions

L’inventaire Conditions **ne doit pas** :

- réimplémenter l’intégralité des expressions conditionnelles au niveau syntaxique,
- absorber le contenu complet de la logique métier ou des effets,
- se transformer en inventaire de dataflows ou de config (ces aspects sont liés mais décrits dans leurs domaines respectifs).

En cas de conditions très complexes ou imbriquées, l’IA doit :

- en donner une **synthèse** dans `conditionSummary.description`,
- indiquer dans `metadata.notes` qu’une refactorisation est recommandée.

---

## 4. 🔗 Relations avec les autres inventaires

- **Conditions ← Structure**
  - Utilise les `ucr` de Structure pour documenter quelles vues sont impactées.

- **Conditions ↔ Logic**
  - Beaucoup de conditions sont placées en amont ou autour de la logique.  
    Références possibles via `relatedLogicUcrs`.

- **Conditions ↔ Config**
  - Les conditions basées sur des flags/settings référencent les `configName` correspondants.

- **Conditions ↔ Dataflows / Services**
  - Certaines conditions déterminent si une requête est exécutée ou non (référencées dans `relatedDataflowIds`).

- **Conditions ↔ Events / Effects**
  - Les conditions peuvent décider d’exécuter ou non certains effets / handlers, mais ces domaines sont décrits dans leurs inventaires respectifs.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de condition sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `conditionSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "conditions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "cond-show-advanced-block-if-flag-enabled-1",
      "kind": "featureFlagCheck",
      "name": "showAdvancedBlockIfFlagEnabled",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-container-advancedBlock-1"],
      "conditionSummary": {
        "expressionType": "ternary",
        "inputs": ["feature flag ENABLE_ADVANCED_PROMOBOOST"],
        "outcomes": [
          "true → affichage du bloc 'Promotions avancées'",
          "false → bloc non rendu"
        ],
        "description": "Affiche la section 'Promotions avancées' uniquement si le flag ENABLE_ADVANCED_PROMOBOOST est actif."
      },
      "metadata": {
        "isCritical": false,
        "isGuard": false,
        "isDuplicated": false
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
  "domain": "conditions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "cond-show-advanced-block-if-flag-enabled-1",
      "kind": "featureFlagCheck",
      "name": "showAdvancedBlockIfFlagEnabled",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-unknown-99"],
      "conditionSummary": {
        "expressionType": "ternary",
        "inputs": ["feature flag ENABLE_ADVANCED_PROMOBOOST"],
        "outcomes": [
          "true → affichage du bloc 'Promotions avancées'",
          "false → bloc non rendu"
        ],
        "description": "Affiche la section 'Promotions avancées' uniquement si le flag ENABLE_ADVANCED_PROMOBOOST est actif."
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

- [ ] `domain` est `"conditions"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de condition sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les **points de décision** clés, pas sur chaque `if` trivial.
- Toujours relier les conditions à :
  - au moins une vue (`targetStructureUcrs`),
  - éventuellement une logique (`relatedLogicUcrs`) ou une config (`relatedConfigNames`).
- Documenter les conditions particulièrement complexes ou imbriquées dans `metadata.notes` et/ou `validation.issues`.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Conditions*
