# 🔧 Guide Inventaire — Dataflows (`inventory.dataflows`)

*(Domaine d’inventaire : **Dataflows** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Dataflows** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **flux de données** qui alimentent ou modifient l’état de la page :
   - lectures (queries, chargement initial, rafraîchissements),
   - écritures (mutations, sauvegardes, suppressions),
   - flux réactifs (subscriptions, websockets, events bus…).
2. Les **sources de données** (APIs HTTP, services internes, stores globaux, caches).
3. Les **cibles** : états locaux, stores, vues, modèles métier.
4. Les **entrées et sorties** de chaque dataflow (données d’entrée, payloads, résultats, erreurs).
5. Les liens entre dataflows et :
   - événements (qui les déclenchent),
   - hooks (qui les orchestrent),
   - logique métier (qui consomme/transforme les données),
   - services / async / routing.

Il répond à la question :

> **“Quels flux de données existent dans cette page, d’où viennent-ils, où vont-ils, et à quels moments sont-ils déclenchés ?”**

Ce domaine ne :

- ne décrit pas en détail les politiques d’async (retries, backoff, parallélisme → `inventory.async`),
- ne remplace pas l’inventaire des services techniques (`inventory.services`),
- ne se substitue pas aux inventaires Logic / Hooks / Events, mais les relie via les flux de données.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.dataflows.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"dataflows"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des dataflows significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "dataflows",
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

Chaque élément de `items[]` représente un **dataflow significatif** (*DataflowItem*).

```text
items[] : DataflowItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du dataflow, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Dataflows.

- `kind` : string  
  Type de dataflow parmi un ensemble contrôlé, par exemple :
  - `"query"` (lecture de données distante),
  - `"mutation"` (écriture / mise à jour distante),
  - `"subscription"` (flux réactif, websocket, event stream),
  - `"cacheSync"` (synchronisation cache/store),
  - `"localPersistence"` (localStorage/sessionStorage lié au métier),
  - `"derivedDataflow"` (flux dérivé combinant plusieurs sources).

- `name` : string  
  Nom logique du dataflow, par exemple :
  - `"fetchCampaignsList"`,
  - `"saveCampaign"`,
  - `"deleteCampaign"`,
  - `"loadUserPermissions"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où ce dataflow est **déclaré** ou **orchestré** (service, hook de data, module métier).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) des vues/composants **consommateurs** de ce dataflow.

- `dataflowSummary` : object  
  Résumé structuré du dataflow, par exemple :
  - `direction`: `"read" | "write" | "readWrite"`,
  - `source`: description de la source (API, service, store, cache…),
  - `targets`: description des destinations (états, stores, vues),
  - `inputs`: paramètres ou contexte nécessaires (filtres, IDs, pagination…),
  - `outputs`: types de données retournées (liste de campagnes, détail d’un élément, etc.),
  - `description`: phrase courte (“Récupère la liste paginée des campagnes filtrées pour alimenter le tableau principal.”).

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen (true si dataflow sensible : paiement, sauvegarde critique, etc.),
  - `isCached`: booléen,
  - `frequency`: string (ex. `"onDemand"`, `"onMount"`, `"polling"`),
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) qui déclenchent ce dataflow.

- `relatedHookUcrs` : array de string  
  Liste des `ucr` de hooks (issus de `inventory.hooks.json`) qui orchestrent ce dataflow (ex. hook de data).

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de logique (issus de `inventory.logic.json`) qui consomment les données de ce dataflow ou en préparent les payloads.

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) représentant des facades techniques pour ce dataflow.

- `relatedAsyncUcrs` : array de string  
  Liste des `ucr` d’éléments asynchrones (issus de `inventory.async.json`) décrivant les stratégies associées (retry, timeout, parallélisme).

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) dont dépend ce dataflow (ex. dataflow déclenché sur navigation).

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui influencent ce dataflow (feature flag, endpoint override, etc.).

