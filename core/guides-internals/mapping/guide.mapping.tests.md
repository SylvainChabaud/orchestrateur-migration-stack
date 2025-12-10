# 🧭 Guide de Mapping — `mapping.tests`

*(Projection des concepts `tests.*` du DSL vers les artefacts de tests de la stack cible)*

---

## 1. 🎯 Rôle du mapping `tests`

Le domaine `tests.*` du DSL décrit **tout ce qui concerne la validation automatique du comportement** :

- tests unitaires (fonctions pures, hooks, services, logique) ;
- tests de composants (render, interactions, snapshot éventuellement) ;
- tests d’intégration (plusieurs couches ensemble) ;
- tests end-to-end (E2E) ou fonctionnels ;
- tests de régression ciblés (bugs connus à ne plus reproduire) ;
- éventuels tests de contrat (entre front et backend).

La **Phase 1 — Analyse** a produit :

- `inventory.tests.json` → inventaire des UCR `tests.*` ;  
- des liens possibles avec pratiquement tous les domaines :
  - `structure.*`, `layout.*`, `styles.*`, `i18n.*` ;
  - `hooks.*`, `logic.*`, `dataflows.*`, `services.*`, `async.*` ;
  - `routing.*`, `effects.*`, `actions.*`, `events.*`, `conditions.*`, `config.*`…

La **Phase 2 — Stage 45 — mapping.tests** doit :

> **Projeter chaque UCR `tests.*` vers un artefact de test concret dans la stack cible**, par exemple :
> - fichier de test unitaire ;
> - spec de composant ;
> - spec d’intégration ;
> - scénario E2E (Cypress / Playwright / autre) ;
> - test de contrat.

Objectif : rendre explicite **où se trouvent les tests, ce qu’ils couvrent, et avec quelle profondeur**.

---

## 2. 📦 Format JSON racine (`mapping.tests.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.tests.json`

Structure racine attendue :

```json
{
  "domain": "tests",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs principaux :

- `domain` : `"tests"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping

---

