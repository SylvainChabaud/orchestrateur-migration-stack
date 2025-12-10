# 🔧 Guide Inventaire — Routing (`inventory.routing`)

*(Domaine d’inventaire : **Routing / Navigation** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Routing** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **routes entrantes** : comment on arrive sur cette page (URL, paramètres, contexte).
2. Les **routes sortantes** : vers où on peut aller à partir de cette page (navigation déclenchée par l’utilisateur ou le système).
3. Les **paramètres d’URL** (route params, query params, fragments) pertinents pour le comportement de la page.
4. Les **guards** et préconditions : auth, permissions, feature flags, état système.
5. Les **flows de navigation structurants** (enchaînement d’écrans, modales, redirect conditionnel).

Il répond à la question :

> **“Dans quel contexte de navigation cette page existe-t-elle, et comment s’insère-t-elle dans les flows de l’application ?”**

Ce domaine ne :

- ne redécrit pas tout le router global de l’application,
- ne remplace pas l’inventaire des dataflows ou services,
- ne se substitue pas à l’inventaire des effets (toasts, tracking…), mais peut y faire référence.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.routing.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"routing"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des routes/flows/guards significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "routing",
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

Chaque élément de `items[]` représente un **élément de routing significatif** (*RouteItem*).

```text
items[] : RouteItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’élément de routing, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Routing.

- `kind` : string  
  Type d’élément de routing parmi un ensemble contrôlé, par exemple :
  - `"pageRoute"` (route principale menant à la page),
  - `"nestedRoute"` (sous-route/tab/segment enfant),
  - `"modalRoute"` (route représentant une modale),
  - `"redirect"` (redirection),
  - `"guard"` (condition d’accès),
  - `"navigationFlow"` (flow de navigation complet, ex. “création campagne”),
  - `"fallbackRoute"` (404/route par défaut).

- `sourcePath` : string  
  Chemin du fichier Legacy principal où l’élément de routing est défini ou orchestré (fichier de routes, composant shell, hook de routing).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) correspondant :
  - pour une route : à la vue/page/layout affichée,
  - pour une modale : à la vue modale correspondante,
  - pour un flow : aux vues impliquées.

- `routingSummary` : object  
  Résumé structuré de l’élément de routing, par exemple :
  - `routeId`: identifiant logique de la route/flow (ex. `"campaigns-detail"`, `"campaigns-create-flow"`),
  - `pathPattern`: pattern d’URL (ex. `"/campaigns/:campaignId"`, `"/campaigns/new"`),
  - `params`: description des route params (nom, type métier, obligatoire/facultatif),
  - `queryParams`: description des query params utilisés,
  - `entryConditions`: conditions d’entrée (auth, feature flags, état requis),
  - `exitDestinations`: routes principales de sortie (suivant le flow),
  - `navigationTriggers`: description des événements/éléments UI déclenchant cette navigation,
  - `dataDependencies`: liens avec des dataflows/services critiques pour cette route,
  - `description`: phrase synthétique expliquant le rôle de cette route/flow.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isPrimaryEntryPoint`: booléen (route d’entrée principale vers `${project.pageName}`),
  - `isModal`: booléen,
  - `isProtected`: booléen (nécessite auth/permissions),
  - `severity`: `"low" | "medium" | "high"` (impact métier si cassé),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) déclenchés lors :
  - de l’entrée sur la route,
  - de changements de params,
  - de sorties conditionnelles.

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) sollicités lors de l’entrée/sortie sur la route.

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` async (issus de `inventory.async.json`) représentant des patterns déclenchés par la navigation (préchargement, loaders).

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) conditionnant l’accès ou le comportement de la route (ex. guards métier).

- `relatedHookUcrs` : array de string  
  Liste des `ucr` de hooks (issus de `inventory.hooks.json`) liés au routing (hooks custom utilisant `useRouter`, `useLocation`, etc.).

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) déclencheurs de navigation (clic sur un bouton, validation de formulaire).

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui influencent le routing (feature flags, chemins conditionnels, routes activées/désactivées).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) associés aux transitions de route (tracking page view, toasts de redirection, etc.).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de routing doivent être **uniques** dans `inventory.routing.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `relatedDataflowUcrs`, `relatedServiceUcrs`, `relatedAsyncUcrs`, `relatedLogicUcrs`, `relatedHookUcrs`, `relatedEventUcrs`, `relatedConfigNames`, `relatedEffectUcrs` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le DSL peut inclure des concepts du type :

- `routing.pageRoute`
- `routing.nestedRoute`
- `routing.modalRoute`
- `routing.redirect`
- `routing.guard`
- `routing.navigationFlow`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour les identifier.  
Si certaines entrées sont manquantes, l’IA doit :

- repérer les usages de routers/frameworks (React Router, Next Router, Vue Router, etc.),
- analyser les composants de navigation (`Link`, `NavLink`, `router.push`, `navigate`),
- identifier les configs de routes et les flows de navigation autour de `${project.pageName}`,
- documenter les incertitudes dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Routing doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour repérer :
   - la ou les routes menant à `${project.pageName}`,
   - les routes enfants (tabs, sous-sections, modales conditionnées par l’URL),
   - les routes vers lesquelles la page redirige ou permet de naviguer,
   - les guards et conditions d’accès (auth, permissions, flags).