- `severity` : string  
  Impact potentiel d’un dysfonctionnement sur ce dataflow (`"low"`, `"medium"`, `"high"`).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de dataflows doivent être **uniques** dans `inventory.dataflows.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `related*` (events, hooks, logic, services, async, routing, config) ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `data.*` du DSL peut inclure par exemple :

- `data.query`
- `data.mutation`
- `data.endpoint`
- `data.cache`
- `data.subscription`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître ces concepts dans le code.  
Si certaines entrées sont manquantes, l’IA doit :

- détecter les appels aux clients HTTP / services connus,
- repérer les hooks de data (`useQuery`, `useMutation`, hooks maison),
- reconnaître les usages de stores/caches liés aux données métier,
- documenter les limites dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Dataflows doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour repérer :
   - les appels API (fetch/axios/clients maison),
   - les hooks de data, y compris ceux qui enveloppent les clients HTTP,
   - les interactions avec des stores globaux / caches au titre de data layer.
2. Pour chaque dataflow significatif :
   - déterminer `kind` (query/mutation/subscription…),
   - identifier :
     - les **sources** (endpoint, service, store),
     - les **cibles** (états, vues, modèles métier),
     - les **entrées** (filtres, IDs, payloads),
     - les **sorties** (types de données, structure attendue),
   - relier le dataflow aux événements, hooks, logique, services, async, routing quand c’est possible.

### 3.3. Restrictions

L’inventaire Dataflows **ne doit pas** :

- dupliquer intégralement la configuration des clients HTTP (headers, bas niveau),
- se transformer en documentation exhaustive des endpoints (c’est un inventaire de flux, pas de contrat API détaillé),
- réimplémenter la logique métier (cela relève de `inventory.logic`).  

Il doit fournir une **vue synthétique mais précise** des flux de données clefs pour la page.

---

## 4. 🔗 Relations avec les autres inventaires

- **Dataflows ← Structure**
  - Rattachement aux vues/composants via `targetStructureUcrs`.

- **Dataflows ↔ Events**
  - Les événements déclenchent souvent les dataflows (submit, clic sur un bouton, etc.).  
    Références via `relatedEventUcrs`.

- **Dataflows ↔ Hooks**
  - Les hooks de data orchestrent des dataflows.  
    Références via `relatedHookUcrs`.

- **Dataflows ↔ Logic**
  - La logique métier consomme et prépare les données des dataflows.  
    Références via `relatedLogicUcrs`.

- **Dataflows ↔ Services / Async / Routing / Config**
  - Ces domaines décrivent :
    - les facades techniques (services),
    - les stratégies asynchrones,
    - l’impact de la route,
    - les paramètres de config qui pilotent les dataflows.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de dataflows sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `dataflowSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "dataflows",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "dataflow-fetchCampaignsList-1",
      "kind": "query",
      "name": "fetchCampaignsList",
      "sourcePath": "src/packages/promo-boost/services/campaignsService.ts",
      "targetStructureUcrs": ["view-table-campaigns-1"],
      "dataflowSummary": {
        "direction": "read",
        "source": "GET /api/campaigns",
        "targets": ["state campaignsList", "vue table des campagnes"],
        "inputs": ["filtres actifs", "paramètres de pagination"],
        "outputs": ["liste paginée de campagnes"],
        "description": "Récupère la liste paginée des campagnes en fonction des filtres et alimente la table principale."
      },
      "metadata": {
        "isCritical": true,
        "isCached": true,
        "frequency": "onDemand"
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

### 6.2. Exemple invalide (commenté)

```json
{
  "domain": "dataflows",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "dataflow-fetchCampaignsList-1",
      "kind": "query",
      "name": "fetchCampaignsList",
      "sourcePath": "src/packages/promo-boost/services/campaignsService.ts",
      "targetStructureUcrs": ["view-unknown-99"],
      "dataflowSummary": {
        "direction": "read",
        "source": "GET /api/campaigns",
        "targets": ["vue table des campagnes"],
        "inputs": ["filtres actifs"],
        "outputs": ["liste de campagnes"],
        "description": "Récupère la liste des campagnes."
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

- [ ] `domain` est `"dataflows"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de dataflows sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les **dataflows structurants** pour la page (chargement initial, actions majeures), pas sur chaque micro-appel technique.
- Toujours relier un dataflow à :
  - au moins une vue (`targetStructureUcrs`),
  - si possible à un événement (`relatedEventUcrs`), un hook (`relatedHookUcrs`) et une logique (`relatedLogicUcrs`).
- Utiliser `metadata` et `validation.issues` pour signaler :
  - les dataflows critiques,
  - les patterns anti-pattern (duplication d’appels, payloads incohérents, etc.).

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Dataflows*
