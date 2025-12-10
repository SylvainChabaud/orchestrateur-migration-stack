# 🧩 Stage 25 – inventory.tests
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 24 – inventory.actions  
**Next :** 26 – inventories-summary

---

## 🎯 Objectif

Construire l’**inventaire des tests** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- l’ensemble des inventaires comportementaux : Events, Logic, Dataflows, Async, Services, Routing, Effects, Actions,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `test.*`.

L’objectif est de produire un JSON `inventory.tests.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **types de tests** présents (unitaires, composants, intégration, end-to-end, visuels, contract tests…),
- les **cibles de test** (vues, hooks, services, actions, flows),
- la **couverture fonctionnelle** par rapport aux inventaires :
  - structure,
  - events / logic / dataflows / async / services / routing / effects / actions,
- les **gaps majeurs** de couverture,
- les **tests critiques** à préserver/rejouer en priorité dans la nouvelle stack.

Cet inventaire ne :

- ne remplace pas un rapport de coverage outillé (Jest, Cypress, etc.),
- ne recopie pas le contenu de chaque test fichier par fichier,
- ne vise pas à analyser toutes les assertions en détail.  

Il fournit une **vision structurée de la couverture de test** autour de `${project.pageName}`.

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

### 2. Code Legacy (lecture seule)

- `${paths.legacySource}`  
  - Fichier d’entrée principal de la page Legacy.
  - Le stage peut suivre les imports vers :
    - les composants,
    - les hooks custom,
    - les services et orchestrateurs liés.

- **Fichiers de tests associés** (détection heuristique) :
  - fichiers voisins (`*.test.*`, `*.spec.*`) dans les mêmes dossiers,
  - fichiers de tests plus globaux ciblant la page ou ses services,
  - fichiers de tests e2e/functional (ex. `cypress/e2e/**`, `playwright/tests/**`, etc.) si la page est nommée dans leurs scénarios.

> Le stage ne copie jamais les fichiers de tests dans `${paths.workspace}` : il les lit/inspecte uniquement.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Tests**
  - `${paths.core}/guides-internals/inventory/guide.inventory.tests.md`
  - Fournit :
    - l’**objectif** du domaine Tests,
    - le **schéma JSON contractuel** de `inventory.tests.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec les autres inventaires),
    - les règles de classification des tests.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les tests respectent le contrat global,
    - garantir que les références vers d’autres inventaires sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les concepts DSL `test.*` si présents :
  - `test.unit`,
  - `test.component`,
  - `test.integration`,
  - `test.e2e`,
  - `test.contract`,
  - `test.visual`,
- et/ou les patterns de nommage et d’outillage (Jest, Testing Library, Cypress, Playwright…).

> Si le bridge ne définit pas explicitement `test.*`, l’IA s’appuie sur les conventions de tests (suffixes `.test/.spec`, dossiers `__tests__`, outils) et documente les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack finale souhaite organiser les tests (unitaires, composants, e2e…),
    - anticiper quelles catégories d’actions / flows devront être re-testées.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions souhaitées pour les tests dans la stack finale,
    - préparer la future translation inventory.tests → mapping.tests → generate-tests.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - relier les tests aux vues/composants testés.

- **Inventaires comportementaux recommandés**
  - `inventory.events.json` (Stage 18)
  - `inventory.logic.json` (Stage 15)
  - `inventory.dataflows.json` (Stage 19)
  - `inventory.async.json` (Stage 20)
  - `inventory.services.json` (Stage 21)
  - `inventory.routing.json` (Stage 22)
  - `inventory.effects.json` (Stage 23)
  - `inventory.actions.json` (Stage 24)

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
Si certains inventaires comportementaux sont absents, l’IA doit quand même construire `inventory.tests.json` mais documenter les limites dans `validation.issues`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.tests.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.tests.md`,
- `domain` doit valoir `"tests"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (events, logic, dataflows, async, services, routing, effects, actions, config…) doivent être cohérentes,
- JSON strictement sérialisable, sans clés non documentées.

---

## 🧠 Actions

1. **Charger le contexte global**
   - Utiliser les valeurs de configuration déjà en mémoire :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`,
     - `paths.stages`,
     - `gates.*`.

2. **Charger les artefacts nécessaires**
   - Lire :
     - `${paths.workspace}/projects/${project.name}/stack/project-structure.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.actions.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.tests.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Identifier l’univers de test autour de la page**
   - À partir de `${paths.legacySource}` :
     - chercher les fichiers de test voisins (même dossier / arborescence proche),
     - rechercher les tests nommant explicitement `${project.pageName}` ou des composants/flows associés,
     - rechercher les tests e2e/functional mentionnant l’URL/route de la page, ou des libellés UI caractéristiques.
   - Catégoriser les tests par :
     - **type** : unit, component, integration, e2e, contract, visual, autre,
     - **cible** : composant, hook, service, action, flow complet.

4. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - relier les tests aux vues/composants testés.
   - À partir des autres inventaires (events, logic, dataflows, async, services, routing, effects, actions) :
     - construire des mappings “nom / identifiant dans le code” → `ucr`,
     - permettre de rattacher les tests à des `ucr` déjà définis.

5. **Construire les items de tests**
   - Créer un `TestItem` par **scénario de test significatif**, en tenant compte :
     - des fichiers de tests individuels,
     - des suites/scénarios e2e,
     - des tests contractuels ou de snapshot importants.
   - Pour chaque item (voir guide pour le schéma) :
     - définir `kind` (unit, component, integration, e2e, contract, visual…),
     - définir `testSummary` :
       - `testName`, `tooling`, `scope`, `mainAssertions`, `targetDomain` (ui, data, routing, actions…), `description`,
     - associer des `targetStructureUcrs` (vues/composants cibles),
     - relier le test aux events, logic, dataflows, async, services, routing, effects, actions, config via les champs prévus.

6. **Évaluer la couverture**
   - Pour chaque domaine (structure, events, logic, dataflows, async, services, routing, effects, actions) :
     - repérer les `ucr` couverts par au moins un test,
     - repérer les `ucr` **non couverts** mais jugés critiques (ex. `metadata.severity = "high"` côté actions).
   - Résumer cette évaluation dans `validation.issues` ou un champ dédié si prévu par le schéma.

7. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

8. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (events, logic, dataflows, async, services, routing, effects, actions, config…),
     - les tests critiques sont bien identifiés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]` (y compris pour signaler les gaps de couverture).

9. **Écriture de l’output**
   - Écrire `inventory.tests.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.tests.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "25",
  "stageName": "inventory.tests",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "bridgeLoaded": true,
    "structureInventoryLoaded": true,
    "legacyParsed": true,
    "testsDetected": true,
    "itemsBuilt": true,
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

- `Gate ✅` si `inventory.tests.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `26-inventories-summary.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
