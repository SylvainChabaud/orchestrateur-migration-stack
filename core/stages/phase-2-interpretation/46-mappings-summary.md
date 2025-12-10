# 🧩 Stage 46 — mappings-summary

**Phase :** Phase 2 — Interprétation  
**Précédent :** 45 — mapping.tests  
**Suivant :** Phase 3 (génération)  

---

## 🎯 Objectif

Construire le fichier `mappings-summary.json` pour la page `${project.pageName}` afin de :

- lister tous les `mapping.<domaine>.json` produits par la Phase 2 ;
- évaluer leur statut (présent / manquant / invalide / rejeté) ;
- mesurer quelques indicateurs de complétude ;
- décider si la Phase 2 est **prête** pour lancer la Phase 3 (génération).

Ce stage **ne lit pas le Legacy** et ne modifie aucun autre artefact que `mappings-summary.json`.

---

## ⚙️ Entrées requises

> Toutes les entrées sont dérivées de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration

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

### 2. Artefacts Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

### 3. Inventaires Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventories-summary.json` (obligatoire)

### 4. Mappings Phase 2 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

Le stage tente de lire les fichiers suivants (certains peuvent être absents sans faire échouer le stage) :

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

Il peut aussi inclure d’éventuels `mapping.*.json` supplémentaires s’ils sont définis pour la stack cible.

