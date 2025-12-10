# 🧩 Stage 24 – inventory.actions
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 23 – inventory.effects  
**Next :** 25 – inventory.tests

---

## 🎯 Objectif

Construire l’**inventaire des actions** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- l’ensemble des inventaires comportementaux : Events, Logic, Dataflows, Async, Services, Routing, Effects,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `action.*` et les chaînes complètes event → logic → effect → dataflows → services → routing.

L’objectif est de produire un JSON `inventory.actions.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **actions métier ou UX significatives** (user actions + system actions),
- leurs **triggers** (événements UI, timers, conditions système, route changes…),
- leur **séquence end-to-end** :
  - événements → logique → dataflows/services → async → routing → effets,
- leurs **préconditions** (guards, état requis, droits, feature flags),
- leurs **issues** (succès, erreurs, annulation),
- leurs **impacts** (UI, data, navigation, tracking, state global).

Cet inventaire ne :

- ne remplace pas les inventaires plus granulaires (events, logic, dataflows, async, services, routing, effects),
- ne décrit pas chaque petite micro-interaction,
- ne vise pas à lister toutes les combinaisons possibles.  

Il se concentre sur les **actions significatives à l’échelle métier / UX**, celles que l’on doit absolument préserver (ou améliorer) dans la migration.

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
    - les composants enfants (buttons, forms, modales…),
    - les hooks custom qui orchestrent des flows complets,
    - les services / dataflows / routing / tracking,
    - les modules de logique métier.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Actions**
  - `${paths.core}/guides-internals/inventory/guide.inventory.actions.md`
  - Fournit :
    - l’**objectif** du domaine Actions,
    - le **schéma JSON contractuel** de `inventory.actions.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec tous les autres inventaires),
    - les relations avec les autres domaines.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les `ucr` introduits pour les actions respectent le contrat global,
    - garantir que `targetStructureUcrs` et autres références sont valides.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les concepts DSL :
  - `action.userAction`,
  - `action.systemAction`,
  - `action.compositeFlow`,
  - `action.backgroundFlow`,
- ainsi que les liens DSL entre :
  - `event.*` → `logic.*` → `effect.*` → `data.*` → `routing.*`.

> Si le bridge ne définit pas explicitement `action.*`, l’IA dérive les actions à partir des chaînes event→logic→effect→dataflows→routing déjà identifiées et documente les limites dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la stack cible souhaite structurer les “use cases” / actions (services d’application, handlers, custom hooks de page, etc.),
    - anticiper la projection des actions Legacy vers cette architecture.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître la manière recommandée de représenter les actions / use cases dans la stack finale,
    - adapter la granularité des `ActionItem`.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues / composants qui initient ou reflètent les actions.

- **Inventaires comportementaux — fortement recommandés**
  - `inventory.events.json` (Stage 18)
  - `inventory.logic.json` (Stage 15)
  - `inventory.dataflows.json` (Stage 19)
  - `inventory.async.json` (Stage 20)
  - `inventory.services.json` (Stage 21)
  - `inventory.routing.json` (Stage 22)
  - `inventory.effects.json` (Stage 23)

- **Autres inventaires optionnels (si déjà générés)**  
  - `inventory.layout.json`,
  - `inventory.styles.json`,
  - `inventory.i18n.json`,
  - `inventory.config.json`,
  - `inventory.tests.json`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
Sans un minimum d’inventaires comportementaux (events/logic/dataflows/services/routing/effects), l’IA doit limiter le périmètre et documenter les manques dans `validation.issues`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.actions.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.actions.md`,
- `domain` doit valoir `"actions"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
- les références vers d’autres inventaires (events, logic, dataflows, async, services, routing, effects, config, tests…) doivent être cohérentes,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.events.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.dataflows.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.async.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.services.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.routing.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.effects.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.tests.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.actions.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - repérer les vues/composants qui concentrent les actions principales (boutons, formulaires, menus contextuels, CTA).
   - À partir de `inventory.events.json` / `inventory.logic.json` :
     - construire des chaînes `eventUcr → logicUcr`,
     - repérer les couples “événement déclencheur” → “logic handler principal”.
   - À partir de `inventory.dataflows.json` / `inventory.async.json` / `inventory.services.json` :
     - repérer les dataflows, patterns async et services typiquement associés à ces handlers.
   - À partir de `inventory.routing.json` / `inventory.effects.json` :
     - repérer les effets de navigation et autres side-effects importants rattachés à ces handlers.
   - Exploiter le bridge DSL pour valider et enrichir ces chaînes.

4. **Identifier les actions significatives**
   - À partir des chaînes event→logic→effect→dataflows→services→routing :
     - regrouper les séquences en **Actions** cohérentes du point de vue métier/UX, par exemple :
       - “Créer une campagne”,
       - “Modifier puis sauvegarder une campagne”,
       - “Dupliquer une campagne”,
       - “Activer/Désactiver une campagne”,
       - “Appliquer un filtre complexe puis rafraîchir la liste”.
   - Classer les actions par :
     - `kind` (userAction, systemAction, compositeAction, backgroundAction…),
     - criticité métier.

5. **Construire les items d’actions**
   - Créer un `ActionItem` par action significative (voir guide pour le schéma) :
     - définir le `kind` (userAction, systemAction, compositeAction, backgroundAction…),
     - définir `actionSummary` :
       - `actionName`, `trigger`, `preconditions`, `mainFlowSteps`, `successCriteria`, `failureHandling`, `sideEffects`, `description`,
     - associer des `targetStructureUcrs` (vues/composants où l’action est initiée/observable),
     - relier l’action aux events, logic, dataflows, async, services, routing, effects, config, tests via les champs prévus.

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les liens vers les autres inventaires sont cohérents (events, logic, dataflows, async, services, routing, effects, config, tests…),
     - les actions critiques sont bien identifiées (via `metadata.severity`, etc.).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.actions.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.actions.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "24",
  "stageName": "inventory.actions",
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

- `Gate ✅` si `inventory.actions.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `25-inventory.tests.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