2. Pour chaque élément significatif :
   - déterminer `kind`,
   - identifier le pattern d’URL (si applicable),
   - recenser les params/queries utilisés par la page,
   - relier les routes :
     - aux dataflows/services (préchargements, actions sur navigation),
     - à l’async (préchargements, loaders),
     - aux événements/logic (déclencheurs de navigation).

### 3.3. Restrictions

L’inventaire Routing **ne doit pas** :

- décrire chaque petit lien de navigation trivial qui n’apporte aucune information structurelle,
- devenir un dump complet de toute la configuration du router de l’application,
- dupliquer le détail des dataflows ou de la logique métier.  

Il doit se concentrer sur les **routes et flows structurants** pour la page `${project.pageName}`.

---

## 4. 🔗 Relations avec les autres inventaires

- **Routing ← Structure**
  - Le routing s’ancre dans les vues/page/layouts via `targetStructureUcrs`.

- **Routing ↔ Dataflows / Services**
  - L’entrée sur une route déclenche souvent des dataflows/services.  
    Références via `relatedDataflowUcrs`, `relatedServiceUcrs`.

- **Routing ↔ Async**
  - Les changements de route s’accompagnent de comportements async (préchargement, loaders, redirections différées).  
    Références via `relatedAsyncUcrs`.

- **Routing ↔ Logic / Hooks / Events**
  - La navigation peut être déclenchée par des événements/logic, et des hooks peuvent abstraire le routing.  
    Références via `relatedLogicUcrs`, `relatedHookUcrs`, `relatedEventUcrs`.

- **Routing ↔ Config / Effects**
  - La config peut activer/désactiver certaines routes, les effets peuvent enregistrer des vues (tracking).  
    Références via `relatedConfigNames`, `relatedEffectUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de routing sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `sourcePath`, `targetStructureUcrs`, `routingSummary`, `metadata`) sont présents.
- [ ] Les liens vers les autres inventaires (dataflows, async, services, logic, hooks, events, config, effects) sont cohérents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "routing",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "route-campaigns-detail-main-1",
      "kind": "pageRoute",
      "sourcePath": "src/packages/promo-boost/router/routes.tsx",
      "targetStructureUcrs": ["view-page-campaignsDetail-1"],
      "routingSummary": {
        "routeId": "campaigns-detail",
        "pathPattern": "/campaigns/:campaignId",
        "params": [
          {
            "name": "campaignId",
            "type": "string",
            "domainType": "campaignId",
            "required": true
          }
        ],
        "queryParams": [
          {
            "name": "tab",
            "domainType": "campaignTab",
            "required": false
          }
        ],
        "entryConditions": [
          "user must be authenticated",
          "user must have permission CAMPAIGNS_READ"
        ],
        "exitDestinations": ["/campaigns", "/campaigns/new"],
        "navigationTriggers": [
          "click on campaign row in campaigns list",
          "redirect after campaign creation"
        ],
        "dataDependencies": [
          "fetchCampaignDetail",
          "fetchCampaignStats"
        ],
        "description": "Route principale permettant d’afficher le détail d’une campagne à partir de son identifiant."
      },
      "metadata": {
        "isPrimaryEntryPoint": true,
        "isModal": false,
        "isProtected": true,
        "severity": "high"
      },
      "relatedDataflowUcrs": [
        "dataflow-fetchCampaignDetail-1",
        "dataflow-fetchCampaignStats-1"
      ],
      "relatedServiceUcrs": [
        "service-CampaignsService-1"
      ],
      "relatedAsyncUcrs": [
        "async-parallelFetchCampaignsAndStats-1"
      ],
      "relatedEventUcrs": [
        "event-onClickCampaignRow-1"
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
  "domain": "routing",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "route-campaigns-detail-main-1",
      "kind": "pageRoute",
      "sourcePath": "src/packages/promo-boost/router/routes.tsx",
      "targetStructureUcrs": ["view-unknown-99"],
      "routingSummary": {
        "routeId": "campaigns-detail",
        "pathPattern": "/campaigns/:campaignId",
        "params": [],
        "queryParams": [],
        "entryConditions": [],
        "exitDestinations": [],
        "navigationTriggers": [],
        "dataDependencies": [],
        "description": "Route."
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

- [ ] `domain` est `"routing"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de routing sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les routes et flows **structurants** pour `${project.pageName}`, pas chaque micro-lien.
- Toujours relier :
  - une route à au moins une vue (`targetStructureUcrs`),
  - si possible à des dataflows / services / événements / hooks.
- Utiliser `metadata.severity` et `validation.issues` pour mettre en lumière :
  - les routes critiques (non sécurisées, fortement couplées au métier),
  - les flows compliqués (redirections multiples, dépendances implicites sur les params).

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Routing*
