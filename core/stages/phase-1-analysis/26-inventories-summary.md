# 🧩 Stage 26 – inventories-summary
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 25 – inventory.tests  
**Next :** 30 – mapping.structure

---

## 🎯 Objectif

Produire une **synthèse globale de tous les inventaires** de la page `${project.pageName}` :

- vérifier la **présence** et la **cohérence minimale** de chaque inventaire (`structure`, `layout`, `styles`, `i18n`, `config`, `logic`, `conditions`, `hooks`, `events`, `dataflows`, `async`, `services`, `routing`, `effects`, `actions`, `tests`),
- agrèger un **résumé par domaine** (nombre d’items, criticité, principaux manques),
- produire un **inventaire de synthèse** `inventories-summary.json` qui servira de :
  - check-point qualité de la phase 1,
  - point d’entrée pour la phase 2 (mappings),
  - support pour les rapports humains (dev, PO, architecte).

L’objectif est de produire un JSON `inventories-summary.json` qui :

- indique pour chaque domaine d’inventaire :
  - s’il est présent,
  - combien d’éléments il contient,
  - son statut de validation local (si disponible),
  - un résumé de ses enjeux (issues majeures),
- fournit une **vue globale** de la qualité de l’analyse Phase 1,
- conclut si la phase 1 est **suffisante** ou non pour passer à la phase 2 (mappings).

---

## ⚙️ Inputs

> Tous les chemins sont dérivés de `project.config.yaml` via `project.*` et `paths.*`.  
> Aucun chemin absolu ne doit être utilisé.

### 1. Configuration projet (en mémoire)

Clés utilisées :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `runtime.regenerateStackGuides`
- `stack.custom`
- `gates.*`
- `stages.*`

---

### 2. Code Legacy (référence, pas de nouvelle analyse)

- `${paths.legacySource}`  
  - utilisé uniquement pour remplir `sourceEntry` et contextualiser les messages.
  - ❌ Aucun parsing approfondi du Legacy ne doit être refait dans ce stage.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Summary**
  - `${paths.core}/guides-internals/inventory/guide.inventory.summary.md`
  - Fournit :
    - l’**objectif** du domaine `inventoriesSummary`,
    - le **schéma JSON contractuel** de `inventories-summary.json`,
    - la liste des domaines d’inventaire attendus et leur rôle,
    - les règles pour agréger les validations locales.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utile pour :
    - garder un vocabulaire cohérent dans les résumés,
    - s’assurer que les références UCR évoquées dans les messages restent alignées avec les inventaires sous-jacents.

---

### 4. Bridge Legacy → DSL & structure cible (Phase 0)

- **Bridge Legacy → DSL**
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
  - Utilisation :
    - optionnelle mais utile pour contextualiser les domaines (ex. s’assurer que chaque grande famille DSL a au moins un inventaire associé).

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - vérifier rapidement si des parties importantes de la structure cible (pages, sous-vues) sont couvertes dans les inventaires.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel)
  - Utilisation :
    - en cas de besoin pour comprendre les attentes de la stack finale (facultatif dans ce stage).

---

### 5. Inventaires Phase 1 (lecture seule, coeur du stage)

Tous les inventaires sont supposés se trouver dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/`

Inventaires attendus (certains peuvent manquer, mais doivent être signalés) :

- `inventory.structure.json`          (stage 10)
- `inventory.layout.json`             (stage 11)
- `inventory.styles.json`             (stage 12)
- `inventory.i18n.json`               (stage 13)
- `inventory.config.json`             (stage 14)
- `inventory.logic.json`              (stage 15)
- `inventory.conditions.json`         (stage 16)
- `inventory.hooks.json`              (stage 17)
- `inventory.events.json`             (stage 18)
- `inventory.dataflows.json`          (stage 19)
- `inventory.async.json`              (stage 20)
- `inventory.services.json`           (stage 21)
- `inventory.routing.json`            (stage 22)
- `inventory.effects.json`            (stage 23)
- `inventory.actions.json`            (stage 24)
- `inventory.tests.json`              (stage 25)

Pour chaque fichier, le stage 26 doit :

- vérifier s’il existe,
- tenter un parse JSON,
- vérifier la présence d’un minimum de champs (`domain`, `pageName`, `items`, `validation`…),
- récupérer des indicateurs clés :
  - `domain`,
  - taille de `items`,
  - `validation.status`,
  - présence d’issues (`validation.issues`).

❗ Sans `inventory.structure.json`, la synthèse devra très probablement conclure à une **phase 1 incomplète** (à détailler dans `validation.issues`).

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire de synthèse

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventories-summary.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.summary.md`,
- `domain` doit valoir `"inventoriesSummary"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- liste tous les domaines d’inventaires (présents ou absents),
- fournit une `globalAssessment` de la phase 1 (OK pour mappings, ou non),
- JSON strictement sérialisable, sans clés non documentées.

---

## 🧠 Actions

1. **Charger le contexte global**
   - Utiliser les valeurs de configuration déjà en mémoire :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`,
     - `paths.stages`,
     - `gates.*`.

