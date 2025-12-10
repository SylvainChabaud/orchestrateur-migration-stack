# 🔧 Guide Inventaire — Actions (`inventory.actions`)

*(Domaine d’inventaire : **Actions métier / UX end-to-end** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Actions** décrit, pour une page ou un module donné (`${project.pageName}`), les **use cases concrets** vus par le métier / l’utilisateur, sous forme de **chaînes end-to-end** :

> **“Quand l’utilisateur (ou le système) fait X, alors la page exécute la séquence Y (events → logic → dataflows → services → async → routing → effects).”**

Il vise à :

1. Nommer clairement les **actions significatives** (create/update/delete/duplicate, changement d’état important, export, validation, etc.).
2. Documenter leurs **triggers** (user events, timers, route changes, conditions système).
3. Décrire les **flows principaux** : étapes clés, dataflows impliqués, services, effets de navigation et UI.
4. Mettre en évidence les **préconditions** (auth, droits, feature flags, état requis).
5. Identifier les **issues** (succès, erreurs, annulation) et les effets associés (toasts, redirections, etc.).

Ce domaine ne :

- ne remplace pas les inventaires détaillés (events, logic, dataflows, async, services, routing, effects),
- ne vise pas à décrire toutes les micro-actions triviales,
- ne duplique pas la logique métier ligne par ligne.  

Il fournit une **vue macro**, lisible par un humain (dev/PO/architecte), de ce que fait réellement la page.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.actions.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"actions"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des actions significatives (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "actions",
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

Chaque élément de `items[]` représente une **Action significative** (*ActionItem*).

```text
items[] : ActionItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’action, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Actions.

- `kind` : string  
  Type d’action parmi un ensemble contrôlé, par exemple :
  - `"userAction"` (déclenchée par l’utilisateur, ex. clic, submit),
  - `"systemAction"` (déclenchée par le système, ex. timer, route change, async callback),
  - `"compositeAction"` (enchaînement d’actions élémentaires, ex. “Créer puis activer une campagne”),
  - `"backgroundAction"` (tâche en arrière-plan peu visible mais importante).

- `sourcePath` : string  
  Chemin du fichier Legacy principal où la **logique orchestrant l’action** est définie (composant, hook, service, orchestrateur).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) correspondant aux vues/composants :
  - où l’action est initiée (CTA, formulaire),
  - et/ou où son résultat est visible.

- `actionSummary` : object  
  Résumé structuré de l’action, par exemple :
  - `actionName`: nom logique de l’action (ex. `"createCampaign"`, `"saveCampaign"`, `"duplicateCampaign"`),
  - `trigger`: description du déclencheur principal (ex. `"onClick:saveButton"`, `"onMount:route=/campaigns/:id"`),
  - `preconditions`: liste textuelle des conditions d’entrée (auth, permissions, état requis, params route),
  - `mainFlowSteps`: liste d’étapes textuelles décrivant le flow end-to-end (ex. `["validate form", "call saveCampaign API", "refresh list", "show success toast"]`),
  - `successCriteria`: description des conditions de succès (ex. “API répond 2xx, state mis à jour, toast succès affiché”),
  - `failureHandling`: description des comportements en échec (toasts d’erreur, retries, redirections, rollbacks),
  - `sideEffects`: résumé des effets secondaires (tracking, navigation, mutation de state global),
  - `description`: phrase synthétique expliquant le use case.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen (impact fort métier si l’action ne fonctionne pas),
  - `isPrimaryFlow`: booléen (fait partie du “happy path” principal),
  - `severity`: `"low" | "medium" | "high"` (importance fonctionnelle),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) déclencheurs ou intermédiaires de cette action.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) participant au flow (validation, orchestration, transformation).

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` async (issus de `inventory.async.json`) utilisés dans le flow (mutations, loaders, retries).

- `relatedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) utilisés par l’action.

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) sollicités dans le flow.

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) impliqués (routes d’entrée, redirections, routes de sortie).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) liés à l’action (toasts, navigation, tracking, mise à jour UI).

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui conditionnent cette action (ex. feature flags).

- `relatedTestUcrs` : array de string  
  Liste des `ucr` de tests (issus de `inventory.tests.json`) couvrant l’action end-to-end ou des parties critiques.

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` d’actions doivent être **uniques** dans `inventory.actions.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `related*Ucrs` / `relatedConfigNames` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le DSL peut inclure des concepts du type :

- `action.userAction`
- `action.systemAction`
- `action.compositeFlow`
- `action.backgroundFlow`

ainsi que des liens entre :

- `event.*` → `logic.*` → `effect.*` → `data.*` → `routing.*`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour les identifier/co-relier.  
Si certaines entrées sont manquantes, l’IA doit :

- reconstruire les chaînes à partir des inventaires Events / Logic / Dataflows / Async / Services / Routing / Effects,
- grouper les chaînes en actions cohérentes du point de vue métier,
- documenter les incertitudes dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Actions doit :

1. Parcourir `inventory.events`, `inventory.logic`, `inventory.dataflows`, `inventory.async`, `inventory.services`, `inventory.routing`, `inventory.effects` pour :
   - identifier les couples “événement déclencheur” → “handler principal”,
   - suivre les dataflows/services appelés, les navigations et effets associés.
2. Regrouper ces éléments en **Actions** :
   - en se basant sur :
     - le contexte UI (même bouton/formulaire/menu),
     - le même objectif métier (ex. “sauvegarder une campagne”),
     - le même cycle de vie (init → traitement → retour utilisateur).
3. Pour chaque Action :
   - déterminer `kind` (user/system/composite/background),
   - renseigner les champs de `actionSummary`,
   - lier l’action à :
     - ses events/logic/dataflows/async/services/routing/effects,
     - la structure UI,
     - la config et les tests le cas échéant.

### 3.3. Restrictions

L’inventaire Actions **ne doit pas** :

- multiplier les actions très techniques sans valeur métier (ex. “mettre à jour un champ local”),
- confondre une Action avec un simple event ou un simple dataflow,
- tenter de couvrir chaque micro-variation de scénario.  

Il doit se concentrer sur les **use cases métier/UX importants**, typiquement ceux qui seraient visibles dans une spec fonctionnelle.

---

## 4. 🔗 Relations avec les autres inventaires

- **Actions ← Structure**
  - Les actions sont initiées/observées dans des vues/composants.  
    Références via `targetStructureUcrs`.

- **Actions ↔ Events / Logic / Effects**
  - Les actions s’appuient sur des événements, orchestrées par de la logique, et produisent des effets.  
    Références via `relatedEventUcrs`, `relatedLogicUcrs`, `relatedEffectUcrs`.

- **Actions ↔ Dataflows / Async / Services**
  - Les actions consomment des dataflows et services, souvent via des patterns async.  
    Références via `relatedDataflowUcrs`, `relatedAsyncUcrs`, `relatedServiceUcrs`.

- **Actions ↔ Routing / Config / Tests**
  - Les actions peuvent déclencher de la navigation, être conditionnées par la config, et/ou être couvertes par des tests.  
    Références via `relatedRoutingUcrs`, `relatedConfigNames`, `relatedTestUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` d’actions sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `sourcePath`, `targetStructureUcrs`, `actionSummary`, `metadata`) sont présents.
- [ ] Les liens vers les autres inventaires (events, logic, dataflows, async, services, routing, effects, config, tests) sont cohérents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "actions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "action-saveCampaign-1",
      "kind": "userAction",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/CampaignsDetail.tsx",
      "targetStructureUcrs": ["view-page-campaignsDetail-1"],
      "actionSummary": {
        "actionName": "saveCampaign",
        "trigger": "onClick:saveButton",
        "preconditions": [
          "user must be authenticated",
          "user must have permission CAMPAIGNS_WRITE",
          "form must be valid"
        ],
        "mainFlowSteps": [
          "validate form",
          "call saveCampaign API",
          "refresh campaigns detail data",
          "show success toast"
        ],
        "successCriteria": "API responds 2xx, detail view shows updated data, success toast displayed.",
        "failureHandling": "Show error toast, keep user on same page, keep form values.",
        "sideEffects": [
          "tracking event CAMPAIGN_SAVE_SUCCESS sent",
          "logs written to technical logger"
        ],
        "description": "Action permettant de sauvegarder une campagne depuis la page CampaignsDetail."
      },
      "metadata": {
        "isCritical": true,
        "isPrimaryFlow": true,
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
        "dataflow-saveCampaign-1",
        "dataflow-fetchCampaignDetail-1"
      ],
      "relatedServiceUcrs": [
        "service-CampaignsService-1"
      ],
      "relatedRoutingUcrs": [],
      "relatedEffectUcrs": [
        "effect-toastOnSaveSuccess-1"
      ],
      "relatedConfigNames": [
        "ENABLE_ADVANCED_CAMPAIGN_SAVE"
      ],
      "relatedTestUcrs": [
        "test-shouldSaveCampaignOnValidForm-1"
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
  "domain": "actions",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "action-saveCampaign-1",
      "kind": "userAction",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/CampaignsDetail.tsx",
      "targetStructureUcrs": ["view-unknown-99"],
      "actionSummary": {
        "actionName": "saveCampaign",
        "trigger": "onClick:saveButton",
        "preconditions": [],
        "mainFlowSteps": [],
        "successCriteria": "",
        "failureHandling": "",
        "sideEffects": [],
        "description": "Save."
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
- `actionSummary` est trop pauvre et n’aide pas à comprendre le flow métier.
- `validation.status` ne devrait pas être `"valid"`.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"actions"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` d’actions sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les **use cases qui compteraient dans une spec fonctionnelle** (création, modification, suppression, actions critiques, flows de validation).
- Utiliser les autres inventaires comme **briques** pour construire les Actions :
  - events = triggers,
  - logic = orchestrateurs,
  - dataflows/services = interactions système,
  - async = temporalité,
  - routing = navigation,
  - effects = feedback utilisateur / side-effects.
- Utiliser `metadata.isCritical` et `metadata.isPrimaryFlow` pour marquer les actions à fort enjeu et le “happy path”.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Actions*