## 3. 🔗 Schéma d’un `MappingItem` pour les tests

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `tests.*` issu de `inventory.tests.json` ;
- à un fichier / scénario de test concret dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.tests.json",
    "domain": "tests",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "testLevel": "string",
    "hints": []
  },
  "coverage": {
    "structureUcrs": [],
    "layoutUcrs": [],
    "stylesUcrs": [],
    "i18nUcrs": [],
    "hooksUcrs": [],
    "logicUcrs": [],
    "dataflowsUcrs": [],
    "servicesUcrs": [],
    "asyncUcrs": [],
    "routingUcrs": [],
    "effectsUcrs": [],
    "actionsUcrs": [],
    "eventsUcrs": [],
    "conditionsUcrs": [],
    "configUcrs": []
  },
  "scope": {
    "focus": "string",
    "criticalPath": false
  },
  "metadata": {
    "isBlocking": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping **unique** dans `mapping.tests.json`.  
  - Préfixe recommandé : `map-tests-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept `tests.*` du DSL, par ex. :
    - `tests.unit` ;
    - `tests.component` ;
    - `tests.integration` ;
    - `tests.e2e` ;
    - `tests.contract`…

- `sourceInventoryRef.file`  
  - Toujours `"inventory.tests.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"tests"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire des tests.

- `toStack.stackKind`  
  - Type de test côté stack, par ex. :
    - `"unitTest"` ;
    - `"componentTest"` ;
    - `"integrationTest"` ;
    - `"e2eTest"` ;
    - `"contractTest"`.

- `toStack.targetId`  
  - Identifiant logique du test (suite, scénario, fichier), ex. :
    - `campaignsDetail.view.test`, `useCampaignForm.test`, `campaigns-e2e.spec`, etc.

- `toStack.targetPath`  
  - Chemin relatif du fichier de test, dérivé de `project-structure.json`, ex. :
    - `src/pages/CampaignsDetail/__tests__/CampaignsDetail.view.test.tsx` ;
    - `tests/e2e/campaigns/campaigns-detail.spec.ts`…

- `toStack.targetLayer`  
  - `"presentation"` pour les tests de composants / vues ;
  - `"application"` pour les tests de use-cases / logique ;
  - `"infrastructure"` pour les tests de services / adaptateurs ;
  - `"e2e"` pour les tests bout-en-bout.

- `toStack.testLevel`  
  - `"unit"`, `"component"`, `"integration"`, `"e2e"`, `"contract"`, etc.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - `"jest"`, `"vitest"`, `"cypress"`, `"playwright"`, `"rtl"` (testing-library), etc.

- `toStack.targetPattern`  
  - `"testFile"`, `"specFile"`, `"describeBlock"`, `"featureFile"`…

- `toStack.hints[]`  
  - Conseils : `"Limiter les mocks pour ce test"`, `"Tester les chemins d'erreur"`, etc.

- `coverage.*Ucrs`  
  - UCR couverts par ce test, par domaine.  
  - Exemple : `coverage.actionsUcrs = ["map-actions-saveCampaign-1"]`.

- `scope.focus`  
  - Description courte de ce qui est testé : `"validation formulaire"`, `"navigation après sauvegarde"`, etc.

- `scope.criticalPath`  
  - `true` si le test couvre un chemin critique (ex. sauvegarde, paiement).

- `metadata.isBlocking`  
  - `true` si le test est bloquant pour le pipeline (ex. smoke / sanity).

- `metadata.priority`  
  - `"low"`, `"normal"`, `"high"`.

---

## 4. ⚙️ Entrées requises pour `mapping.tests`

### 4.1. Configuration (obligatoire)

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

### 4.2. Artefacts Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tests.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.testing-strategy.md` (si présent)

### 4.3. Inventaires Phase 1 (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.tests.json` (obligatoire)  
- `inventories-summary.json` (obligatoire)  
- inventaires optionnels (pour enrichir la couverture) :
  - `inventory.structure.json`
  - `inventory.layout.json`
  - `inventory.styles.json`
  - `inventory.i18n.json`
  - `inventory.hooks.json`
  - `inventory.logic.json`
  - `inventory.dataflows.json`
  - `inventory.services.json`
  - `inventory.async.json`
  - `inventory.routing.json`
  - `inventory.effects.json`
  - `inventory.actions.json`
  - `inventory.events.json`
  - `inventory.conditions.json`
  - `inventory.config.json`

### 4.4. Mappings Phase 2 déjà produits (lecture seule)

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

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

---

## 5. 🧠 Règles d’interprétation

1. **Aucune relecture du Legacy**  
   - Toutes les décisions se basent sur les inventaires, la structure cible, les guides et les mappings.

2. **Tests alignés avec les responsabilités**  
   - Un test de composant doit couvrir principalement la **vue** + interactions de base ;
   - Un test de use-case doit couvrir la **logique** + orchestrations (services, effets, routing) ;
   - Un test E2E doit couvrir un **scénario métier complet**.

3. **Couverture traçable par UCR**  
   - Chaque test devrait idéalement référencer les UCR principaux qu’il couvre (`coverage.*Ucrs`).

4. **Hiérarchie de criticité**  
   - Prioriser les tests sur :
     - les actions critiques ;
     - les flux financiers ;
     - les chemins de navigation centraux ;
     - les effets réglementaires (RGPD, conformité…).

5. **Limiter la duplication**  
   - Éviter plusieurs tests couvrant exactement la même chose au même niveau ;
   - Répartir les responsabilités entre unit / component / integration / e2e.

---

## 6. Exemple simplifié de `mapping.tests.json`

```json
{
  "domain": "tests",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-tests-component-CampaignsDetail-view-1",
      "fromDsl": "tests.component",
      "sourceInventoryRef": {
        "file": "inventory.tests.json",
        "domain": "tests",
        "itemUcr": "tests-component-CampaignsDetail-view-1"
      },
      "toStack": {
        "stackKind": "componentTest",
        "targetId": "CampaignsDetailView.test",
        "targetPath": "src/pages/CampaignsDetail/__tests__/CampaignsDetail.view.test.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "jest+rtl",
        "targetPattern": "testFile",
        "testLevel": "component",
        "hints": [
          "Couvrir l'affichage des champs principaux",
          "Tester les états loading / error / success"
        ]
      },
      "coverage": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-main-1"],
        "stylesUcrs": [],
        "i18nUcrs": ["i18n-page-CampaignsDetail-title-1"],
        "hooksUcrs": ["hooks.viewModel-CampaignsDetail-1"],
        "logicUcrs": ["logic.viewLifecycle-CampaignsDetail-1"],
        "dataflowsUcrs": ["dataflows.read-CampaignsDetail-byId-1"],
        "servicesUcrs": ["service.http.CampaignsService.getById-1"],
        "asyncUcrs": ["async.polling-CampaignsDetail-status-1"],
        "routingUcrs": ["routing-page-CampaignsDetail-main-1"],
        "effectsUcrs": [],
        "actionsUcrs": [],
        "eventsUcrs": ["events.ui.open-CampaignsDetail-main-1"],
        "conditionsUcrs": [],
        "configUcrs": ["config.features.campaignsDetail.enabled"]
      },
      "scope": {
        "focus": "rendu initial + états de base",
        "criticalPath": true
      },
      "metadata": {
        "isBlocking": true,
        "priority": "high",
        "notes": "Test de fumée pour la vue principale CampaignsDetail."
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

## 7. ✅ Checklist de validation

- [ ] `inventory.tests.json` présent et déclaré exploitable dans `inventories-summary.json`  
- [ ] `project-structure.json` accessible  
- [ ] Les mappings précédents sont suffisants pour référencer les UCR couverts (`coverage.*Ucrs`)  
- [ ] Chaque UCR `tests.*` important a une projection dans `mapping.tests.json`  
- [ ] Tous les `MappingItem` ont `toStack.*` complets + `testLevel` cohérent  
- [ ] Les tests critiques et/ou bloquants sont marqués (`scope.criticalPath`, `metadata.isBlocking`)  
- [ ] `validation.status` est `"valid"` ou `"rejected"` et cohérent avec `validation.issues`  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.tests` (Stage 45 — Phase 2 : Interprétation)*