2. **Lister les inventaires attendus**
   - Construire en mémoire une liste des domaines attendus :
     - `"structure"`, `"layout"`, `"styles"`, `"i18n"`, `"config"`,
     - `"logic"`, `"conditions"`, `"hooks"`, `"events"`,
     - `"dataflows"`, `"async"`, `"services"`, `"routing"`,
     - `"effects"`, `"actions"`, `"tests"`.
   - Associer à chaque domaine :
     - le nom de fichier attendu (`inventory.<domain>.json`),
     - le rôle fonctionnel (rappel depuis `guide.inventory.summary.md`).

3. **Inspecter chaque inventaire**
   - Pour chaque domaine attendu :
     - construire le chemin du fichier :
       - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.<domain>.json`
     - vérifier s’il existe :
       - si non → marquer `present = false`, `status = "missing"`,
       - si oui :
         - tenter un parse JSON,
         - vérifier les champs de base (`domain`, `pageName`, `items`, `validation`),
         - relever :
           - `itemCount = len(items)`,
           - `inventoryDomain = json.domain`,
           - `validationStatus = json.validation.status`,
           - `issuesCount = len(json.validation.issues || [])`,
           - éventuelles anomalies flagrantes (ex. `pageName` != `${project.pageName}`).

4. **Construire les SummaryItems**
   - Pour chaque domaine, construire un `SummaryItem` (voir guide pour le schéma) :
     - `domainId` (ex. `"structure"`, `"logic"`, `"actions"`),
     - `fileName` / `filePath`,
     - `present`,
     - `itemCount`,
     - `validationStatus` (ex. `"valid"`, `"rejected"`, `"unknown"`, `"missing"`),
     - `issuesSummary` (ex. nombre d’issues + extraits importants),
     - `importanceLevel` (ex. `"core"`, `"behaviour"`, `"integration"`, `"quality"`),
     - `notes` (ex. “inventaire non généré, mappings dépendants devront être prudents”).

5. **Évaluer la complétude globale**
   - Déterminer :
     - la liste des domaines obligatoires pour démarrer les mappings (au minimum : `structure`, `logic`, `events`, `dataflows`, `services`, `routing`, `actions`),
     - quels domaines manquent ou sont en `status = "rejected"`.
   - Calculer un champ global `globalAssessment` :
     - `phase1ReadyForMappings`: booléen,
     - `blockingDomains`: liste des domaines bloquants,
     - `warnings`: liste de messages textuels.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `summaryItems[]`, `globalAssessment`, `validation`.
   - Renseigner `validation` pour le summary lui-même :
     - `status`: `"valid"` ou `"rejected"`,
     - `issues`: ex. “inventaires manquants : layout, tests…”.

7. **Validation interne**
   - Vérifier que :
     - tous les `summaryItems` ont un `domainId` unique,
     - les chemins de fichiers pour les inventaires présents sont cohérents,
     - les `validationStatus` des inventaires sont correctement remontés,
     - `globalAssessment.phase1ReadyForMappings` est cohérent avec les `blockingDomains`,
     - `validation.status` reflète l’état global du summary (mais ne bloque pas nécessairement les mappings — ce sera au Gate de décider).

8. **Écriture de l’output**
   - Écrire `inventories-summary.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventories-summary.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "26",
  "stageName": "inventories-summary",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "inventoriesScanned": true,
    "summaryItemsBuilt": true,
    "globalAssessmentComputed": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

---

## 🧩 Gate

Utiliser exactement l’un des blocs suivants :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Recommandation d’usage :

- `Gate ✅` si :
  - les inventaires structurants (au moins `structure`, `logic`, `events`, `dataflows`, `services`, `routing`, `actions`) sont présents
  - **ET** que `inventories-summary.json.globalAssessment.phase1ReadyForMappings = true`.
- `Gate ❌` si :
  - des inventaires absolument nécessaires aux mappings sont absents ou invalides,
  - ou si la vision globale est trop lacunaire pour projeter sereinement vers la phase 2.

---

## 📦 Next

> Continuer avec `30-mapping.structure.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