### 5. Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de synthèse des mappings**
  - `${paths.core}/guides-internals/mapping/guide.mapping.summary.md`
  - Fournit :
    - l'objectif du mapping de synthèse,
    - le schéma JSON contractuel de `mappings-summary.json`,
    - la liste exacte des domaines `requiredForPhase3`,
    - des règles avancées sur la complétude et les warnings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir la cohérence des UCR entre tous les mappings,
    - valider la traçabilité globale.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mappings-summary.json`

Structure racine attendue :

```jsonc
{
  "domain": "mappings-summary",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "domains": [],
  "global": {
    "phase2ReadyForGeneration": false,
    "blockingDomains": [],
    "warnings": []
  },
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

---

## 🧠 Actions (logique du stage)

### Étape 1 — Charger configuration et contexte minimal

1.1. Charger `core/configs/project.config.yaml` et résoudre les `${paths.*}`.  
1.2. Charger `project-structure.json`.  
1.3. Charger `inventories-summary.json` et en extraire :
- la liste des domaines inventoriés pour `${project.pageName}` ;
- l’état d’exploitabilité des inventaires.

1.4. Si `inventories-summary.json` est manquant ou illisible :
- initialiser un `mappingsSummaryRoot` minimal ;
- ajouter une issue bloquante dans `mappingsSummaryRoot.validation.issues` ;
- fixer `mappingsSummaryRoot.validation.status = "rejected"` ;
- fixer `global.phase2ReadyForGeneration = false` ;  
- écrire `mappings-summary.json` et conclure le stage en **Gate ❌**.

### Étape 2 — Initialiser `mappingsSummaryRoot`

Construire en mémoire :

```jsonc
{
  "domain": "mappings-summary",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "domains": [],
  "global": {
    "phase2ReadyForGeneration": false,
    "blockingDomains": [],
    "warnings": []
  },
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

Nommer cet objet `mappingsSummaryRoot`.

### Étape 3 — Définir la liste des domaines à évaluer

3.1. Construire en mémoire une liste de domaines connus avec leurs propriétés par défaut, par ex. :

```jsonc
[
  { "domain": "structure",  "fileName": "mapping.structure.json",  "requiredForPhase3": true  },
  { "domain": "layout",     "fileName": "mapping.layout.json",     "requiredForPhase3": true  },
  { "domain": "styles",     "fileName": "mapping.styles.json",     "requiredForPhase3": false },
  { "domain": "i18n",       "fileName": "mapping.i18n.json",       "requiredForPhase3": true  },
  { "domain": "config",     "fileName": "mapping.config.json",     "requiredForPhase3": true  },
  { "domain": "logic",      "fileName": "mapping.logic.json",      "requiredForPhase3": true  },
  { "domain": "conditions", "fileName": "mapping.conditions.json", "requiredForPhase3": false },
  { "domain": "hooks",      "fileName": "mapping.hooks.json",      "requiredForPhase3": false },
  { "domain": "events",     "fileName": "mapping.events.json",     "requiredForPhase3": true  },
  { "domain": "dataflows",  "fileName": "mapping.dataflows.json",  "requiredForPhase3": true  },
  { "domain": "async",      "fileName": "mapping.async.json",      "requiredForPhase3": false },
  { "domain": "services",   "fileName": "mapping.services.json",   "requiredForPhase3": true  },
  { "domain": "routing",    "fileName": "mapping.routing.json",    "requiredForPhase3": true  },
  { "domain": "effects",    "fileName": "mapping.effects.json",    "requiredForPhase3": false },
  { "domain": "actions",    "fileName": "mapping.actions.json",    "requiredForPhase3": true  },
  { "domain": "tests",      "fileName": "mapping.tests.json",      "requiredForPhase3": false }
]
```

3.2. Si le guide `${paths.core}/guides-internals/mapping/guide.mapping.summary.md` fournit une configuration différente, surcharger :
- `requiredForPhase3` par domaine ;
- éventuellement la liste des domaines à considérer.

### Étape 4 — Construire un `DomainSummary` pour chaque domaine

Pour chaque domaine de la liste :

4.1. Initialiser un `DomainSummary` avec les valeurs par défaut :

```jsonc
{
  "domain": "<nomDuDomaine>",
  "fileName": "<mapping.fileName>",
  "status": "missing",
  "requiredForPhase3": <bool>,
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

4.2. Vérifier la présence du fichier `mapping.<domaine>.json`.  
- Si le fichier n’existe pas :
  - laisser `status = "missing"` ;
  - compléter `completeness.notes` si besoin (ex. `"mapping absent"`).

4.3. Si le fichier existe, tenter de le parser :

- Si le JSON est illisible ou ne contient pas une structure racine attendue :
  - `status = "invalid"` ;
  - `validationStatus = "unknown"` ;
  - `completeness.notes` = `"JSON invalide ou structure inattendue"`.

- Sinon :
  - `status = "present"` ;
  - `itemsCount = root.items.length` (ou `0` si pas de champ `items`) ;
  - si `root.validation` existe :
    - `validationStatus = root.validation.status` ;
    - `validationIssuesCount = (root.validation.issues || []).length` ;
  - sinon :
    - `validationStatus = "unknown"`.

4.4. Compléter `completeness` à partir de `inventories-summary.json` :

- `completeness.hasItems = (itemsCount > 0)` ;
- `completeness.hasInventories = true` si le domaine apparaît comme inventorié pour `${project.pageName}` ;
- `completeness.mappedFromInventories = true` si :
  - `completeness.hasInventories === true` ;
  - ET `status === "present"` ;
  - ET `validationStatus !== "rejected"`.

4.5. Renseigner éventuellement `dependencies.upstreamDomains` et `dependencies.requiredInputs` selon des règles simples ou un guide (ex. : `actions` dépend de `events`, `dataflows`, `services`, `effects`).

4.6. Ajouter ce `DomainSummary` à `mappingsSummaryRoot.domains[]`.

### Étape 5 — Calculer les agrégats globaux

5.1. Initialiser :  
- `blockingDomains = []` ;  
- `warnings = []`.

5.2. Pour chaque `DomainSummary` :

- Si `requiredForPhase3 === true` ET `status ∈ ["missing", "invalid", "rejected"]` → ajouter `domain` à `blockingDomains`.  
- Si `status === "present"` et `validationStatus !== "valid"` → ajouter un warning descriptif.  
- Si `requiredForPhase3 === false` et `status === "missing"` → éventuellement ajouter un warning informatif.

5.3. Calculer `phase2ReadyForGeneration` :

- `true` si `blockingDomains.length === 0` ;
- `false` sinon.

5.4. Affecter dans `mappingsSummaryRoot.global` :

```jsonc
{
  "phase2ReadyForGeneration": <bool>,
  "blockingDomains": <blockingDomains[]>,
  "warnings": <warnings[]>
}
```

### Étape 6 — Validation interne

6.1. Vérifier :  
- `mappingsSummaryRoot.domain === "mappings-summary"` ;  
- `mappingsSummaryRoot.pageName === project.pageName` ;  
- aucun doublon de `domain` dans `mappingsSummaryRoot.domains[]`.

6.2. Si problème bloquant (ex. : pas de `domains[]` construit) :  
- ajouter une issue dans `mappingsSummaryRoot.validation.issues` ;  
- fixer `mappingsSummaryRoot.validation.status = "rejected"`.

6.3. Sinon :  
- fixer `mappingsSummaryRoot.validation.status = "valid"` ;  
- s’assurer que `validation.issues` est un tableau (éventuellement vide).

### Étape 7 — Écriture du fichier de sortie

7.1. Sérialiser `mappingsSummaryRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mappings-summary.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier dans le workspace.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer dans sa réponse (non écrit sur disque) :

```json
{
  "stageId": "46",
  "stageName": "mappings-summary",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "outputsWritten": true
  },
  "global": {
    "phase2ReadyForGeneration": true,
    "blockingDomains": [],
    "warningsCount": 0
  }
}
```

- `inputsAvailable` = `false` si la configuration ou `inventories-summary.json` sont manquants.  
- `outputsWritten` = `false` si le fichier `mappings-summary.json` n’a pas pu être écrit.  
- `global.phase2ReadyForGeneration`, `global.blockingDomains`, `global.warningsCount` doivent refléter les valeurs du fichier écrit.

---

## 🧩 Gate

Fin de fichier, écrire **exactement l’un** des blocs :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` uniquement si le stage n’a pas pu produire une synthèse exploitable (configuration ou `inventories-summary.json` manquants, impossibilité d’écrire la sortie, etc.).

Dans tous les autres cas, même si `phase2ReadyForGeneration === false`, le stage doit rester en **Gate ✅** : il a rempli son rôle de **diagnostic**.

---

© 2025 — ai-orchestrator-v4
