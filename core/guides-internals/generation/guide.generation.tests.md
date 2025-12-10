# 🔧 Guide Génération — Tests

*(Domaine de génération : **tests** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de génération

Le domaine **tests** décrit comment générer des **tests automatisés** pour `${project.pageName}` à partir :

- des **UCR métier** (use-cases, workflows, effets, validations) ;
- du **bridge legacy → DSL**, qui explicite les comportements attendus ;
- des **mappings de tests de Phase 2** (`mapping.tests.json`) qui précisent quoi tester et comment ;  
- des artefacts de génération de Phase 3 (services, stores, hooks, components, pages, routing, i18n) ;
- des **stack-guides de tests**, qui définissent la manière concrète d’écrire les tests dans la stack cible.

Objectif final : obtenir une base de tests **cohérente, traçable et extensible**, alignée sur les comportements décrits par le DSL, et directement exécutable par l’outillage de tests de la stack (décrit dans les stack-guides).

Ce guide reste **agnostique** : il ne nomme ni framework de tests, ni runner spécifique. Il décrit uniquement les **types de tests**, les **lignes de force**, et la **structure conceptuelle** à générer.

---

## 2. 🔌 Entrées du domaine de génération

### 2.1. Configuration & chemins

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `stack.custom`

Les tests sont générés sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/`

Les sous-dossiers exacts, conventions de nommage et extensions sont définis par :

- `project-structure.json` ;
- `stack-guides/guide.tests.md`.

### 2.2. Artefacts Phase 0 — Stack, structure, bridge, tests

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `bridge-legacy-to-dsl.json`
- `stack-guides/guide.stack.md`
- `stack-guides/guide.tests.md`
- éventuellement :
  - `stack-guides/guide.services.md`
  - `stack-guides/guide.store.md`
  - `stack-guides/guide.ui-components.md`
  - `stack-guides/guide.ui-pages.md`

Le guide de tests doit préciser notamment :

- les **types de tests** utilisés (unitaires, intégration, UI, e2e, contract, snapshot, etc.) ;
- la **structure des fichiers** (un fichier par artefact, par fonctionnalité, par type de test…) ;
- les conventions pour :  
  - déclarer des suites de tests ;  
  - déclarer des cas de test ;  
  - gérer les mocks et fixtures ;  
  - faire des assertions.

### 2.3. Artefacts Phase 2 — Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.tests.json` (scénarios de tests abstraits) ;
- `mapping.actions.json` (use-cases métiers à tester) ;
- `mapping.logic.json` (logique métier, state transitions) ;
- `mapping.dataflows.json` (flux de données critiques) ;
- `mapping.services.json` (services externes à mocker) ;
- `mapping.structure.json` (pages, vues, sections impliquées) ;
- `mapping.routing.json` (navigation à tester) ;
- `mapping.i18n.json` (présence/qualité des messages) ;
- `mapping.effects.json` (toasts, logs, analytics) ;
- `mappings-summary.json` (validation globale de la Phase 2).

### 2.4. Artefacts Phase 3 — Cibles de tests

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

- `services/`
- `stores/`
- `hooks-logic/`
- `hooks-data/`
- `components/atoms/`
- `components/containers/`
- `pages/`
- `routing/`
- `i18n/`

Ces artefacts sont les **cibles concrètes** des tests générés.  
Les tests doivent éviter de cibler directement le legacy.

### 2.5. DSL + UCR + bridge

- `Spec Dsl Orchestrator`
- `Spec Ucr Orchestrator`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Ils permettent de :

- recenser les **UCR critiques** à couvrir (use-cases, workflows, règles, erreurs, effets) ;
- prioriser certains tests (ex. flows métier majeurs vs flows secondaires) ;
- vérifier la **couverture fonctionnelle** réelle de la page.

---

## 3. 🧠 Règles générales de génération des tests

### 3.1. Tests comme miroir des UCR

Les tests doivent être construits comme un **miroir des UCR importants** :

- `actions.*` → tests de use-cases ;  
- `logic.*` → tests de transitions d’état ;  
- `dataflows.*` → tests de flux de données (inputs/outputs) ;  
- `effects.*` → tests de notifications, logs, side-effects ;  
- `tests.*` → scénarios explicitement prévus pour la page.

Les scénarios de `mapping.tests.json` doivent être **alignés** avec ces UCR.

### 3.2. Stratification des types de tests

Le guide de tests doit encourager une **stratification** claire :

- **Tests unitaires** pour :  
  - services métier isolés ;  
  - fonctions utilitaires ;  
  - hooks sans UI (logic-only).  

- **Tests d’intégration** pour :  
  - services + stores + hooks ;  
  - interactions simples avec des composants containers.  

- **Tests UI / composant** pour :  
  - containers / pages critiques ;  
  - scénarios d’interaction (clic, saisie, navigation locale).  

- **Tests e2e (optionnel, si stack le prévoit)** pour :  
  - parcours utilisateur end-to-end (depuis le routing).  

Le mapping peut indiquer explicitement le type de test à privilégier pour chacun.

### 3.3. Couverture minimale attendue

Le générateur doit viser au minimum :

- chaque **use-case majeur** (`actions.*` de haut niveau) couvert par au moins un test ;  
- chaque **workflow critique** (ex. multi-étapes) couvert par au moins un test d’intégration ou e2e ;  
- les **principaux états d’erreur** couverts (validations, erreurs serveur).

Les UCR non couverts doivent être recensés dans `tests.meta.json` (champ `coverageHints.missingUcr`).

### 3.4. Usage strict des stack-guides de tests

Ce guide ne définit pas :

- la fonction équivalente à `describe` ou `it` ;  
- l’API d’assertion ;  
- la manière de lancer les tests.

Tout cela est défini dans `guide.tests.md`.  
Le domaine de génération **tests** doit simplement :

- structurer les suites et cas de test ;  
- brancher les artefacts cibles ;  
- injecter les scénarios et assertions logiques.

---

## 4. 🧬 Patterns de tests (conceptuels)

### 4.1. Test unitaire de service

Cible : un service métier généré par le stage 51.

Caractéristiques :

- mock des dépendances (ex. clients HTTP) selon `guide.tests.md` ;
- exécution de la méthode métier principale ;
- vérification du résultat (données, erreurs) ;
- vérification de certains effets (ex. appels à des hooks d’effets si exposés).

### 4.2. Test unitaire de hook logique

Cible : un hook généré par `hooks-logic` (stage 53).

Caractéristiques :

- exécution du hook dans un environnement de test abstrait (fourni par la stack) ;
- manipulation de l’API du hook (fonctions de callback, setters…) ;
- vérification des changements d’état, des retours et des effets.

### 4.3. Test d’intégration service + store + hook

Cible : un petit graphe d’artefacts.

Caractéristiques :

- initialisation d’un store ;  
- appel d’un service via un hook ;  
- vérification que les changements d’état correspondent aux UCR métier couverts.

### 4.4. Test de composant container / page

Cible : un container ou une page générée par les stages 56/57.

Caractéristiques :

- rendu du composant via les primitives de tests UI décrites par `guide.tests.md` ;  
- simulation d’interactions (clic, saisie, navigation locale) ;  
- vérification des effets visibles :
  - rendu conditionnel ;
  - appels d’actions ;
  - toasts / messages d’erreur.

### 4.5. Test de navigation / routing (si stack/tests le permettent)

Cible : artefacts de routing (stage 58) + pages.

Caractéristiques :

- vérification que les routes de la page mènent au bon rendu ;  
- paramétrage de segments de route (ids, slugs) ;  
- vérification des redirections et guards (si décrits dans le mapping.tests).

---

## 5. 🗂 Organisation des fichiers de tests

Les tests sont générés sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/`

L’organisation interne dépend de :

- `project-structure.json` ;
- `stack-guides/guide.tests.md`.

Exemples conceptuels (non prescriptifs) :

- `tests/unit/services/<ServiceName>.test.ext`  
- `tests/unit/hooks/<HookName>.test.ext`  
- `tests/integration/pages/<PageName>.test.ext`  
- `tests/ui/components/<ComponentName>.test.ext`  

Le guide de tests doit préciser :

- conventions de suffixes (`.test`, `.spec`, etc.) ;  
- définition de suites (par fichier, par bloc) ;  
- dossiers pour fixtures / mocks partagés.

---

## 6. 📝 `tests.meta.json`

Le stage génère aussi :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/tests/tests.meta.json`

Ce fichier contient au minimum :

```jsonc
{
  "pageName": "${project.pageName}",
  "testSuitesCount": 3,
  "testCasesCount": 12,
  "coveredUcr": {
    "tests": [
      "tests.useCase.createCampaign-1"
    ],
    "actions": [
      "actions.createCampaign-1"
    ],
    "logic": [
      "logic.workflow.createCampaign-1"
    ],
    "structure": [
      "structure.rootView.CampaignsCreate-1"
    ]
  },
  "generatedFiles": [
    "tests/unit/services/CreateCampaignService.test.ext",
    "tests/integration/pages/CampaignsCreatePage.test.ext"
  ],
  "issues": [
    "Missing test for actions.deleteCampaign-1"
  ],
  "coverageHints": {
    "missingUcr": [
      "actions.deleteCampaign-1"
    ],
    "weaklyCoveredUcr": []
  }
}
```

Ce fichier :

- sert de base à des rapports de couverture métier ;  
- permet d’identifier rapidement les zones non ou peu testées ;  
- peut être enrichi ultérieurement par les étapes de validation / reporting.

---

## 7. ✅ Checklist de génération pour `tests`

Avant de considérer que la génération de tests est complète pour `${project.pageName}` :

- [ ] `mapping.tests.json` est présent (si requis) et validé dans `mappings-summary.json`  
- [ ] Les use-cases métier principaux (`actions.*` de haut niveau) sont couverts au moins une fois  
- [ ] Les workflows critiques (`logic.workflow.*`) sont testés en intégration ou e2e  
- [ ] Les erreurs et validations importantes ont des tests dédiés  
- [ ] Les services, stores et hooks principaux ont au moins un test unitaire ou d’intégration  
- [ ] Les pages / containers critiques ont au moins un test de rendu / interaction  
- [ ] Les fichiers de tests sont générés dans `phase-3-generation/tests/` selon les conventions des stack-guides  
- [ ] `tests.meta.json` a été généré avec les nombres de suites, cas, UCR couverts et hints de couverture  
- [ ] Aucun issue bloquant ne remet en cause l’exécution des tests dans la stack cible

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
