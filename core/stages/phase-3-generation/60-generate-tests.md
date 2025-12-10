# 🧪 Stage 60 – generate-tests
**Phase:** Phase 3 – Generation  
**Prev:** 59 – generate-i18n  
**Next:** 61 – optimize-imports  

---

## 🎯 Objective

Générer les **artefacts de tests automatisés** pour `${project.pageName}` dans la stack cible, en s’appuyant sur :

- les **mappings de Phase 2** (principalement `mapping.tests.json`, mais aussi les autres mappings pour la couverture fonctionnelle) ;
- les artefacts générés en Phase 3 (services, stores, hooks, components, pages, routing, i18n) ;
- les **stack-guides de tests** produits en Phase 0 (patterns de tests, types de tests, structure de fichiers) ;
- le **DSL + UCR + bridge legacy → DSL** comme référence fonctionnelle (use-cases, comportements attendus, erreurs, scénarios).

Le stage reste **agnostique de la stack et du framework de tests** (Jest, Vitest, Playwright, Cypress, etc.).  
Il applique uniquement les patterns décrits dans les **stack-guides de tests** pour générer :

- une ou plusieurs suites de tests pour `${project.pageName}`,
- un fichier de métadonnées de tests.

---

## 🔌 Inputs

> Toutes les entrées sont résolues à partir de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration globale

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

### 2. Artefacts Phase 0 – Stack / structure / bridge / tests

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `bridge-legacy-to-dsl.json`  
- `stack-guides/guide.stack.md`  
- `stack-guides/guide.tests.md` (obligatoire)  
- éventuellement :  
  - `stack-guides/guide.ui-components.md`  
  - `stack-guides/guide.ui-pages.md`  
  - `stack-guides/guide.services.md`  
  - `stack-guides/guide.store.md`  

Les stack-guides de tests doivent préciser :

- les **types de tests** supportés par la stack (unitaires, intégration, UI, e2e, etc.) ;
- la **structure des fichiers de tests** (sous-dossiers, nommage, extensions) ;
- les **primitives de test** attendues (abstractions pour describe/test, mocks, asserts, fixtures, etc.) ;
- comment référencer les artefacts générés (services, stores, hooks, components, pages).

### 3. Artefacts Phase 2 – Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.tests.json` (domaine principal de ce stage)  
- `mapping.actions.json` (use-cases à tester)  
- `mapping.logic.json` (scénarios de logique métier à couvrir)  
- `mapping.dataflows.json` (flux critiques à tester)  
- `mapping.services.json` (points d’intégration externes à mocker / stubber)  
- `mapping.structure.json` (pages, vues, sections à tester)  
- `mapping.layout.json` (si certains tests concernent l’assemblage visuel)  
- `mapping.routing.json` (tests de navigation, routes, guards)  
- `mapping.i18n.json` (tests de présence/usage de certaines clés de texte)  
- `mapping.effects.json` (notifications, logs, analytics à vérifier)  
- `mappings-summary.json` (readiness globale de Phase 2)  

### 4. Artefacts Phase 3 – Prérequis

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

- `services/` (stage 51)  
- `stores/` (stage 52)  
- `hooks-logic/` (stage 53)  
- `hooks-data/` (stage 54)  
- `components/atoms/` (stage 55)  
- `components/containers/` (stage 56)  
- `pages/` (stage 57)  
- `routing/` (stage 58)  
- `i18n/` (stage 59)  

Les tests générés doivent s’appuyer sur ces artefacts, pas sur le legacy.

### 5. DSL, UCR et bridge

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`  

Ils servent à :

- identifier les **UCR de tests** (`tests.*`, `actions.*`, `logic.*`, `effects.*`, etc.) à couvrir ;
- relier chaque scénario de test à un ou plusieurs UCR métier ;
- comprendre les comportements attendus (succès, erreurs, transitions d’état).

---

## 📤 Outputs

Ce stage doit générer tous les artefacts de tests pour `${project.pageName}` sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/`

Au minimum :

1. **Un ou plusieurs fichiers de tests exécutables**, dont :  

   - les noms, extensions, patterns de dossiers sont définis par `project-structure.json` + `guide.tests.md` ;  
   - les tests correspondent à des scénarios décrits dans `mapping.tests.json` et reliés aux artefacts Phase 3.

