# 🔧 Guide Inventaire — Async (`inventory.async`)

*(Domaine d’inventaire : **Asynchrone / Orchestration temporelle** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Async** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **points asynchrones** (fonctions async, Promises, appels concurrents).
2. Les **stratégies de fiabilité** : retry, backoff, timeout, cancellation.
3. Les **patterns temporels** : séquencement, parallélisme, “race”, polling, debounce, throttle.
4. Les liens entre ces patterns et :
   - les **dataflows** (flux de données),
   - les **services** (facades techniques),
   - les **événements** (déclencheurs),
   - les **hooks**,
   - les **effects** (effets déclenchés après les opérations async),
   - les **vues** (spinners, loaders, état d’attente).

Il répond à la question :

> **“Comment cette page gère-t-elle le temps, l’attente, les erreurs réseau et la concurrence des appels ?”**

Ce domaine ne :

- ne duplique pas les détails des flux de données (`inventory.dataflows`),
- ne décrit pas l’intégralité de la logique métier (`inventory.logic`),
- ne remplace pas les inventaires Services / Effects, mais s’y réfère.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.async.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"async"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des patterns async significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "async",
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

Chaque élément de `items[]` représente un **pattern async significatif** (*AsyncItem*).

```text
items[] : AsyncItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du pattern async, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Async.

- `kind` : string  
  Type de pattern async parmi un ensemble contrôlé, par exemple :
  - `"simpleAsyncCall"` (appel unique, sans orchestration complexe),
  - `"parallelCalls"` (plusieurs appels en parallèle, ex. `Promise.all`),
  - `"sequencedCalls"` (appels en série dépendants),
  - `"retryPattern"` (réessais avec ou sans backoff),
  - `"pollingPattern"` (répétition périodique),
  - `"debouncePattern"`,
  - `"throttlePattern"`,
  - `"racePattern"` (ex. `Promise.race`),
  - `"timeoutPattern"`,
  - `"cancellationPattern"`.

- `name` : string  
  Nom logique du pattern async, par exemple :
  - `"parallelFetchCampaignsAndStats"`,
  - `"retrySaveCampaignWithBackoff"`,
  - `"pollingCampaignsStatus"`,
  - `"debounceFiltersChange"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où ce pattern est implémenté (service, hook, composant).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) des vues/composants impactés (spinners, boutons, sections dépendantes).

- `asyncSummary` : object  
  Résumé structuré du pattern async, par exemple :
  - `strategy`: description courte du pattern (ex. `"parallelCallsWithSharedSpinner"`),
  - `relatedDataflows`: nom logique des dataflows concernés,
  - `ordering`: description de l’ordre (parallèle, séquentiel, mixte),
  - `errorHandling`: résumé de la gestion d’erreur (try/catch, fallback, logs),
  - `cancellation`: présence ou non d’un mécanisme de cancellation,
  - `description`: phrase courte expliquant ce que fait ce pattern et pourquoi.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCritical`: booléen,
  - `hasExplicitRetry`: booléen,
  - `hasTimeout`: booléen,
  - `hasCancellation`: booléen,
  - `severity`: `"low" | "medium" | "high"`,
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedDataflowUcrs` : array de string  
  Liste des `ucr` de dataflows (issus de `inventory.dataflows.json`) associés.

- `relatedServiceUcrs` : array de string  
  Liste des `ucr` de services (issus de `inventory.services.json`) utilisés dans ce pattern.

- `relatedEventUcrs` : array de string  
  Liste des `ucr` d’événements (issus de `inventory.events.json`) déclencheurs.

