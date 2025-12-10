# 🔧 Guide Inventaire — Services (`inventory.services`)

*(Domaine d’inventaire : **Services / Facades techniques** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Services** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **services / facades techniques** utilisés (clients API, repositories, adaptateurs, intégrations externes).
2. Leurs **responsabilités** : quelles zones fonctionnelles ou métiers ils adressent.
3. Leurs **opérations exposées** (méthodes publiques pertinentes pour le domaine).
4. Les **dataflows** qu’ils encapsulent ou orchestrent.
5. Les liens entre services, **UI**, **logique**, **async**, **config**, **routing**, **effects**.

Il répond à la question :

> **“Par quels services/facades cette page parle‑t‑elle au reste du système (backend, tiers, stores, caches) ?”**

Ce domaine ne :

- ne liste pas chaque appel HTTP en détail (`inventory.dataflows` s’en charge),
- ne décrit pas toute la logique métier (`inventory.logic`),
- ne remplace pas l’inventaire Async / Routing / Config, mais s’y rattache.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.services.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"services"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des services significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "services",
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

Chaque élément de `items[]` représente un **service significatif** (*ServiceItem*).

```text
items[] : ServiceItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du service, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Services.

- `kind` : string  
  Type de service parmi un ensemble contrôlé, par exemple :
  - `"httpService"` (facade sur API HTTP),
  - `"domainService"` (service métier combinant plusieurs dataflows),
  - `"repository"` (accès à un ensemble d’agrégats/modèles),
  - `"externalIntegration"` (tracking, outil tiers, etc.),
  - `"cacheService"` (cache interne, store local),
  - `"featureService"` (feature flags, toggles),
  - `"configService"` (lecture de configuration).

- `name` : string  
  Nom logique du service, par exemple :
  - `"CampaignsService"`,
  - `"PromoBoostApiClient"`,
  - `"FeatureFlagService"`,
  - `"UserRepository"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où ce service est défini.

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) correspondant aux vues/composants dépendants de ce service (directement ou via hooks).  
  - Peut être vide pour un service purement technique, mais doit être renseigné dès qu’un lien UI clair existe.

- `serviceSummary` : object  
  Résumé structuré du service, par exemple :
  - `domainScope`: domaine métier adressé (ex. `"campaigns"`, `"users"`, `"analytics"`),
  - `operations`: liste textuelle des méthodes exposées pertinentes (ex. `["fetchCampaigns", "saveCampaign", "deleteCampaign"]`),
  - `dataflows`: noms logiques des dataflows principaux encapsulés,
  - `responsibilities`: description courte des responsabilités (“centralise tous les appels API liés aux campagnes”),
  - `description`: phrase synthétique.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen,
  - `isSharedAcrossPages`: booléen,
  - `isLegacyBoundary`: booléen (frontière entre ancien et nouveau monde),
  - `severity`: `"low" | "medium" | "high"` (impact si ce service est défaillant),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `ownedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) **principalement encapsulés** par ce service.

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` async (issus de `inventory.async.json`) représentant les stratégies temporelles associées à ce service.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) directement liés à ce service (préparation payloads, post‑processing).

