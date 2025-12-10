# 🔧 Guide Inventaire — Hooks (`inventory.hooks`)

*(Domaine d’inventaire : **Hooks** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Hooks** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **hooks React standards** utilisés (`useState`, `useEffect`, `useMemo`, etc.).
2. Les **hooks custom** spécifiques au domaine ou au projet (ex. `useCampaigns`, `usePromoBoostFilters`).
3. Le **rôle de ces hooks** (gestion d’état, orchestration de logique, gestion de data, effets, formulaires, routing, etc.).
4. Leurs **dépendances** (logique, conditions, config, dataflows, effets).
5. Les **vues / composants** qu’ils impactent.

Il répond à la question :

> **“Quels hooks structurent cette page, à quoi servent-ils, et comment s’articulent-ils avec la logique, les données et les effets ?”**

Ce domaine ne :

- ne détaille pas entièrement la logique interne (→ `inventory.logic`),
- ne décompose pas toutes les conditions (→ `inventory.conditions`),
- ne remplace pas les inventaires Dataflows / Services / Effects, mais les relie aux hooks.

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.hooks.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"hooks"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des hooks significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "hooks",
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

Chaque élément de `items[]` représente un **hook** significatif (*HookItem*).

```text
items[] : HookItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du hook, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Hooks.

- `kind` : string  
  Type de hook parmi un ensemble contrôlé, par exemple :
  - `"stateHook"` (useState, useReducer…),
  - `"effectHook"` (useEffect, useLayoutEffect…),
  - `"memoHook"` (useMemo, useCallback),
  - `"customLogicHook"` (hook custom encapsulant de la logique),
  - `"dataHook"` (hook de data fetching, ex. useQuery),
  - `"formHook"`,
  - `"routingHook"`,
  - `"integrationHook"` (store global, analytics, etc.).

- `name` : string  
  Nom du hook au niveau code ou nom logique, par exemple :
  - `"useState"` couplé à un commentaire plus précis,
  - `"useCampaigns"`,
  - `"usePromoBoostFilters"`,
  - `"useUserPermissions"`.

- `sourcePath` : string  
  Chemin du fichier Legacy principal où ce hook est **déclaré** (pour un custom) ou **utilisé** (pour un hook standard, dans le contexte de la page).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) impactés par le hook :
  - composants qui utilisent ce hook,
  - vues dont le comportement est piloté par ce hook.

- `hookSummary` : object  
  Résumé structuré du rôle du hook, par exemple :
  - `role`: string (ex. `"manageCampaignFilters"`, `"fetchCampaigns"`, `"synchronizeUrlWithFilters"`),
  - `inputs`: description des données/états/props en entrée,
  - `outputs`: description des données/états fournis par le hook,
  - `description`: phrase courte expliquant son rôle global.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isCustom`: booléen (hook custom ou non),
  - `isSharedAcrossPages`: booléen,
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de `LogicItem` (issus de `inventory.logic.json`) encapsulés ou orchestrés par ce hook.

- `relatedConditionUcrs` : array de string  
  Liste des `ucr` de `ConditionItem` (issus de `inventory.conditions.json`) dans lesquelles ce hook intervient (ex. conditions internes au hook).

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) utilisés par le hook.

- `relatedDataflowIds` : array de string  
  Identifiants logiques de dataflows / services utilisés dans ce hook (ex. queries/mutations).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) déclenchés ou gérés par ce hook.

- `lifecycle` : string  
  Caractérisation simple de son rôle dans le cycle de vie de la page (ex. `"onMount"`, `"onUpdate"`, `"onUnmount"`, `"continuous"`).

- `complexity` : string  
  Indicateur qualitatif (`"low"`, `"medium"`, `"high"`).