2. Un fichier de **métadonnées de tests** :

   - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/tests.meta.json`  

   Contenant au minimum :

   ```jsonc
   {
     "pageName": "${project.pageName}",
     "testSuitesCount": 0,
     "testCasesCount": 0,
     "coveredUcr": {
       "tests": [],
       "actions": [],
       "logic": [],
       "structure": []
     },
     "generatedFiles": [],
     "issues": [],
     "coverageHints": {
       "missingUcr": [],
       "weaklyCoveredUcr": []
     }
   }
   ```

Règles :

- ne modifier **aucun autre répertoire** que `phase-3-generation/tests/` ;
- si des schémas `core/schemas/tests.*.schema.json` existent, les respecter.

---

## 🧠 Actions

### Étape 1 – Charger configuration et contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître la structure de tests attendue (dossiers, naming…).  
1.3. Charger `bridge-legacy-to-dsl.json` pour identifier les UCR importants à couvrir (`tests.*`, `actions.*`, `logic.*`).  

### Étape 2 – Vérifier readiness de Phase 2

2.1. Charger `mappings-summary.json`.  
2.2. Vérifier que le domaine `tests` est présent et non bloquant, si marqué comme `requiredForPhase3`.  
2.3. Vérifier également l’état des domaines critiques : `actions`, `logic`, `dataflows`, `services`.  
2.4. En cas de manque bloquant (tests requis mais mapping absent/rejeté) :

- consigner l’erreur dans `issues` ;
- générer un `tests.meta.json` minimal ;
- terminer le stage avec **Gate ❌**.

### Étape 3 – Analyser `mapping.tests.json`

3.1. Charger `mapping.tests.json`.  
3.2. Pour chaque scénario de test, collecter :  

- UCR d’origine (`sourceInventoryRef.itemUcr`) ;  
- type de test souhaité (unit, integration, ui, e2e, contract, etc.) ;  
- cible principale (service, store, hook, component, page, route) ;  
- préconditions, steps, résultats attendus ;  
- éventuelles dépendances (mocks de services externes, fixtures de données).  

3.3. Construire en mémoire une liste de **scénarios de test abstraits** de la forme :

```jsonc
{
  "id": "TestScenario-1",
  "ucr": "tests.useCase.createCampaign-1",
  "type": "integration",
  "target": "service.CreateCampaignService",
  "preconditions": [],
  "steps": [],
  "expected": {}
}
```

(La forme exacte peut être adaptée à un schéma interne pré-défini).

### Étape 4 – Relier les scénarios aux artefacts générés

4.1. À partir des artefacts Phase 3 (services, stores, hooks, components, pages) :  

- résoudre les identifiants de cibles (ex. `CreateCampaignService` → chemin de fichier concret) ;
- identifier où se trouve la logique à tester (service vs hook vs component).  

4.2. À partir de `mapping.actions`, `mapping.logic`, `mapping.dataflows`, `mapping.effects` :  

- enrichir les scénarios avec :
  - les actions à déclencher ;
  - les flux de données à vérifier ;
  - les effets attendus (toasts, logs, navigation, etc.).  

4.3. Cette étape garantit que chaque scénario de test abstrait **pointe vers des artefacts réels** générés par les autres stages de la Phase 3.

### Étape 5 – Appliquer les patterns de `guide.tests.md`

5.1. Charger `stack-guides/guide.tests.md`.  
5.2. Identifier pour chaque **type de test** abstrait :  

- le pattern concret à utiliser (unit, integration, ui/e2e, etc.) ;
- la manière de structurer les fichiers (describe/it, suites, fixtures, etc.) ;
- la manière de référencer les artefacts cibles (imports, factories, wrappers).  

5.3. Déterminer :  

- les chemins des fichiers de tests (répertoires, suffixes, extensions) ;
- la manière de grouper les scénarios en suites (par page, par domaine, par artefact).  

Le stage ne doit **jamais inventer** ces conventions : il applique ce que définissent les stack-guides.

### Étape 6 – Générer les fichiers de tests

6.1. Pour chaque scénario de test abstrait :  

- choisir la suite de tests adéquate (nouvelle ou existante) ;
- générer un cas de test concret suivant le pattern de `guide.tests.md`.  

6.2. Pour chaque suite de tests :  

- inclure les imports nécessaires vers services/stores/hooks/components/pages ;
- configurer les mocks/stubs/fixtures selon les stack-guides ;  
- ajouter les assertions nécessaires pour vérifier :
  - résultats métier ;
  - effets (i18n, navigation, state) ;
  - erreurs attendues.  

6.3. Écrire les fichiers de tests dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/`  
- en respectant les chemins, noms, extensions et organisation définis par `project-structure.json` + `guide.tests.md`.

### Étape 7 – Générer `tests.meta.json`

7.1. Construire `testsMeta` avec au minimum :  

- `pageName` ;  
- nombre de suites (`testSuitesCount`) ;  
- nombre de cas (`testCasesCount`) ;  
- `coveredUcr` : listes d’UCR couverts par les tests (tests, actions, logic, structure) ;  
- `generatedFiles` : chemins relatifs des fichiers de tests générés ;  
- `issues` :  
  - scénarios non générés ;
  - artefacts cibles introuvables ;
  - UCR importants sans test associé.  
- `coverageHints` :  
  - `missingUcr` : UCR importants non couverts ;
  - `weaklyCoveredUcr` : UCR couverts par un seul test minimal.  

7.2. Écrire :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/tests.meta.json`

### Étape 8 – Validation et Gate

8.1. Conditions possibles pour **Gate ❌** :  

- `mapping.tests.json` requis mais absent/illisible ;  
- aucun fichier de tests généré alors qu’il existe des scénarios de tests obligatoires ;  
- des erreurs critiques empêchent la génération des fichiers.  

8.2. Conditions pour **Gate ✅** :  

- au moins une suite de tests générée ;  
- `tests.meta.json` écrit ;  
- pas d’erreur bloquante.  

---

## 🧩 Gate

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` en cas de problème bloquant (inputs indispensables manquants, génération impossible, sortie non écrite).

---

## 📦 Next

> Continuer avec `61-optimize-imports.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