- `relatedHookUcrs` : array de string  
  Liste des `ucr` de hooks (issus de `inventory.hooks.json`) utilisant ce service (hooks de data, hooks métier).

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) dont le traitement dépend de ce service.

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui modulent ce service (URL d’endpoint, feature flag, toggles).

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) qui conditionnent l’utilisation du service (ex. chargé uniquement sur certains écrans/routes).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) déclenchés en réponse à des opérations de ce service.

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de services doivent être **uniques** dans `inventory.services.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `ownedDataflowUcrs`, `relatedAsyncUcrs`, `relatedLogicUcrs`, `relatedHookUcrs`, `relatedEventUcrs`, `relatedConfigNames`, `relatedRoutingUcrs`, `relatedEffectUcrs` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le DSL peut inclure des concepts du type :

- `data.serviceFacade`
- `data.repository`
- `data.endpointCluster`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour les identifier.  
Si certaines entrées sont manquantes, l’IA doit :

- repérer les modules dont le nom ou le chemin évoque un service (`services/`, `api/`, `repositories/`, `client`, etc.),
- analyser les fichiers qui concentrent plusieurs appels dataflows,
- documenter les incertitudes dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Services doit :

1. Parcourir le code à partir de `${paths.legacySource}` et suivre les imports vers :
   - services API / HTTP,
   - repositories / facades métiers,
   - services d’intégration (analytics, feature flags).
2. Pour chaque service significatif :
   - identifier ses **opérations publiques** (fonctions exportées, méthodes),
   - repérer quels dataflows sont **encapsulés** ou **orchestrés** par lui,
   - relier ses usages :
     - aux vues (via hooks, composants),
     - aux événements déclencheurs,
     - aux patterns async (retry, parallel, etc.).

### 3.3. Restrictions

L’inventaire Services **ne doit pas** :

- dupliquer toute la configuration technique des clients HTTP (headers, tokens…),
- décrire chaque dataflow en détail (c’est le rôle d’`inventory.dataflows`),
- modéliser la logique métier interne en profondeur (cela relève d’`inventory.logic`).  

Il doit fournir une **carte claire** de la couche services telle qu’elle est vue depuis la page.

---

## 4. 🔗 Relations avec les autres inventaires

- **Services ← Dataflows**
  - Les services encapsulent ou orchestrent des dataflows.  
    Références via `ownedDataflowUcrs`.

- **Services ↔ Async**
  - Les services sont associés à des patterns async (retry, parallel, polling…).  
    Références via `relatedAsyncUcrs`.

- **Services ↔ Logic / Hooks / Events**
  - La logique métier s’appuie sur les services, souvent via des hooks, eux-mêmes déclenchés par des événements.  
    Références via `relatedLogicUcrs`, `relatedHookUcrs`, `relatedEventUcrs`.

- **Services ↔ Config / Routing / Effects**
  - La config, le routing et les effets conditionnent l’utilisation des services (endpoints, scope par route, toasts, tracking).  
    Références via `relatedConfigNames`, `relatedRoutingUcrs`, `relatedEffectUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de services sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `serviceSummary`, `metadata`) sont présents.
- [ ] Les liens vers les autres inventaires (dataflows, async, logic, hooks, events, config, routing, effects) sont cohérents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "services",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "service-CampaignsService-1",
      "kind": "httpService",
      "name": "CampaignsService",
      "sourcePath": "src/packages/promo-boost/services/campaignsService.ts",
      "targetStructureUcrs": ["view-table-campaigns-1", "view-form-campaign-1"],
      "serviceSummary": {
        "domainScope": "campaigns",
        "operations": [
          "fetchCampaigns",
          "fetchCampaignById",
          "saveCampaign",
          "deleteCampaign"
        ],
        "dataflows": [
          "fetchCampaignsList",
          "fetchCampaignDetail",
          "saveCampaignDataflow",
          "deleteCampaignDataflow"
        ],
        "responsibilities": "Centralise tous les appels API liés aux campagnes pour la page CampaignsDetail.",
        "description": "Service HTTP qui expose les opérations principales sur les campagnes (listage, lecture, sauvegarde, suppression)."
      },
      "metadata": {
        "isCritical": true,
        "isSharedAcrossPages": true,
        "isLegacyBoundary": true,
        "severity": "high"
      },
      "ownedDataflowUcrs": [
        "dataflow-fetchCampaignsList-1",
        "dataflow-saveCampaign-1"
      ],
      "relatedAsyncUcrs": [
        "async-retrySaveCampaignWithBackoff-1"
      ],
      "relatedHookUcrs": [
        "hook-useCampaigns-1"
      ],
      "relatedEventUcrs": [
        "event-onClickSaveCampaign-1"
      ],
      "relatedConfigNames": [
        "PROMOBOOST_API_BASE_URL",
        "ENABLE_ADVANCED_CAMPAIGN_API"
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
  "domain": "services",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "service-CampaignsService-1",
      "kind": "httpService",
      "name": "CampaignsService",
      "sourcePath": "src/packages/promo-boost/services/campaignsService.ts",
      "targetStructureUcrs": ["view-unknown-99"],
      "serviceSummary": {
        "domainScope": "campaigns",
        "operations": ["fetchCampaigns"],
        "dataflows": ["fetchCampaignsList"],
        "responsibilities": "Service campagnes.",
        "description": "Service campagnes."
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

- [ ] `domain` est `"services"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de services sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Identifier les services **structurants** pour la page (domaine campagnes, utilisateurs, feature flags…), pas chaque petit helper technique.
- Toujours relier un service :
  - à au moins un dataflow (si possible),
  - à au moins une vue/composant consommateur,
  - à la logique métier ou aux événements clés.
- Utiliser `metadata` et `validation.issues` pour mettre en lumière :
  - les services “god objects” à découper,
  - les services critiques pour la migration (frontière Legacy / nouvelle stack).

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Services*
