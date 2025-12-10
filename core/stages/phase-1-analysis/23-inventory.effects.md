# 🧩 Stage 23 – inventory.effects
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 22 – inventory.routing  
**Next :** 24 – inventory.actions

---

## 🎯 Objectif

Construire l’**inventaire des effets** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- les inventaires Logic, Events, Dataflows, Async, Services, Routing, Actions si disponibles,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `effect.*`.

L’objectif est de produire un JSON `inventory.effects.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **effets UI** (focus, scroll, animations, overlays, toasts…),
- les **effets de navigation** (redirections, changement de route déclenchés par la logique),
- les **effets de lifecycle** (montage/démontage, subscriptions, side-effects de hooks),
- les **effets liés aux données** (post-traitement, synchronisation, mise en cache),
- les **effets techniques** (tracking, logs, métriques, instrumentation),
- leurs déclencheurs (événements, conditions, actions, changements d’état),
- leurs cibles (UI, data, routing, services…).

Cet inventaire ne :

- ne duplique pas l’analyse détaillée de la logique (`inventory.logic`),
- ne remplace pas l’inventaire Async (`inventory.async`) ni Routing (`inventory.routing`),
- ne décrit pas tous les dataflows (`inventory.dataflows`).  

Il se concentre sur **“ce qui se passe en réaction à quelque chose”** : événements, changements d’état, lifecycle, navigation.

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
    - les composants enfants,
    - les hooks custom,
    - les modules de services / data,
    - les utilitaires d’effets (tracking, logs, animations, modales…).
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Effects**
  - `${paths.core}/guides-internals/inventory/guide.inventory.effects.md`
  - Fournit :
    - l’**objectif** du domaine Effects,
    - le **schéma JSON contractuel** de `inventory.effects.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure / Logic / Events / Async / Routing / Actions),
    - les relations avec les autres inventaires.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les effets respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `effect.*`, par exemple :
  - `effect.logicTriggered`
  - `effect.ui.focus`
  - `effect.ui.scroll`
  - `effect.async`
  - `effect.lifecycle.mount`
  - `effect.lifecycle.unmount`
  - `effect.navigation`
- aider à identifier :
  - les hooks d’effets (`useEffect`, équivalents Vue/Angular…),
  - les callbacks qui déclenchent des effets,
  - les effets de navigation/tracking/animation.

> Si le bridge ne définit pas explicitement certains concepts `effect.*`, l’IA s’appuie sur les APIs du framework (React hooks, observers, écouteurs d’événements…) et documente les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible structure les effets (hooks dédiés, services de tracking, gestion centralisée des toasts),
    - anticiper la projection des effets Legacy vers une architecture d’effets plus propre.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions côté stack finale pour les effets (tracking centralisé, toasts unifiés, navigation déclarative…),
    - ajuster la granularité des `EffectItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants,
    - rattacher les effets à des vues/conteneurs (UI impactée).

- **Inventaires fortement recommandés**
  - `inventory.logic.json` (Stage 15)
    - pour relier les effets à la logique qui les déclenche,
  - `inventory.events.json` (Stage 18)
    - pour relier les effets aux événements déclencheurs (UI, système),
  - `inventory.dataflows.json` (Stage 19)
    - pour relier les effets aux dataflows (ex. effet “rafraîchir les données”),
  - `inventory.async.json` (Stage 20)
    - pour relier les effets aux patterns async (chargement, retry, polling),
  - `inventory.services.json` (Stage 21)
    - pour relier les effets aux services (ex. tracking, logging),
  - `inventory.routing.json` (Stage 22)
    - pour relier les effets aux changements de route.

- **Autres inventaires optionnels (si déjà générés)**  
  - `inventory.layout.json`,
  - `inventory.styles.json`,
  - `inventory.i18n.json`,
  - `inventory.config.json`,
  - `inventory.actions.json`,
  - `inventory.tests.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.logic.json` / `inventory.events.json` / `inventory.async.json` doit être signalée dans `validation.issues` si elle empêche de bien qualifier les effets.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.effects.md`,
- `domain` doit valoir `"effects"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (logic, events, dataflows, async, services, routing, actions, config, tests…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.actions.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.tests.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.effects.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - repérer les vues/composants où se concentrent les effets (toasts, banners, modales, overlays, panels de navigation…).
   - À partir de `inventory.logic.json` et `inventory.events.json` (si présents) :
     - construire des liens `logicUcr` ↔ `eventUcr` ↔ `structureUcr`.
   - À partir de `inventory.dataflows.json` / `inventory.async.json` / `inventory.services.json` :
     - repérer les effets déclenchant ou dépendant de dataflows, de patterns async ou de services.
   - À partir de `inventory.routing.json` :
     - repérer les effets de navigation (redirections, changements de route).
   - À partir du bridge :
     - exploiter les patterns `effect.*` pour reconnaître les effets typiques (UI, lifecycle, navigation, data, tracking).

4. **Analyser le code Legacy pour identifier les effets**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les hooks d’effets (`useEffect`, `useLayoutEffect`, équivalents Vue/Angular…),
       - les callbacks déclenchant des effets (ex. `onClick`, `onChange` qui déclenchent navigation, tracking, mutations globales),
       - les appels à des services de tracking/logging/toasts,
       - les accès à des APIs de navigation (router, history) utilisés comme side-effects,
       - les subscriptions (WebSockets, observables…) et leur nettoyage.
     - caractériser, pour chaque effet significatif :
       - le déclencheur (event, lifecycle, changement de state),
       - la cible (UI, data, routing, services, config),
       - le timing (montage, mise à jour, démontage, différé).

5. **Construire les items d’effets**
   - Créer un `EffectItem` par effet significatif (voir guide pour le schéma) :
     - définir le `kind` :
       - `"uiEffect"`, `"navigationEffect"`, `"lifecycleEffect"`, `"dataEffect"`, `"trackingEffect"`, `"globalStateEffect"`, etc.,
     - définir `effectSummary` :
       - `effectType`, `trigger`, `target`, `timing`, `dependencies`, `description`,
     - associer des `targetStructureUcrs` (vues impactées par l’effet),
     - relier les effets aux events, à la logique, aux dataflows, à l’async, aux services, au routing, aux actions via les champs prévus.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (logic, events, dataflows, async, services, routing, actions, config, tests…),
     - les effets critiques sont bien identifiés (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.effects.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "23",
  "stageName": "inventory.effects",
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

- `Gate ✅` si `inventory.effects.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `24-inventory.actions.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
