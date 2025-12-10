# 🔧 Guide Inventaire — Events (`inventory.events`)

*(Domaine d’inventaire : **Événements** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Events** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **événements utilisateurs** (click, change, submit, keyboard, etc.).
2. Les **événements de navigation** déclenchés par l’UI.
3. Les **événements métiers** (actions significatives du point de vue fonctionnel).
4. Les liens entre ces événements et :
   - les **vues / composants**,
   - la **logique** (règles métier),
   - les **conditions**,
   - les **hooks**,
   - les **dataflows / services**,
   - les **effets** (notifications, logs, etc.),
   - le **routing**.

Il répond à la question :

> **“Quels événements existent dans cette page, qui les émet, que déclenchent-ils et sur quoi reposent-ils ?”**

Ce domaine ne :

- ne redécrit pas la logique métier (→ `inventory.logic`),
- ne remplace pas l’inventaire des effets (→ `inventory.effects`),
- ne modélise pas directement les flux de données (→ `inventory.dataflows`, `inventory.services`),
- ne définit pas la structure de l’UI (→ `inventory.structure`).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.events.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"events"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des événements significatifs (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "events",
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

Chaque élément de `items[]` représente un **événement significatif** (*EventItem*).

```text
items[] : EventItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’événement, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Events.

- `kind` : string  
  Type d’événement parmi un ensemble contrôlé, par exemple :
  - `"uiEvent"` (click, change, focus…),
  - `"formEvent"` (submit, reset),
  - `"navigationEvent"` (changement de route déclenché par l’UI),
  - `"domainEvent"` (action métier logique, ex. “campagne sauvegardée”),
  - `"keyboardEvent"`,
  - `"pointerEvent"`.

- `name` : string  
  Nom logique de l’événement, par exemple :
  - `"onClickSaveCampaign"`,
  - `"onSubmitFiltersForm"`,
  - `"onClickOpenCampaignDetail"`,
  - `"campaignSavedEvent"` (événement métier logique).

- `sourcePath` : string  
  Chemin du fichier Legacy principal où l’événement est câblé (composant, hook, module).

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) d’où **part l’événement** (composants/vues émetteurs).

- `eventSummary` : object  
  Résumé structuré de l’événement, par exemple :
  - `triggerType`: `"click" | "change" | "submit" | "keyboard" | "navigation" | "custom"`,
  - `inputs`: description des données/états utilisés au moment du déclenchement,
  - `outcomes`: description des actions principales déclenchées (logique, data, navigation),
  - `description`: phrase courte (“Au clic sur le bouton 'Sauvegarder', déclenche la validation du formulaire et l’appel API de sauvegarde.”).

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `isUserInitiated`: booléen,
  - `isCritical`: booléen (événement sensible type paiement, sauvegarde…),
  - `isDebouncedOrThrottled`: booléen,
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `relatedHookUcrs` : array de string  
  Liste des `ucr` de `HookItem` (issus de `inventory.hooks.json`) qui exposent ou encapsulent ce handler.

- `relatedLogicUcrs` : array de string  
  Liste des `ucr` de `LogicItem` (issus de `inventory.logic.json`) appelés par ce handler.

- `relatedConditionUcrs` : array de string  
  Liste des `ucr` de `ConditionItem` (issus de `inventory.conditions.json`) évalués lors du traitement de l’événement.

- `relatedConfigNames` : array de string  
  Liste des `configName` (issus de `inventory.config.json`) utilisés dans ce handler.

- `relatedDataflowIds` : array de string  
  Identifiants logiques de dataflows / services déclenchés par cet événement (ex. appels API).

- `relatedEffectUcrs` : array de string  
  Liste des `ucr` d’effets (issus de `inventory.effects.json`) déclenchés (ex. toast, log, tracking).

- `relatedRoutingUcrs` : array de string  
  Liste des `ucr` d’éléments de routing (issus de `inventory.routing.json`) affectés (ex. navigation vers une route).

- `severity` : string  
  Impact potentiel d’un dysfonctionnement sur cet événement (`"low"`, `"medium"`, `"high"`).

Tout champ optionnel utilisé doit être **documenté** ici et cohérent avec les autres inventaires.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` d’événements doivent être **uniques** dans `inventory.events.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Les champs `related*` (hooks, logic, conditions, dataflows, effects, routing) ne doivent contenir que des identifiants valides dans leurs inventaires respectifs (si ceux-ci existent).
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Le domaine `event.*` du DSL peut inclure par exemple :

- `event.handler`
- `event.click`
- `event.change`
- `event.submit`
- `event.keyboard`
- `event.navigation`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) fournit les patterns pour reconnaître ces concepts dans le code.  
Si certaines entrées sont manquantes, l’IA doit :

- s’appuyer sur les props `onXxx` des composants (ou équivalents),
- reconnaître les patterns classiques (handlers passés à des composants UI, hooks qui renvoient des callbacks),
- documenter la limitation dans `validation.issues`.

### 3.2. Règles d’analyse

L’inventaire Events doit :

1. Parcourir le code à partir de `${paths.legacySource}` pour :
   - repérer les points de câblage d’événements (props `onClick`, `onChange`, `onSubmit`…) dans l’UI,
   - suivre les références de handlers vers leurs définitions (fonctions, hooks),
   - identifier les événements qui entraînent :
     - des modifications d’état/logique,
     - des appels à des dataflows / services,
     - des effets (notifications, logs),
     - des navigations (changement de route).
2. Pour chaque événement significatif :
   - déterminer `kind` et `triggerType`,
   - identifier les vues émettrices via `targetStructureUcrs`,
   - lier autant que possible :
     - au hook qui fournit le handler (`relatedHookUcrs`),
     - à la logique métier (`relatedLogicUcrs`),
     - à des conditions (`relatedConditionUcrs`),
     - à la data (`relatedDataflowIds`),
     - aux effets (`relatedEffectUcrs`),
     - au routing (`relatedRoutingUcrs`).

### 3.3. Restrictions

L’inventaire Events **ne doit pas** :

- dupliquer intégralement le contenu des inventaires Logic / Dataflows / Effects,
- descendre dans le détail technique de chaque expression (reste au niveau de l’unité “événement”),
- lister tous les événements triviaux sans impact fonctionnel réel (focus sur les événements structurant l’expérience).

En cas de handlers “god functions” (beaucoup de responsabilités) :

- le signaler dans `metadata.notes`,
- envisager un marquage `severity: "high"` si l’impact fonctionnel est fort.

---

## 4. 🔗 Relations avec les autres inventaires

- **Events ← Structure**
  - Les événements sont attachés à des vues/composants via `targetStructureUcrs`.

- **Events ↔ Hooks**
  - De nombreux handlers sont fournis par des hooks custom ou standard.  
    Références possibles via `relatedHookUcrs`.

- **Events ↔ Logic**
  - Les événements déclenchent souvent de la logique métier.  
    Références via `relatedLogicUcrs`.

- **Events ↔ Conditions**
  - Certaines conditions sont évaluées dans le cadre d’un événement (guards, validations).  
    Références via `relatedConditionUcrs`.

- **Events ↔ Config**
  - Certains événements ne se déclenchent ou n’agissent que selon des flags/settings.  
    Références via `relatedConfigNames`.

- **Events ↔ Dataflows / Services**
  - De nombreux événements sont à l’origine de requêtes API.  
    Références via `relatedDataflowIds`.

- **Events ↔ Effects**
  - Les événements déclenchent des effets (toasts, logs, tracking…).  
    Références via `relatedEffectUcrs`.

- **Events ↔ Routing**
  - Certains événements déclenchent des navigations.  
    Références via `relatedRoutingUcrs`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` d’événements sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `kind`, `name`, `sourcePath`, `targetStructureUcrs`, `eventSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "events",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "event-onClickSaveCampaign-1",
      "kind": "uiEvent",
      "name": "onClickSaveCampaign",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-button-saveCampaign-1"],
      "eventSummary": {
        "triggerType": "click",
        "inputs": ["état du formulaire de campagne", "feature flag ENABLE_ADVANCED_PROMOBOOST"],
        "outcomes": [
          "validation de la campagne en mémoire",
          "appel API POST /campaigns/save",
          "affichage d’un toast de succès ou d’erreur"
        ],
        "description": "Au clic sur le bouton 'Sauvegarder', valide le formulaire de campagne, envoie les données au backend et affiche un feedback à l’utilisateur."
      },
      "metadata": {
        "isUserInitiated": true,
        "isCritical": true,
        "isDebouncedOrThrottled": false
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
  "domain": "events",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "event-onClickSaveCampaign-1",
      "kind": "uiEvent",
      "name": "onClickSaveCampaign",
      "sourcePath": "src/packages/promo-boost/components/campaignsDetail/index.js",
      "targetStructureUcrs": ["view-unknown-99"],
      "eventSummary": {
        "triggerType": "click",
        "inputs": ["état du formulaire de campagne"],
        "outcomes": [
          "validation de la campagne en mémoire",
          "appel API POST /campaigns/save"
        ],
        "description": "Au clic sur le bouton 'Sauvegarder', valide la campagne."
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

- [ ] `domain` est `"events"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` d’événements sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se concentrer sur les événements **structurels** pour le parcours utilisateur (clics principaux, submits, navigations…).
- Relier autant que possible les événements à :
  - la logique (pour comprendre les actions métiers),
  - la data (pour comprendre ce qui est déclenché),
  - les effets et le routing (pour comprendre l’expérience globale).
- Utiliser `metadata` et `validation.issues` pour signaler :
  - les handlers trop complexes,
  - les événements critiques mal structurés,
  - les comportements difficiles à interpréter.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Events*
