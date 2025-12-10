# 🔧 Guide Inventaire — Effects (`inventory.effects`)

*(Domaine d’inventaire : **Effects / Side-effects** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Effects** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **effets UI** (focus, scroll, animations, overlays, toasts, notifications…).
2. Les **effets de navigation** (redirections, changement de route déclenchés par la logique).
3. Les **effets de lifecycle** (montage/démontage, subscriptions, side-effects de hooks).
4. Les **effets liés aux données** (post-traitement, synchro, mise à jour de caches/stores globaux).
5. Les **effets techniques** (tracking analytics, logs, instrumentation).

Il répond à la question :

> **“Quels comportements secondaires (non purement fonctionnels) se déclenchent en réaction aux événements et changements d’état de la page ?”**

Ce domaine ne :

- ne remplace pas `inventory.logic` (qui décrit la logique métier),
- ne remplace pas `inventory.async` (qui se concentre sur les patterns temporels et stratégies asynchrones),
- ne remplace pas `inventory.routing` (qui cartographie les routes et flows de navigation).  

Il se concentre sur la **réaction** : *quand X se passe, alors Y est déclenché comme effet secondaire*.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.effects.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"effects"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des effets significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "effects",
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

Chaque élément de `items[]` représente un **effet significatif** (*EffectItem*).

```text
items[] : EffectItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’effet, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Effects.

- `kind` : string  
  Type d’effet parmi un ensemble contrôlé, par exemple :
  - `"uiEffect"` (affichage/masquage de modale, toast, focus, scroll),
  - `"navigationEffect"` (redirection, push/replace de route),
  - `"lifecycleEffect"` (subscription, nettoyage, initialisation),
  - `"dataEffect"` (post-traitement, mise à jour de cache/store),
  - `"trackingEffect"` (analytics, logs, instrumentation),
  - `"globalStateEffect"` (mutation de state global ou contexte partagé).

- `sourcePath` : string  
  Chemin du fichier Legacy principal où l’effet est défini (composant, hook, service…).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) correspondant aux vues/composants impactés par cet effet.  
  - Pour un effet de navigation global, la vue principale de `${project.pageName}` peut suffire.

- `effectSummary` : object  
  Résumé structuré de l’effet, par exemple :
  - `effectType`: description courte (ex. `"toastOnSaveSuccess"`, `"redirectOnUnauthorized"`, `"focusOnFirstErrorField"`),
  - `trigger`: facteur déclencheur (lifeCycle, event, asyncResult, routeChange, configChange…),
  - `target`: cible principale de l’effet (ui, navigation, data, service, globalState),
  - `timing`: `"onMount" | "onUnmount" | "onUpdate" | "onEvent" | "deferred" | "immediate"`,
  - `dependencies`: liste textuelle des dépendances principales (state, props, params, features),
  - `description`: phrase synthétique détaillant le *“quand / quoi / pourquoi”*.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen,
  - `isGlobal`: booléen (impacte tout le module/store global),
  - `severity`: `"low" | "medium" | "high"` (impact si mal migré),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) déclencheurs de cet effet.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) au sein desquels cet effet est encastré.

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` async (issus de `inventory.async.json`) dont l’issue déclenche cet effet (succès/erreur/timeout).

- `relatedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) directement liés à l’effet (ex. rafraîchissement de données).

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) utilisés dans l’effet (tracking, logs, API secondaires).

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) lorsque l’effet déclenche ou dépend d’un changement de route.

- `relatedActionUcrs` : array de string  
  Liste des `ucr` d’actions (issus de `inventory.actions.json`) dans lesquelles l’effet intervient.

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui conditionnent l’effet (feature flags, toggles).

- `relatedTestUcrs` : array de string  
  Liste des `ucr` de tests (issus de `inventory.tests.json`) couvrant explicitement cet effet.

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` d’effets doivent être **uniques** dans `inventory.effects.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `related*Ucrs` / `relatedConfigNames` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le DSL peut inclure des concepts du type :

- `effect.logicTriggered`
- `effect.ui.focus`
- `effect.ui.scroll`
- `effect.ui.feedback` (toasts, banners)
- `effect.lifecycle.mount`
- `effect.lifecycle.unmount`
- `effect.navigation`
- `effect.data.sync`
- `effect.tracking`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour les identifier.  
Si certaines entrées sont manquantes, l’IA doit :

- repérer les hooks d’effets (`useEffect`, `useLayoutEffect`, équivalents Vue/Angular…),
- repérer les callbacks qui déclenchent des side-effects (tracking, navigation, accès global store),
- identifier les effets automatisés (subscriptions, timers) et leurs cleanups,
- documenter les incertitudes dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Effects doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour repérer :
   - les hooks/constructs d’effets (React/Vue/Angular…),
   - les appels à des services de tracking/logging/toasts,
   - les navigations déclenchées côté logique (router.push/history.push…),
   - les effets sur des stores globaux ou contextes,
   - les subscriptions (WebSocket, observables…) et leurs cleanups.
2. Pour chaque effet significatif :
   - déterminer `kind`,
   - identifier le déclencheur (event, lifecycle, async, routing, config),
   - identifier la cible (UI, data, routing, global state, external service),
   - relier l’effet :
     - à la vue impactée (`targetStructureUcrs`),
     - aux events/logic/dataflows/async/services/routing/actions/tests concernés.

### 3.3. Restrictions

L’inventaire Effects **ne doit pas** :

- modéliser chaque petit side-effect trivial (ex. log de debug passager) si non pertinent,
- dupliquer toute la logique métier,
- dupliquer les détails asynchrones (cela reste dans `inventory.async`).  

Il doit se concentrer sur les **effets structurants** pour le comportement et l’expérience utilisateur.

---

## 4. 🔗 Relations avec les autres inventaires

- **Effects ← Structure**
  - Les effets impactent toujours une vue, un composant ou un layout.  
    Références via `targetStructureUcrs`.

- **Effects ↔ Events / Logic / Actions**
  - Les effets sont souvent déclenchés par des événements, orchestrés par de la logique et regroupés dans des actions.  
    Références via `relatedEventUcrs`, `relatedLogicUcrs`, `relatedActionUcrs`.

- **Effects ↔ Async / Dataflows / Services**
  - Les effets peuvent réagir à l’issue de dataflows/operations async/sur des services.  
    Références via `relatedAsyncUcrs`, `relatedDataflowUcrs`, `relatedServiceUcrs`.

- **Effects ↔ Routing / Config / Tests**
  - Certains effets concernent la navigation, d’autres sont conditionnés par la config ou couverts par des tests dédiés.  
    Références via `relatedRoutingUcrs`, `relatedConfigNames`, `relatedTestUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` d’effets sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `sourcePath`, `targetStructureUcrs`, `effectSummary`, `metadata`) sont présents.
- [ ] Les liens vers les autres inventaires (events, logic, dataflows, async, services, routing, actions, config, tests) sont cohérents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "effects",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "effect-toastOnSaveSuccess-1",
      "kind": "uiEffect",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/CampaignsDetail.tsx",
      "targetStructureUcrs": ["view-page-campaignsDetail-1"],
      "effectSummary": {
        "effectType": "toastOnSaveSuccess",
        "trigger": "asyncResult:saveCampaign.success",
        "target": "ui",
        "timing": "onEvent",
        "dependencies": [
          "result of saveCampaign mutation",
          "feature flag ENABLE_CAMPAIGN_TOASTS"
        ],
        "description": "Affiche un toast de succès lorsqu’une campagne est sauvegardée avec succès sur la page CampaignsDetail."
      },
      "metadata": {
        "isCritical": false,
        "isGlobal": false,
        "severity": "medium"
      },
      "relatedEventUcrs": [
        "event-onClickSaveCampaign-1"
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
      "relatedConfigNames": [
        "ENABLE_CAMPAIGN_TOASTS"
      ],
      "relatedTestUcrs": [
        "test-shouldShowSuccessToastOnSave-1"
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
  "domain": "effects",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "effect-toastOnSaveSuccess-1",
      "kind": "uiEffect",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/CampaignsDetail.tsx",
      "targetStructureUcrs": ["view-unknown-99"],
      "effectSummary": {
        "effectType": "toastOnSaveSuccess",
        "trigger": "asyncResult:saveCampaign.success",
        "target": "ui",
        "timing": "onEvent",
        "dependencies": [],
        "description": "Toast succès."
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
- `validation.status` ne devrait pas être `"valid"`.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"effects"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` d’effets sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne pas surcharger l’inventaire avec chaque log de debug ou effet mineur sans enjeu fonctionnel.
- Se concentrer sur les effets qui impactent :
  - la navigation,
  - l’état global,
  - l’expérience utilisateur visible (toasts, modales, loaders),
  - la cohérence des données (rafraîchissements, cache, synchro).
- Utiliser `metadata.severity` + `validation.issues` pour mettre en lumière :
  - les effets critiques à sécuriser lors de la migration,
  - les patterns d’effets dispersés qu’il serait pertinent de centraliser.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Effects*