- `relatedHookUcrs` : array de string  
  Liste des `ucr` de hooks (issus de `inventory.hooks.json`) qui orchestrent ce pattern.

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) déclenchés en réponse.

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` de routing (issus de `inventory.routing.json`) concernés (ex. cancellation sur changement de route).

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) qui influencent la stratégie async (timeouts paramétrés, feature flags, etc.).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` async doivent être **uniques** dans `inventory.async.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `related*` ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `effect.async` du DSL (et les concepts `data.*` liés) couvre par exemple :

- `effect.async.dataQuery`
- `effect.async.dataMutation`
- `effect.async.parallel`
- `effect.async.retry`
- `effect.async.polling`
- `effect.async.debounce`
- `effect.async.throttle`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître certains de ces concepts.  
Si certaines entrées sont manquantes, l’IA doit :

- se baser sur les constructions asynchrones standard (Promises, `async/await`, helpers maison),
- repérer les patterns temporels (Promise.all, setInterval, debounce, etc.),
- documenter les limites dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Async doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour repérer :
   - les fonctions `async` et leurs `await`,
   - les chaînes de Promises (`.then`, `.catch`, `.finally`),
   - les appels à `Promise.all`, `Promise.race`, etc.,
   - les patterns de retry / backoff explicites,
   - les timers (setTimeout, setInterval) utilisés pour de l’async métier,
   - les helpers debounce/throttle.
2. Pour chaque pattern significatif :
   - déterminer `kind`,
   - identifier les dataflows/services impliqués,
   - identifier les événements déclencheurs,
   - rattacher les vues impactées (spinners, messages d’erreur, sections dépendantes).

### 3.3. Restrictions

L’inventaire Async **ne doit pas** :

- dupliquer toute la configuration technique des clients HTTP,
- modéliser chaque `await` trivial s’il n’apporte aucune logique temporelle intéressante,
- devenir un inventaire général de dataflows (c’est le rôle d’`inventory.dataflows`).  

Il doit se concentrer sur les **patterns temporels pertinents** pour la fiabilité et l’expérience utilisateur.

---

## 4. 🔗 Relations avec les autres inventaires

- **Async ← Structure**
  - Les vues sont impactées par l’async (spinners, états de chargement, blocs désactivés).

- **Async ↔ Dataflows**
  - Les dataflows s’exécutent selon des patterns async.  
    Références via `relatedDataflowUcrs`.

- **Async ↔ Services**
  - Les services encapsulent souvent les appels async.  
    Références via `relatedServiceUcrs`.

- **Async ↔ Events**
  - Les événements déclenchent des séquences async.  
    Références via `relatedEventUcrs`.

- **Async ↔ Hooks**
  - Les hooks orchestrent l’async (states loading/error, etc.).  
    Références via `relatedHookUcrs`.

- **Async ↔ Effects / Routing / Config**
  - Les effets, le routing et la config modulent le comportement async (cancellation, timeouts, etc.).

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` async sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `asyncSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "async",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "async-parallelFetchCampaignsAndStats-1",
      "kind": "parallelCalls",
      "name": "parallelFetchCampaignsAndStats",
      "sourcePath": "src/packages/promo-boost/hooks/useCampaignsWithStats.ts",
      "targetStructureUcrs": ["view-table-campaigns-1", "view-panel-stats-1"],
      "asyncSummary": {
        "strategy": "parallelCallsWithSharedSpinner",
        "relatedDataflows": ["fetchCampaignsList", "fetchCampaignsStats"],
        "ordering": "parallel",
        "errorHandling": "catch global avec affichage d’un message d’erreur et fallback sur données partielles",
        "cancellation": "aucune cancellation explicite",
        "description": "Lance en parallèle la récupération de la liste des campagnes et des statistiques associées, avec un spinner commun."
      },
      "metadata": {
        "isCritical": true,
        "hasExplicitRetry": false,
        "hasTimeout": false,
        "hasCancellation": false,
        "severity": "medium"
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
  "domain": "async",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "async-parallelFetchCampaignsAndStats-1",
      "kind": "parallelCalls",
      "name": "parallelFetchCampaignsAndStats",
      "sourcePath": "src/packages/promo-boost/hooks/useCampaignsWithStats.ts",
      "targetStructureUcrs": ["view-unknown-99"],
      "asyncSummary": {
        "strategy": "parallelCallsWithSharedSpinner",
        "relatedDataflows": ["fetchCampaignsList", "fetchCampaignsStats"],
        "ordering": "parallel",
        "errorHandling": "catch global",
        "cancellation": "aucune",
        "description": "Lance en parallèle la récupération des campagnes et des stats."
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

- [ ] `domain` est `"async"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` async sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne pas surcharger l’inventaire avec chaque petit `await` dénué d’enjeu fonctionnel.
- Se concentrer sur les **patterns temporels structurants** (parallel, retry, polling, debounce, etc.).
- Toujours chercher à relier l’async :
  - à des dataflows concrets,
  - à des événements déclencheurs,
  - à des vues impactées (chargement, erreurs, indisponibilité temporaires).
- Utiliser `metadata.severity` et `validation.issues` pour mettre en lumière :
  - les patterns risqués,
  - l’absence de gestion d’erreur ou de cancellation là où ce serait critique.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Async*
