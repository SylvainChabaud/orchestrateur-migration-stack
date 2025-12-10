# 🧭 Guide Mapping — Template générique (`mapping.<domain>`)

*(Gabarit pour tous les guides de mapping : structure, layout, styles, i18n, config, logic, conditions, hooks, events, dataflows, async, services, routing, effects, actions, tests, summary…)*

---

## 1. 🎯 Rôle des mappings (Phase 2 — Interprétation)

La **Phase 1** a produit des **inventaires DSL** : une description neutre et canonique du Legacy, basée sur :

- le **DSL interne** (concepts `structure.*`, `logic.*`, `event.*`, `data.*`, `effect.*`, `action.*`, `test.*`, etc.),
- des **UCR** stables (identifiants canoniques) pour chaque élément important,
- des fichiers `inventory.<domain>.json` versionnés dans `${paths.workspace}`.

La **Phase 2** (mappings) ne lit plus directement le code Legacy.  
Elle doit :

> **Projeter les inventaires DSL vers la stack cible décrite par :**  
> - la configuration de stack (`*.stack.yaml`),  
> - les guides de stack générés (Phase 0),  
> - la structure cible de projet (`project-structure.json`).  

Elle reste **agnostique de la technologie** : la stack cible peut être React, Vue, Angular, Nest, services backend, BFF, micro-front, etc.  
Les mappings décrivent *où* et *comment* chaque UCR DSL doit vivre dans la stack cible, sans forcer un framework particulier.

---

## 2. 📦 Format JSON racine commun à tous les mappings

Tous les fichiers `mapping.<domain>.json` suivent un schéma racine commun :

- `domain` : string — identifiant du domaine de mapping (ex. `"structure"`, `"logic"`, `"events"`…).
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`).
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`).
- `items` : array de `MappingItem` — une entrée par UCR (ou groupe d’UCR) issu de l’inventaire.
- `validation` : object — statut de validation du mapping lui-même.

Exemple minimal de racine :

```json
{
  "domain": "structure",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Chaque guide de mapping concret (`guide.mapping.<domain>.md`) doit :

- **spécialiser** la signification de `domain`,
- préciser les contraintes sur `items[]`,
- définir la forme exacte de `toStack` pour ce domaine.

---

## 3. 🔗 Schéma générique d’un `MappingItem`

Un `MappingItem` relie :

- un **UCR DSL** (issu d’un inventaire)  
- à une **projection dans la stack cible** (fichiers, artefacts, patterns, responsabilités).

Schéma générique :

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.<domain>.json",
    "domain": "<domain>",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "hints": []
  },
  "relations": {
    "structureUcrs": [],
    "logicUcrs": [],
    "eventUcrs": [],
    "dataflowUcrs": [],
    "asyncUcrs": [],
    "serviceUcrs": [],
    "routingUcrs": [],
    "effectUcrs": [],
    "actionUcrs": [],
    "testUcrs": [],
    "configNames": []
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.1. Champs obligatoires

- `ucr` : string  
  - Identifiant canonique du mapping, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans `mapping.<domain>.json`.
  - Recommandation : dérivé de l’UCR inventaire ou clairement lié à lui.

- `fromDsl` : string  
  - Concept DSL source (ex. `"structure.viewNode"`, `"logic.businessRule"`, `"event.click"`, `"data.query"`, `"action.userAction"`, `"test.integration"`).
  - Doit exister dans la spécification du DSL.

- `sourceInventoryRef` : object  
  - **Ancre** le mapping sur l’inventaire Phase 1.
  - Champs minimaux :
    - `file` : nom du fichier inventaire source (ex. `"inventory.structure.json"`),
    - `domain` : domaine inventaire (ex. `"structure"`),
    - `itemUcr` : UCR de l’élément inventorié.

- `toStack` : object  
  - Projection vers la stack cible.  
  - Le contenu exact est défini par chaque guide de mapping, mais il doit au minimum contenir :
    - `stackKind` : type d’artefact dans la stack cible, par ex. :
      - `"uiComponent"`, `"uiView"`, `"uiLayout"`,  
      - `"domainService"`, `"applicationService"`, `"repository"`,  
      - `"workflowNode"`, `"job"`, `"schedulerTask"`,  
      - `"route"`, `"routerConfig"`,  
      - `"testCase"`, `"testSuite"`, etc.
    - `targetId` : identifiant logique dans la stack cible (nom du composant, du service, de la route, etc.).
    - `targetPath` : chemin relatif dans l’arborescence de la stack cible (projet cible, pas Legacy).
    - `targetLayer` : couche logique (ex. `"presentation"`, `"application"`, `"domain"`, `"infrastructure"`).
  - D’autres champs (`targetTechnology`, `targetPattern`, `hints`) sont optionnels mais recommandés.

- `metadata` : object  
  - Informations complémentaires :
    - `isCritical` : booléen,
    - `priority` : `"low" | "normal" | "high"`,
    - `notes` : string libre.

### 3.2. Champs optionnels suggérés

- `relations` : object  
  - Permet de relier le mapping courant à des **UCR d’autres domaines** :
    - `structureUcrs`, `logicUcrs`, `eventUcrs`, `dataflowUcrs`, `asyncUcrs`, `serviceUcrs`, `routingUcrs`, `effectUcrs`, `actionUcrs`, `testUcrs`, `configNames`.
  - Ce bloc est facultatif mais très utile pour :
    - suivre les dépendances inter-domaines,
    - guider les phases de génération,
    - fournir des rapports d’impact.

Chaque guide concret est libre :

- de **restreindre** certains champs,
- d’**ajouter** des sous-champs à `toStack` pertinents pour le domaine concerné (ex. pour `routing` : `routePath`, `routeParams`, `guardKind`, etc.),
- tant que le contrat commun est respecté.

---

## 4. ⚙️ Entrées communes attendues (Phase 2)

Tout stage de mapping (30–46) travaille au minimum avec :

### 4.1. Configuration projet

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource` (uniquement pour `sourceEntry`)
- `paths.stages`
- `stack.custom` (ex. `${paths.core}/configs/stacks/my-custom-enterprise.stack.yaml`)
- `gates.*`, `stages.*`

