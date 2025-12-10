# 🔧 Guide Inventaire — Summary (`inventories-summary`)

*(Domaine d’inventaire : **Synthèse de la phase 1 – Inventaires** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Summary** a pour rôle de **coiffer l’ensemble des inventaires de la phase 1** pour une page donnée (`${project.pageName}`) et de répondre à plusieurs questions clés :

1. **Quels inventaires ont été générés ?**
2. **Dans quel état sont-ils ?** (présents / manquants, valides / rejetés, riches / pauvres)
3. **Combien d’éléments décrivent chaque domaine ?** (structure, events, logic, dataflows, …)
4. **Quels sont les principaux points de vigilance ?** (issues, incohérences, trous fonctionnels)
5. **La phase 1 est-elle suffisante pour attaquer la phase 2 (mappings) ?**

Il s’agit d’un **inventaire meta**, qui ne regarde plus directement le code, mais **les artefacts produits par les stages 10 → 25**.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventories-summary.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"inventoriesSummary"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `summaryItems` : array d’objets — un élément par domaine d’inventaire (structure, layout, styles, i18n, …)
- `globalAssessment` : object — appréciation globale de la phase 1 pour cette page
- `validation` : object — statut de validation du summary lui-même

Exemple minimal :

```json
{
  "domain": "inventoriesSummary",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "summaryItems": [],
  "globalAssessment": {
    "phase1ReadyForMappings": false,
    "blockingDomains": [],
    "warnings": []
  },
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

### 2.2. Schéma interne — `summaryItems[]`

Chaque élément de `summaryItems[]` représente la **synthèse pour un domaine d’inventaire** (*SummaryItem*).

```text
summaryItems[] : SummaryItem
```

#### 2.2.1. Champs obligatoires

- `domainId` : string  
  Identifiant du domaine d’inventaire, parmi :
  - `"structure"`,
  - `"layout"`,
  - `"styles"`,
  - `"i18n"`,
  - `"config"`,
  - `"logic"`,
  - `"conditions"`,
  - `"hooks"`,
  - `"events"`,
  - `"dataflows"`,
  - `"async"`,
  - `"services"`,
  - `"routing"`,
  - `"effects"`,
  - `"actions"`,
  - `"tests"`.

- `fileName` : string  
  Nom de fichier attendu pour ce domaine, par ex. `"inventory.structure.json"`.

- `filePath` : string  
  Chemin complet vers le fichier si présent, ex. :
  `"${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json"`  
  ou `""` / `null` si le fichier est absent (mais le champ doit exister).

- `present` : boolean  
  - `true` si le fichier d’inventaire existe et a pu être parsé,
  - `false` sinon.

- `validationStatus` : string  
  - `"valid"` si l’inventaire sous-jacent est présent et `validation.status == "valid"`,
  - `"rejected"` si l’inventaire existe avec `validation.status == "rejected"`,
  - `"missing"` si aucun fichier n’existe,
  - `"unknown"` si le fichier existe mais `validation.status` est absent ou inexploitable.

- `itemCount` : number  
  Nombre d’éléments dans `inventory.<domain>.json.items` si accessible, sinon `0`.

- `importanceLevel` : string  
  Niveau d’importance de ce domaine pour la migration, par exemple :
  - `"core"` (ex. structure, logic, events, dataflows, services, routing, actions),
  - `"behaviour"` (ex. effects, async, conditions, hooks),
  - `"integration"` (ex. config, i18n, layout, styles),
  - `"quality"` (ex. tests).

- `issuesSummary` : object  
  Résumé synthétique des problèmes remontés par l’inventaire, par exemple :
  - `issuesCount` : number,
  - `hasBlockingIssues` : boolean,
  - `highLevelNotes` : array de string (extraits de issues importantes).

- `notes` : string  
  Commentaire textuel court, ex. :
  - `"Inventaire non généré. Les mappings dépendants seront limités."`,
  - `"Présent mais avec beaucoup d’issues. À surveiller."`.

#### 2.2.2. Champs optionnels suggérés

- `dependsOnDomains` : array de string  
  Liste des autres domaines d’inventaire dont celui-ci dépend fortement (informatif).

- `usedByStages` : array de string  
  Liste d’IDs de stages de mapping/génération qui s’appuieront sur cet inventaire (informatif).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec le reste du pipeline.

---

### 2.3. Objet `globalAssessment`

L’objet `globalAssessment` fournit une vue d’ensemble de la phase 1 pour la page :

- `phase1ReadyForMappings` : boolean  
  - `true` si les inventaires jugés “core” sont présents et en état acceptable,
  - `false` sinon.

- `blockingDomains` : array de string  
  Liste des `domainId` pour lesquels la situation est jugée bloquante pour les mappings (ex. `["structure", "logic"]`).

- `warnings` : array de string  
  Liste de messages textuels expliquant :
  - les risques,
  - les manques,
  - les précautions à prendre en phase 2,
  - les inventaires à générer/améliorer en priorité.

L’IA s’appuie sur :

- les `importanceLevel`,
- les `validationStatus`,
- les `issuesSummary`  

pour remplir ces champs.

---

### 2.4. Objet `validation` (du summary lui-même)

- `status` : string — `"valid"` ou `"rejected"`
- `issues` : array d’objets ou de strings décrivant les problèmes détectés dans la construction du summary lui-même, par exemple :
  - incohérences entre `pageName` et les inventaires sous-jacents,
  - erreurs de parse JSON,
  - impossibilité de lire certains inventaires supposés présents.

Exemple :

```json
{
  "validation": {
    "status": "valid",
    "issues": [
      {
        "kind": "missingInventory",
        "domainId": "tests",
        "message": "inventory.tests.json is missing; test coverage cannot be assessed."
      }
    ]
  }
}
```

---

## 3. 🧠 Règles d’agrégation

### 3.1. Liste des domaines d’inventaire attendus

La synthèse doit toujours considérer les domaines suivants, même si leurs inventaires sont absents :

- **Socle UI & Config**  
  - `"structure"` — squelette de la page, vues, composants.  
  - `"layout"` — organisation spatiale.  
  - `"styles"` — styles et design tokens.  
  - `"i18n"` — internationalisation, traductions.  
  - `"config"` — configuration, feature flags, toggles.

- **Comportement & Interactions**  
  - `"logic"` — logique métier et UI.  
  - `"conditions"` — branches, guards, feature flags au niveau comportement.  
  - `"hooks"` — hooks natifs & custom.  
  - `"events"` — événements UI/système.

- **Données & Intégration**  
  - `"dataflows"` — flux de données, entrées/sorties.  
  - `"async"` — comportements asynchrones (loading, retry, polling…).  
  - `"services"` — services/API/clients externes.  
  - `"routing"` — navigation, routes, guards, redirections.

- **Effects & Actions & Tests**  
  - `"effects"` — side-effects, lifecycle, navigation déclenchée, tracking.  
  - `"actions"` — use cases end-to-end (événements → logique → dataflows → routing → effects).  
  - `"tests"` — couverture de test autour de la page et ses flows.

### 3.2. Importance par domaine

Recommandation d’`importanceLevel` :

- `"core"` : `"structure"`, `"logic"`, `"events"`, `"dataflows"`, `"services"`, `"routing"`, `"actions"`
- `"behaviour"` : `"effects"`, `"async"`, `"conditions"`, `"hooks"`
- `"integration"` : `"config"`, `"i18n"`, `"layout"`, `"styles"`
- `"quality"` : `"tests"`

### 3.3. Règles pour `phase1ReadyForMappings`

`globalAssessment.phase1ReadyForMappings` peut être mis à `true` si, par exemple :

- tous les domaines `"core"` sont :
  - `present == true`,
  - `validationStatus` dans `["valid", "unknown"]`,
- et aucun de ces domaines n’a `issuesSummary.hasBlockingIssues == true`.

Sinon, `phase1ReadyForMappings` doit être mis à `false` et les domaines bloquants listés dans `blockingDomains`.

Ces règles peuvent être adaptées par la suite, mais doivent rester documentées dans ce guide.

---

## 4. 🔗 Relations avec les autres inventaires

- **Summary ← Tous les inventaires 10–25**
  - le summary ne crée pas de nouvelles entités du domaine métier,
  - il se contente de **décrire l’état** des inventaires déjà produits.

- **Summary → Mappings (Phase 2)**
  - `inventories-summary.json` peut être utilisé par la phase 2 pour :
    - décider de **skipper** certains mappings si des inventaires sont manquants ou en `rejected`,
    - adapter la stratégie d’interprétation en fonction de la complétude des inputs.

- **Summary → Rapports de migration**
  - il fournit une base structurée pour générer :
    - des rapports de migration,
    - des tableaux de bord qualité,
    - des checklists de complétion avant go-live.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape 26, l’IA doit vérifier au minimum :

- [ ] Tous les `domainId` de `summaryItems` appartiennent à la liste des domaines attendus.  
- [ ] Chaque `domainId` apparaît au plus une fois dans `summaryItems`.  
- [ ] `fileName` est correct pour chaque domaine (ex. `"inventory.structure.json"`).  
- [ ] `present`, `validationStatus`, `itemCount`, `importanceLevel`, `issuesSummary`, `notes` sont renseignés.  
- [ ] `globalAssessment.phase1ReadyForMappings` est cohérent avec les `summaryItems`.  
- [ ] `validation.status` et `validation.issues` reflètent les problèmes rencontrés lors de la construction du summary.

---

## 6. 📘 Exemple de JSON

Exemple partiel de `inventories-summary.json` réaliste :

```json
{
  "domain": "inventoriesSummary",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "summaryItems": [
    {
      "domainId": "structure",
      "fileName": "inventory.structure.json",
      "filePath": ".ai-tools/ai-orchestrator-v4/workspace/projects/legacy-to-newStack-migration/pages/CampaignsDetail/phase-1-analysis/inventories/inventory.structure.json",
      "present": true,
      "validationStatus": "valid",
      "itemCount": 34,
      "importanceLevel": "core",
      "issuesSummary": {
        "issuesCount": 0,
        "hasBlockingIssues": false,
        "highLevelNotes": []
      },
      "notes": "Inventaire structure complet et valide."
    },
    {
      "domainId": "actions",
      "fileName": "inventory.actions.json",
      "filePath": ".ai-tools/ai-orchestrator-v4/workspace/projects/legacy-to-newStack-migration/pages/CampaignsDetail/phase-1-analysis/inventories/inventory.actions.json",
      "present": true,
      "validationStatus": "valid",
      "itemCount": 7,
      "importanceLevel": "core",
      "issuesSummary": {
        "issuesCount": 1,
        "hasBlockingIssues": false,
        "highLevelNotes": [
          "Une action composite agrège plusieurs flows peu testés."
        ]
      },
      "notes": "Couverture des actions principales jugée suffisante pour attaquer les mappings."
    },
    {
      "domainId": "tests",
      "fileName": "inventory.tests.json",
      "filePath": "",
      "present": false,
      "validationStatus": "missing",
      "itemCount": 0,
      "importanceLevel": "quality",
      "issuesSummary": {
        "issuesCount": 1,
        "hasBlockingIssues": false,
        "highLevelNotes": [
          "Aucun inventaire de tests détecté pour cette page."
        ]
      },
      "notes": "Les mappings pourront être générés, mais la migration devra être sécurisée par de nouveaux tests."
    }
  ],
  "globalAssessment": {
    "phase1ReadyForMappings": true,
    "blockingDomains": [],
    "warnings": [
      "Aucun inventaire de tests : prévoir un plan de test dédié pour la migration."
    ]
  },
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"inventoriesSummary"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les domaines attendus sont présents dans `summaryItems` (au moins avec `present=false`)  
- [ ] Chaque `domainId` est unique dans `summaryItems`  
- [ ] `globalAssessment.phase1ReadyForMappings` est cohérent avec les états des domaines  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les problèmes rencontrés  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Summary (Phase 1)*
