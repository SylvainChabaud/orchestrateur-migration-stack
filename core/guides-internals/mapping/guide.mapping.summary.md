# 🧭 Guide de Mapping — `mappings-summary`

*(Synthèse transversale de tous les `mapping.<domaine>.json` de la Phase 2)*

---

## 1. 🎯 Rôle de `mappings-summary`

La Phase 2 produit une série de fichiers de mapping par domaine :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`
- `mapping.hooks.json`
- `mapping.events.json`
- `mapping.dataflows.json`
- `mapping.async.json`
- `mapping.services.json`
- `mapping.routing.json`
- `mapping.effects.json`
- `mapping.actions.json`
- `mapping.tests.json`

Le stage **46 — mappings-summary** ne crée **pas** un nouveau mapping de domaine.  
Il construit une **vue globale** de la Phase 2 pour répondre à trois questions :

1. Quels `mapping.<domaine>.json` existent réellement pour `${project.pageName}` ?  
2. Dans quel état se trouvent-ils ?  
   - présent / manquant / invalide / rejeté ;
   - nombre d’items, statut de validation.
3. La Phase 2 est-elle **suffisamment prête** pour lancer la Phase 3 (génération) ?  
   - `phase2ReadyForGeneration: boolean` ;
   - `blockingDomains: string[]` ;
   - `warnings: string[]`.

C’est donc un **rapport de santé de la Phase 2** pour la page `${project.pageName}`.

---

## 2. 📦 Fichier de sortie (`mappings-summary.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mappings-summary.json`

### 2.1. Structure racine

```json
{
  "domain": "mappings-summary",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "domains": [],
  "global": {
    "phase2ReadyForGeneration": false,
    "blockingDomains": [],
    "warnings": []
  },
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs principaux :

- `domain` : `"mappings-summary"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `domains` : tableau de `DomainSummary` (voir ci-dessous)  
- `global` : agrégat global de readiness  
- `validation` : statut du fichier de synthèse lui-même

---

## 3. 🔎 Schéma d’un `DomainSummary`

Chaque entrée de `domains[]` résume **un domaine de mapping** (structure, layout, logic, …, tests).

```jsonc
{
  "domain": "string",
  "fileName": "string",
  "status": "string",
  "requiredForPhase3": true,
  "itemsCount": 0,
  "validationStatus": "unknown",
  "validationIssuesCount": 0,
  "completeness": {
    "hasItems": false,
    "hasInventories": false,
    "mappedFromInventories": false,
    "notes": ""
  },
  "dependencies": {
    "upstreamDomains": [],
    "requiredInputs": []
  }
}
```

### 3.1. Champs obligatoires

- `domain`  
  - Nom du domaine de mapping, ex. :
    - `"structure"`, `"layout"`, `"styles"`, `"i18n"`, `"config"`, `"logic"`, `"conditions"`, `"hooks"`, `"events"`, `"dataflows"`, `"async"`, `"services"`, `"routing"`, `"effects"`, `"actions"`, `"tests"`…

- `fileName`  
  - Nom du fichier de mapping attendu ou trouvé, ex. `"mapping.structure.json"`.

- `status`  
  - Statut synthétique pour ce domaine :  
    - `"present"` : fichier trouvé + JSON lisible ;  
    - `"missing"` : fichier introuvable ;  
    - `"invalid"` : fichier illisible (JSON invalide ou racine inattendue) ;  
    - `"rejected"` : fichier présent mais `validation.status === "rejected"`.

- `requiredForPhase3`  
  - `true` si ce domaine est considéré **bloquant** pour lancer la génération.  
  - Typiquement `true` pour :  
    - `structure`, `layout`, `config`, `logic`, `dataflows`, `services`, `actions`, `routing`…  
  - Les autres peuvent être optionnels ou recommandés (tests, effects, styles… selon stratégie).

- `itemsCount`  
  - Nombre d’items dans `mapping.<domaine>.json` (`root.items.length` ou `0` si absent / invalide).

- `validationStatus`  
  - Valeurs possibles :  
    - `"valid"` (si `root.validation.status === "valid"`) ;  
    - `"rejected"` ;  
    - `"pending"` ;  
    - `"unknown"` (si pas de champ `validation` ou fichier absent).

- `validationIssuesCount`  
  - `root.validation.issues.length` ou `0` si non applicable.

### 3.2. Champs de complétude

- `completeness.hasItems`  
  - `true` si `itemsCount > 0`.

- `completeness.hasInventories`  
  - `true` si le domaine apparaît comme inventorié pour `${project.pageName}` dans `inventories-summary.json`.

- `completeness.mappedFromInventories`  
  - `true` si le domaine a **à la fois** :
    - un inventaire présent et exploitable ;
    - un mapping présent et non rejeté.

- `completeness.notes`  
  - Remarques courtes (ex. : `"mapping présent mais vide"`, `"inventaire manquant"`…).

### 3.3. Dépendances (optionnel mais recommandé)

- `dependencies.upstreamDomains`  
  - Autres domaines de mapping dont celui-ci dépend logiquement, ex. :
    - pour `actions` : `["events", "dataflows", "services", "effects"]` ;  
    - pour `tests` : `["structure", "logic", "actions"]`.

- `dependencies.requiredInputs`  
  - Entrées minimales pour ce domaine (nommées fonctionnellement), ex. :
    - `["inventory.actions.json", "mapping.services.json"]`.

---

## 4. 🌐 Agrégat global `global`

Le bloc `global` décrit l’état de readiness de la Phase 2 pour `${project.pageName}` :

```jsonc
"global": {
  "phase2ReadyForGeneration": false,
  "blockingDomains": [],
  "warnings": []
}
```

- `phase2ReadyForGeneration`  
  - `true` si **aucun** domaine `requiredForPhase3 === true` n’est en `status` bloquant :  
    - `"missing"`, `"rejected"`, `"invalid"`.

- `blockingDomains`  
  - Liste des `domain` qui sont à la fois :  
    - `requiredForPhase3 === true` et  
    - `status` ∈ `["missing", "rejected", "invalid"]`.

- `warnings`  
  - Collection de messages lisibles (texte naturel) pour :  
    - domaines requis, présents mais `validationStatus !== "valid"` ;  
    - domaines optionnels manquants ;  
    - anomalies de complétude (`mapping` présent mais inventaire manquant, etc.).

Exemple :

```jsonc
"global": {
  "phase2ReadyForGeneration": false,
  "blockingDomains": ["structure", "dataflows"],
  "warnings": [
    "mapping.structure.json manquant alors que le domaine structure est requis pour la génération.",
    "mapping.dataflows.json présent mais validation.status = 'rejected'.",
    "mapping.tests.json manquant (recommandé mais non bloquant)."
  ]
}
```

---

## 5. ⚙️ Entrées requises pour `mappings-summary`

### 5.1. Configuration (obligatoire)

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

### 5.2. Artefacts Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

### 5.3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventories-summary.json` (obligatoire)

### 5.4. Mappings Phase 2 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

Le stage doit considérer **au minimum** les fichiers suivants (s’ils existent) :

- `mapping.structure.json`
- `mapping.layout.json`
- `mapping.styles.json`
- `mapping.i18n.json`
- `mapping.config.json`
- `mapping.logic.json`
- `mapping.conditions.json`
- `mapping.hooks.json`
- `mapping.events.json`
- `mapping.dataflows.json`
- `mapping.async.json`
- `mapping.services.json`
- `mapping.routing.json`
- `mapping.effects.json`
- `mapping.actions.json`
- `mapping.tests.json`

Il peut aussi intégrer d’autres `mapping.*.json` si la stack en définit davantage.

### 5.5. Guides (optionnel)

Un guide spécifique peut être fourni, par exemple :

- `${paths.core}/guides/guide.mapping.summary.md`

Il peut préciser :  
- la liste exacte des domaines `requiredForPhase3` ;  
- des règles avancées de complétude ;  
- des seuils de qualité minimum pour lancer la Phase 3.

---

## 6. 🧠 Règles d’interprétation de la synthèse

1. **Aucune relecture du Legacy**  
   - Ce stage se base exclusivement sur :
     - la configuration ;
     - `project-structure.json` ;
     - `inventories-summary.json` ;
     - les fichiers `mapping.*.json` produits par la Phase 2 ;
     - éventuellement un guide de synthèse.

2. **Tolérance aux absences de mapping**  
   - L’absence de `mapping.<domaine>.json` n’est pas une erreur technique :  
     - elle est **reflétée** dans `status = "missing"` ;  
     - si le domaine est `requiredForPhase3`, cela contribue aux `blockingDomains`.

3. **Distinction technique / fonctionnelle**  
   - Le stage ne juge pas la qualité métier des mappings ;  
   - il se limite à : présence, lisibilité JSON, statut de validation, liens avec les inventaires.

4. **Ready ou non pour la génération**  
   - `phase2ReadyForGeneration === true` uniquement si **tous** les domaines requis :  
     - ont `status === "present"` ;  
     - et `validationStatus === "valid"` (ou un statut non bloquant).

5. **Warnings lisibles pour l’utilisateur**  
   - `warnings[]` doit rester lisible par un humain (message court et explicite).

---

## 7. Exemple simplifié de `mappings-summary.json`

```json
{
  "domain": "mappings-summary",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "domains": [
    {
      "domain": "structure",
      "fileName": "mapping.structure.json",
      "status": "present",
      "requiredForPhase3": true,
      "itemsCount": 12,
      "validationStatus": "valid",
      "validationIssuesCount": 0,
      "completeness": {
        "hasItems": true,
        "hasInventories": true,
        "mappedFromInventories": true,
        "notes": ""
      },
      "dependencies": {
        "upstreamDomains": [],
        "requiredInputs": [
          "inventory.structure.json"
        ]
      }
    },
    {
      "domain": "tests",
      "fileName": "mapping.tests.json",
      "status": "missing",
      "requiredForPhase3": false,
      "itemsCount": 0,
      "validationStatus": "unknown",
      "validationIssuesCount": 0,
      "completeness": {
        "hasItems": false,
        "hasInventories": false,
        "mappedFromInventories": false,
        "notes": "Aucun mapping de tests généré pour cette page."
      },
      "dependencies": {
        "upstreamDomains": [
          "structure",
          "logic",
          "actions"
        ],
        "requiredInputs": []
      }
    }
  ],
  "global": {
    "phase2ReadyForGeneration": true,
    "blockingDomains": [],
    "warnings": [
      "mapping.tests.json manquant : recommandé mais non bloquant pour la génération."
    ]
  },
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 8. ✅ Checklist de validation

- [ ] `inventories-summary.json` présent et lisible pour `${project.pageName}`  
- [ ] `project-structure.json` accessible  
- [ ] Tous les `mapping.*.json` trouvés ont été tentés en lecture
- [ ] Chaque domaine connu possède un `DomainSummary` cohérent (`status`, `itemsCount`, etc.)  
- [ ] `global.phase2ReadyForGeneration`, `global.blockingDomains`, `global.warnings` sont cohérents avec les domaines  
- [ ] `validation.status` de `mappings-summary.json` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine transversal `mappings-summary` (Stage 46 — Phase 2 : Interprétation)*