### 4.2. Artefacts Phase 0 (obligatoires)

- **Bridge Legacy → DSL**  
  `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- **Structure cible du projet**  
  `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- **Guides de stack générés** (optionnels mais recommandés)  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md`

### 4.3. Inventaires Phase 1

Selon le domaine de mapping, les fichiers suivants (dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/`) sont utilisés :

- `inventory.<domain>.json` (inventaire primaire pour ce mapping),
- `inventory.structure.json` (presque toujours nécessaire),
- les inventaires voisins (logic, events, dataflows, async, services, routing, effects, actions, tests, config…),
- `inventories-summary.json` (vision globale de complétude).

### 4.4. Guides internes

Dans `${paths.core}/guides-internals/` :

- `guide.ucr.md` — contrat UCR global,
- `guide.dsl.*.md` (ou équivalent) — description des concepts DSL,
- `guide.mapping.<domain>.md` — ce guide concret, dérivé du présent template.

---

## 5. 🧠 Règles d’interprétation génériques (tous mappings)

1. **Ne jamais re-scanner le Legacy en Phase 2**  
   - La source de vérité fonctionnelle est l’inventaire DSL (Phase 1).
   - Le Legacy n’est utilisé que comme référence dans `sourceEntry`.

2. **Rester stack-agnostique dans la logique, stack-spécifique dans `toStack`**  
   - La logique de mapping ne suppose pas que la stack cible est React, Vue ou autre.  
   - `toStack.targetTechnology` et `toStack.targetPattern` peuvent refléter une stack concrète (ex. `"react"`, `"vue"`, `"nest"`, `"rest-api"`, `"lambda"`…), mais le contrat du mapping reste générique.

3. **Toujours passer par les artefacts Phase 0**  
   - Décisions de mapping guidées par :
     - `project-structure.json` (arborescence cible),
     - guides de stack (`guide.stack-*.md`),
     - configuration de stack (`*.stack.yaml`).

4. **Conserver la traçabilité UCR**  
   - Chaque élément mappé doit se référer à l’UCR inventaire (`sourceInventoryRef.itemUcr`).
   - Aucun `MappingItem` ne doit être complètement “hors sol”.