Tout champ optionnel utilisé doit être **documenté** ici.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de hooks doivent être **uniques** dans `inventory.hooks.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- `relatedLogicUcrs`, `relatedConditionUcrs`, `relatedEffectUcrs` ne doivent contenir que des `ucr` valides dans leurs inventaires respectifs (si ceux-ci existent).
- `relatedConfigNames` doit référencer des `configName` valides de `inventory.config.json` si utilisé.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `hooks.*` du DSL peut inclure par exemple :

- `hooks.state`
- `hooks.effect`
- `hooks.memoization`
- `hooks.customLogic`
- `hooks.data`
- `hooks.form`
- `hooks.routing`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître ces concepts dans le code.  
Si certaines entrées sont manquantes, l’IA doit :

- se baser sur les signatures de hooks (`useXxx`),
- analyser leurs usages (state, effect, data, etc.),
- documenter les limites dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Hooks doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour :
   - lister les appels à hooks React standards,
   - repérer les hooks custom importés ou définis,
   - classer les hooks par rôle (state, effect, data, logique métier, formulaires, routing…).
2. Pour chaque hook :
   - identifier les **entrées** (props, config, data, états) et **sorties** (données, callbacks, flags),
   - associer les composants/vues concernés via `targetStructureUcrs`,
   - relier le hook à :
     - de la logique (`relatedLogicUcrs`),
     - des conditions (`relatedConditionUcrs`),
     - de la config (`relatedConfigNames`),
     - de la data (`relatedDataflowIds`),
     - des effets (`relatedEffectUcrs`),
     quand ces liens peuvent être détectés.

### 3.3. Restrictions

L’inventaire Hooks **ne doit pas** :

- se transformer en inventaire complet de logique ou de data (réservés à leurs domaines),
- détailler toutes les branches internes des hooks (cela relève Logic/Conditions),
- répéter à l’identique les inventaires Effects / Dataflows / Services.

Il doit plutôt se positionner comme une **vue “orchestrateur”** :  
quelles briques (logique, data, effets, conditions) sont connectées via quels hooks.

---

## 4. 🔗 Relations avec les autres inventaires

- **Hooks ← Structure**
  - Les hooks sont rattachés à des vues/composants via `targetStructureUcrs`.

- **Hooks ↔ Logic**
  - De nombreux hooks encapsulent ou orchestrent la logique métier.  
    Références possibles via `relatedLogicUcrs`.

- **Hooks ↔ Conditions**
  - Certaines conditions sont internes à des hooks (guards, filtres).  
    Références possibles via `relatedConditionUcrs`.

- **Hooks ↔ Config**
  - Les hooks consomment souvent des flags / settings.  
    Références via `relatedConfigNames`.

- **Hooks ↔ Dataflows / Services**
  - Les hooks de data (`useQuery`, etc.) orchestrent des flux de données.  
    Références via `relatedDataflowIds`.

- **Hooks ↔ Effects**
  - Les hooks de type effect sont intimement liés aux effets.  
    Références via `relatedEffectUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de hooks sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `hookSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "hooks",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "hook-useCampaigns-1",
      "kind": "dataHook",
      "name": "useCampaigns",
      "sourcePath": "src/packages/promo-boost/hooks/useCampaigns.js",
      "targetStructureUcrs": ["view-table-campaigns-1"],
      "hookSummary": {
        "role": "fetchCampaigns",
        "inputs": ["filters actifs", "paramètres de pagination"],
        "outputs": ["liste de campagnes", "état de chargement", "erreur éventuelle"],
        "description": "Gère la récupération des campagnes à afficher dans le tableau, en fonction des filtres et de la pagination."
      },
      "metadata": {
        "isCustom": true,
        "isSharedAcrossPages": true
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
  "domain": "hooks",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "hook-useCampaigns-1",
      "kind": "dataHook",
      "name": "useCampaigns",
      "sourcePath": "src/packages/promo-boost/hooks/useCampaigns.js",
      "targetStructureUcrs": ["view-unknown-99"],
      "hookSummary": {
        "role": "fetchCampaigns",
        "inputs": ["filters actifs", "paramètres de pagination"],
        "outputs": ["liste de campagnes", "état de chargement", "erreur éventuelle"],
        "description": "Gère la récupération des campagnes à afficher."
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

- [ ] `domain` est `"hooks"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de hooks sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne pas lister chaque hook trivial si cela n’apporte pas de valeur (focus sur les hooks structurants).
- Toujours relier les hooks à :
  - des vues (`targetStructureUcrs`),
  - et, si possible, aux autres inventaires (logic, conditions, data, effects).
- Documenter les hooks “god objects” (trop de responsabilités) dans `metadata.notes` et éventuellement `validation.issues`.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Hooks*
