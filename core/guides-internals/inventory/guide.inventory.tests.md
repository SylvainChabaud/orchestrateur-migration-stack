# 🔧 Guide Inventaire — Tests (`inventory.tests`)

*(Domaine d’inventaire : **Tests & Couverture fonctionnelle** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Tests** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **tests existants** (unitaires, composants, intégration, e2e, contract, visuels…).
2. Les **cibles de ces tests** (vues, hooks, services, actions, flows métier).
3. Leur **rôle fonctionnel** (ce que ces tests cherchent à garantir).
4. La **relation** entre les tests et les inventaires :
   - Structure,
   - Events, Logic, Dataflows, Async, Services, Routing, Effects, Actions,
   - Config éventuelle,
5. Les **gaps de couverture** : zones critiques non testées ou peu couvertes.

Il répond à la question :

> **“Qu’est-ce qui est réellement vérifié aujourd’hui par les tests autour de cette page, et qu’est-ce qui ne l’est pas ?”**

Ce domaine ne :

- ne remplace pas un rapport de couverture chiffré (coverage %),
- ne vise pas à décrire chaque assertion dans le détail,
- ne liste pas tous les tests techniques peu pertinents pour la migration.  

Il fournit une **carte de la couverture de test**, utile pour sécuriser la migration et prioriser les futures suites de tests dans la stack cible.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.tests.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"tests"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des tests/scénarios significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies, y compris les gaps de couverture

Exemple minimal :

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

---

### 2.2. Schéma interne — `items[]`

Chaque élément de `items[]` représente un **scénario de test significatif** (*TestItem*).

```text
items[] : TestItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du test/scénario, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Tests.

- `kind` : string  
  Type de test parmi un ensemble contrôlé, par exemple :
  - `"unit"`,
  - `"component"`,
  - `"integration"`,
  - `"e2e"`,
  - `"contract"`,
  - `"visual"`,
  - `"other"` (en dernier recours, avec explication).

- `sourcePath` : string  
  Chemin du fichier de test principal (`*.test.*`, `*.spec.*`, fichier e2e, etc.).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) correspondant aux vues/composants principalement ciblés par ce test.  
  - Peut être vide pour des tests purement back/service, mais à documenter dans `testSummary.scope`.

- `testSummary` : object  
  Résumé structuré du test, par exemple :
  - `testName`: nom logique du test/scénario (souvent proche du nom dans le code),
  - `tooling`: outil/framework de test (ex. `"jest"`, `"rtl"`, `"cypress"`, `"playwright"`),
  - `scope`: `"component" | "page" | "service" | "hook" | "flow" | "global"`,
  - `targetDomain`: domaine ciblé (ui, data, routing, actions, config, performance, etc.),
  - `mainAssertions`: liste textuelle des principaux comportements garantis,
  - `description`: phrase synthétique résumant le rôle du test.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen (test critique à conserver absolument),
  - `isRegressionGuard`: booléen (sert de garde-fou pour un bug déjà rencontré),
  - `severity`: `"low" | "medium" | "high"` (importance fonctionnelle),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) principalement couverts par ce test.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) ciblés par le test.

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` async (issus de `inventory.async.json`) dont le comportement est vérifié (succès/erreur/timeout…).

- `relatedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) vérifiés par le test.

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) impliqués dans le test.

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) pour les tests couvrant la navigation (redirections, guards, routes d’entrée/sortie).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) explicitement vérifiés (toasts, focus, tracking…).

- `relatedActionUcrs` : array de string  
  Liste des `ucr` d’actions (issus de `inventory.actions.json`) lorsque le test couvre un flow métier complet.

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) explicitement testés (feature flags, comportements conditionnels).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de tests doivent être **uniques** dans `inventory.tests.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json` (sauf pour tests purement services, à documenter).
- Les champs `related*Ucrs` / `relatedConfigNames` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le DSL peut inclure des concepts du type :

- `test.unit`
- `test.component`
- `test.integration`
- `test.e2e`
- `test.contract`
- `test.visual`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour :

- classifier les tests par type,
- relier certains tests à leurs cibles fonctionnelles (components, services, flows).

Si certaines entrées sont manquantes, l’IA doit :

- se baser sur :
  - la structure des fichiers (`*.test.*`, `*.spec.*`, dossiers e2e),
  - l’outillage utilisé (jest, rtl, cypress, playwright…),
  - les patterns d’API (ex. `describe/it`, `test()`, `cy.*`),
- documenter les incertitudes dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Tests doit :

1. **Identifier les tests pertinents pour `${project.pageName}`** :
   - par proximité de fichiers (tests dans le même dossier ou sous-arborescence),
   - par nommage (mention de la page, des composants, des services, des flows liés),
   - par contenu (URL de la page, labels UI caractéristiques, noms d’actions métier).
2. **Classifier** chaque test :
   - par `kind` (unit/component/integration/e2e/…),
   - par `scope` (component/page/service/flow/global),
   - par `targetDomain` (ui/data/routing/actions…).
3. **Relier** chaque test aux inventaires :
   - structure (vues testées),
   - events/logic/dataflows/async/services/routing/effects/actions/config,
   - afin de pouvoir raisonner sur la couverture.

### 3.3. Restrictions

L’inventaire Tests **ne doit pas** :

- lister tous les tests techniques trivials (ex. snapshot sans enjeu, test purement esthétique obsolète) sauf s’ils couvrent un comportement métier important,
- dupliquer l’intégralité du contenu des fichiers de test,
- tenter d’inférer une couverture chiffrée exacte (pourcentages).

Il doit se concentrer sur les **tests qui ont un impact réel sur la sécurisation fonctionnelle** de la page.

---

## 4. 🔗 Relations avec les autres inventaires

- **Tests ← Structure**
  - Les tests ciblent des vues/composants.  
    Références via `targetStructureUcrs`.

- **Tests ↔ Events / Logic / Actions**
  - Les tests vérifient des réactions à des événements, de la logique métier et des actions complètes.  
    Références via `relatedEventUcrs`, `relatedLogicUcrs`, `relatedActionUcrs`.

- **Tests ↔ Dataflows / Async / Services**
  - Les tests couvrent souvent des interactions asynchrones et des appels de services.  
    Références via `relatedAsyncUcrs`, `relatedDataflowUcrs`, `relatedServiceUcrs`.

- **Tests ↔ Routing / Effects / Config**
  - Certains tests vérifient la navigation, les effets UI et les comportements conditionnés par la config.  
    Références via `relatedRoutingUcrs`, `relatedEffectUcrs`, `relatedConfigNames`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de tests sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json` (ou explicitement vides/justifiés pour les tests purement services).
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `sourcePath`, `targetStructureUcrs`, `testSummary`, `metadata`) sont présents.
- [ ] Les liens vers les autres inventaires (events, logic, dataflows, async, services, routing, effects, actions, config) sont cohérents.
- [ ] `validation.status` et `validation.issues` sont cohérents, y compris pour les gaps de couverture.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "tests",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "test-saveCampaign-success-1",
      "kind": "integration",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/__tests__/CampaignsDetail.saveCampaign.test.tsx",
      "targetStructureUcrs": ["view-page-campaignsDetail-1"],
      "testSummary": {
        "testName": "should save campaign and show success toast",
        "tooling": "jest+rtl",
        "scope": "page",
        "targetDomain": "actions",
        "mainAssertions": [
          "when user fills form and clicks Save, API is called with correct payload",
          "detail view shows updated data on success",
          "success toast is displayed"
        ],
        "description": "Vérifie le scénario nominal de sauvegarde de campagne sur la page CampaignsDetail."
      },
      "metadata": {
        "isCritical": true,
        "isRegressionGuard": true,
        "severity": "high"
      },
      "relatedEventUcrs": [
        "event-onClickSaveCampaign-1"
      ],
      "relatedLogicUcrs": [
        "logic-handleSaveCampaign-1"
      ],
      "relatedAsyncUcrs": [
        "async-saveCampaignMutation-1"
      ],
      "relatedDataflowUcrs": [
        "dataflow-saveCampaign-1"
      ],
      "relatedServiceUcrs": [
        "service-CampaignsService-1"
      ],
      "relatedRoutingUcrs": [],
      "relatedEffectUcrs": [
        "effect-toastOnSaveSuccess-1"
      ],
      "relatedActionUcrs": [
        "action-saveCampaign-1"
      ],
      "relatedConfigNames": [
        "ENABLE_ADVANCED_CAMPAIGN_SAVE"
      ]
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
  "domain": "tests",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "test-saveCampaign-success-1",
      "kind": "integration",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/__tests__/CampaignsDetail.saveCampaign.test.tsx",
      "targetStructureUcrs": ["view-unknown-99"],
      "testSummary": {
        "testName": "should save campaign",
        "tooling": "jest",
        "scope": "page",
        "targetDomain": "actions",
        "mainAssertions": [],
        "description": "Test."
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
- `testSummary.mainAssertions` est vide et n’apporte aucune information.
- `validation.status` ne devrait pas être `"valid"`.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"tests"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de tests sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json` (ou explicitement vides/justifiés)  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées et les gaps de couverture  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les tests qui ont une **valeur fonctionnelle** claire pour la migration.
- Utiliser les autres inventaires pour relier les tests aux éléments importants :
  - structure (composants/pages),
  - events/logic/actions (flows métier),
  - dataflows/async/services (interactions système),
  - routing/effects/config (navigation, feedback utilisateur, comportements conditionnels).
- Exploiter `metadata.isCritical` et `metadata.isRegressionGuard` pour prioriser :
  - les tests à rejouer après migration,
  - les tests à re-créer si absents dans la stack cible.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Tests*