5. **Ne pas dupliquer la logique métier ou les données**  
   - Les mappings décrivent des **projections** (où et comment), pas des réécritures fonctionnelles.

---

## 6. 🔗 Relations typiques entre mappings

Les guides concrets peuvent s’appuyer sur ces relations pour expliquer les liens entre domaines :

- `mapping.structure` : support de base pour tous les autres mappings (où vivent les vues / composants / pages).
- `mapping.logic` / `mapping.events` / `mapping.effects` : s’ancrent dans la structure.
- `mapping.dataflows` / `mapping.async` / `mapping.services` : décrivent où vivent les flux de données et les services dans la stack cible.
- `mapping.routing` : projette la navigation et les routes.
- `mapping.actions` : agrège structure + events + logic + dataflows + routing + effects, sous forme de use cases stack-cible.
- `mapping.tests` : projette la stratégie de tests vers la stack finale (où vivent les tests, quel type, quel périmètre).

Ces relations doivent apparaître dans le bloc `relations` quand elles sont pertinentes.

---

## 7. 🧪 Validation générique des mappings

Avant d’accepter un mapping (quel que soit le domaine), l’IA doit vérifier au minimum :

- [ ] `domain` correspond au domaine actuel (structure, logic, etc.).  
- [ ] `pageName` et `sourceEntry` sont corrects.  
- [ ] Chaque `MappingItem` possède un `ucr` unique.  
- [ ] `fromDsl` est un concept valide du DSL.  
- [ ] `sourceInventoryRef.file` et `sourceInventoryRef.itemUcr` existent réellement dans l’inventaire concerné.  
- [ ] `toStack.stackKind`, `toStack.targetId`, `toStack.targetPath`, `toStack.targetLayer` sont renseignés.  
- [ ] Le JSON est strictement valide (pas de trailing commas, pas de clés inconnues).  
- [ ] `validation.status` ∈ {`"valid"`, `"rejected"`}.  
- [ ] `validation.issues` décrit clairement les problèmes si `status = "rejected"`.

---

## 8. 📘 Exemple générique de mapping (structure)

Exemple illustratif (stack cible fictive) :

```json
{
  "domain": "structure",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-structure-viewPage-CampaignsDetail-1",
      "fromDsl": "structure.viewNode",
      "sourceInventoryRef": {
        "file": "inventory.structure.json",
        "domain": "structure",
        "itemUcr": "view-page-CampaignsDetail-1"
      },
      "toStack": {
        "stackKind": "uiView",
        "targetId": "CampaignsDetailView",
        "targetPath": "apps/web/src/pages/campaigns/detail/CampaignsDetailView.tsx",
        "targetLayer": "presentation",
        "targetTechnology": "react",
        "targetPattern": "pageComponent",
        "hints": [
          "Use layout MainLayout from stack-guides",
          "Expose this view as entrypoint route for /campaigns/:id"
        ]
      },
      "relations": {
        "routingUcrs": ["routing-route-campaignsDetail-1"],
        "actionUcrs": ["action-saveCampaign-1", "action-duplicateCampaign-1"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Main detail page view for campaigns."
      }
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Dans une stack non-React, seuls `targetPath`, `targetLayer`, `stackKind`, `targetPattern` et `targetTechnology` changeraient, le contrat restant identique.

---

## 9. 📋 Checklist pour créer un guide de mapping concret

Pour chaque `guide.mapping.<domain>.md`, à partir de ce template :

- [ ] Spécifier clairement l’**objectif** du mapping pour ce domaine.  
- [ ] Définir les champs obligatoires de `toStack` pour ce domaine (ex. routes, services, hooks, jobs…).  
- [ ] Documenter les relations `relations.*` pertinentes.  
- [ ] Lister les inventaires requis / recommandés.  
- [ ] Lister les parties des stack-guides / project-structure utilisées.  
- [ ] Donner au moins un **exemple JSON réaliste**.  
- [ ] Ajouter une checklist spécifique de validation pour ce domaine.

---

© 2025 — ai-orchestrator-v4  
*Template générique des guides de mapping (Phase 2 — Interprétation)*
