# 🧩 Stage 18 – inventory.events
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 17 – inventory.hooks  
**Next :** 19 – inventory.dataflows

---

## 🎯 Objectif

Construire l’**inventaire des événements** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires de logique, conditions et hooks (`inventory.logic.json`, `inventory.conditions.json`, `inventory.hooks.json`) si disponibles,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `event.*`.

L’objectif est de produire un JSON `inventory.events.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **événements utilisateurs** (click, change, submit, keyboard, mouse…),
- les **événements de navigation** (changement de route déclenché par l’UI),
- les **événements métier** (actions significatives côté domaine),
- les liens entre événements, **vues**, **logique**, **conditions**, **hooks**, **dataflows** et **effects**.

Cet inventaire ne détaille pas :

- la logique exécutée (→ `inventory.logic`),
- les conditions associées (→ `inventory.conditions`),
- les effets/side-effects déclenchés (→ `inventory.effects`),
- les flux de données consommés/émis (→ `inventory.dataflows`, `inventory.services`),
- le routing lui-même (→ `inventory.routing`).  

Il se concentre sur **“qui déclenche quoi, où, et comment”** dans l’UI.

---

## ⚙️ Inputs

> Tous les chemins sont dérivés de `project.config.yaml` via `project.*` et `paths.*`.  
> Aucun chemin absolu ne doit être utilisé.

### 1. Configuration projet (en mémoire)

Clés utilisées :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `runtime.regenerateStackGuides`
- `stack.custom`
- `gates.*`
- `stages.*`

---

### 2. Code Legacy (lecture seule)

- `${paths.legacySource}`  
  - Fichier d’entrée principal de la page Legacy.
  - Le stage peut suivre les imports vers :
    - les composants contenant des handlers d’événements (`onClick`, `onChange`, `onSubmit`, etc.),
    - les hooks ou utilitaires qui définissent des callbacks d’événements,
    - les modules de routing / navigation déclenchés par des actions utilisateur.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Events**
  - `${paths.core}/guides-internals/inventory/guide.inventory.events.md`
  - Fournit :
    - l’**objectif** du domaine Events,
    - le **schéma JSON contractuel** de `inventory.events.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure, Logic, Hooks, Effects, Dataflows),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les événements respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `event.*`, par exemple :
  - `event.handler`
  - `event.click`
  - `event.change`
  - `event.submit`
  - `event.keyboard`
  - `event.navigation`
- aider à distinguer :
  - les événements purement UI (click, focus, input…),
  - les événements déclenchant des actions métiers,
  - les événements qui provoquent une navigation.

> Si le bridge ne définit que partiellement `event.*`, utiliser les conventions d’UI (props `onXxx`, listeners, etc.) et documenter les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment les événements devront être projetés (handlers par composant, patterns d’actions, etc.),
    - anticiper les regroupements (ex. centralisation des handlers dans des hooks ou des modules d’actions).

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de gestion d’événements dans la stack cible (ex. pattern “actions” ou “controller”),
    - influencer la granularité des `EventItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - ancrer chaque événement à un élément concret de l’UI.

- **Inventaires recommandés**
  - `inventory.logic.json` (Stage 15)
    - pour lier les événements aux unités de logique invoquées,
  - `inventory.conditions.json` (Stage 16)
    - pour relier les événements aux conditions de déclenchement,
  - `inventory.hooks.json` (Stage 17)
    - pour relier les événements aux hooks qui exposent les callbacks.

- **Autres inventaires optionnels**
  - `inventory.config.json` (flags / settings influençant les handlers),
  - `inventory.dataflows.json` / `inventory.services.json` (événements qui déclenchent des appels data),
  - `inventory.effects.json` (événements qui déclenchent des effets),
  - `inventory.routing.json` (événements qui déclenchent des navigations).

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence des autres inventaires n’est pas bloquante, mais doit être notée dans `validation.issues` si elle limite la qualité des liens.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.events.md`,
- `domain` doit valoir `"events"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (logic, hooks, conditions, dataflows, effects, routing…) doivent être cohérentes,
- JSON strictement sérialisable, sans clés non documentées.

---

## 🧠 Actions

1. **Charger le contexte global**
   - Utiliser les valeurs de configuration déjà en mémoire :
     - `project.name`, `project.pageName`,
     - `paths.root`, `paths.core`, `paths.workspace`, `paths.legacySource`,
     - `paths.stages`,
     - `gates.*`.

2. **Charger les artefacts nécessaires**
   - Lire :
     - `${paths.workspace}/projects/${project.name}/stack/project-structure.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`,
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.conditions.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.hooks.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.events.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - identifier les vues/clusters d’UI avec forte interaction utilisateur (boutons, formulaires, listes interactives…).
   - À partir des autres inventaires (si présents) :
     - indexer `LogicItem`, `ConditionItem`, `HookItem`, `EffectItem`, `DataflowItem`, etc. pour pouvoir faire des liens.
   - À partir du bridge :
     - extraire les patterns `event.*` et les indexer par `dslId`.

4. **Analyser le code Legacy pour les événements**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les props d’événements (`onClick`, `onChange`, `onSubmit`, `onKeyDown`, etc.),
       - les callbacks passés à ces props (fonctions inline ou références),
       - les hooks qui retournent des callbacks utilisés comme handlers,
       - les événements qui déclenchent :
         - de la logique métier,
         - des effets (notifications, logs, etc.),
         - des requêtes data,
         - des navigations (changement de route).
     - pour chaque événement significatif :
       - identifier le **type d’événement** (UI / navigation / métier),
       - identifier la **source UI** (vue/composant),
       - repérer les **cibles logiques** (fonctions, hooks, actions).

5. **Construire les items d’événements**
   - Créer un `EventItem` par événement significatif (voir guide pour le schéma) :
     - définir le `kind` (uiEvent, formEvent, navigationEvent, domainEvent, etc.),
     - donner un `name` logique (ex. `"onClickSaveCampaign"`, `"onSubmitFiltersForm"`),
     - associer des `targetStructureUcrs` (composants/vues qui émettent l’événement),
     - relier aux unités de logique / hooks / conditions / dataflows / effects via les champs prévus,
     - résumer le rôle de l’événement dans `eventSummary`.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (logic, hooks, dataflows, effects, routing…),
     - les événements critiques sont identifiés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.events.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "18",
  "stageName": "inventory.events",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "guidesLoaded": true,
    "bridgeLoaded": true,
    "structureInventoryLoaded": true,
    "legacyParsed": true,
    "itemsBuilt": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

---

## 🧩 Gate

Utiliser exactement l’un des blocs suivants :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

- `Gate ✅` si `inventory.events.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `19-inventory.dataflows.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
