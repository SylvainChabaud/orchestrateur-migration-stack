# 🧩 Stage 15 – inventory.logic
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 14 – inventory.config  
**Next :** 16 – inventory.conditions

---

## 🎯 Objectif

Construire l’**inventaire de logique** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- éventuellement les inventaires layout / styles / i18n / config,
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `logic.*`.

L’objectif est de produire un JSON `inventory.logic.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **états locaux** (local state) et **états dérivés** (derived state),
- les **règles métier** (business rules) agissant sur ces états,
- les **transformations de données purement logiques**,
- les **cycles de vie de la logique** (initialisation, reset logique, etc.), hors effets.

Cet inventaire **ne traite pas** :

- les conditions (if/else, guards → `inventory.conditions`),
- les hooks (React, custom hooks → `inventory.hooks`),
- les événements et handlers (`inventory.events`),
- les effets et side-effects (`inventory.effects`),
- les appels de données (`inventory.dataflows`, `inventory.services`).  

Il se concentre sur la **logique pure** et les **modèles d’état**.

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
  - Le stage peut suivre les imports pour atteindre :
    - les composants / modules contenant la logique métier,
    - les hooks custom,
    - les utilitaires de calcul ou de transformation de données,
    - les reducers / stores locaux (si utilisés).
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Logic**
  - `${paths.core}/guides-internals/inventory/guide.inventory.logic.md`
  - Fournit :
    - l’**objectif** du domaine Logic,
    - le **schéma JSON contractuel** de `inventory.logic.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure, séparation nette d’avec Conditions / Events / Effects),
    - les relations avec les autres inventaires (conditions, hooks, dataflows…).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les références aux vues (`targetStructureUcrs`, etc.) utilisent des UCR valides,
    - s’assurer que les UCR introduits pour la logique (`logicUcr`, etc.) respectent le contrat global.

---

### 4. Bridge Legacy → DSL (recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `logic.*`, par exemple :
  - `logic.localState`
  - `logic.derivedState`
  - `logic.businessRule`
  - `logic.formValidation`
  - `logic.computation`
- s’appuyer sur les patterns pour :
  - distinguer les parties de logique pure des effets / événements,
  - repérer les modules utilitaires de logique métier.

> Si les concepts `logic.*` sont partiellement définis dans le bridge, le stage doit rester robuste : utiliser ce qui existe, documenter les limitations dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment la logique sera projetée (hooks, services, stores…),
    - anticiper la répartition future entre composants/container/hooks.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître la philosophie de la stack cible (logique dans des hooks, dans des services, dans des stores…),
    - ajuster la granularité des unités de logique en conséquence.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues,
    - permettre de lier chaque unité logique à une ou plusieurs vues cibles ou containers logiques.

- **Autres inventaires (optionnels mais utiles)**
  - `inventory.layout.json` (Stage 11)
  - `inventory.styles.json` (Stage 12)
  - `inventory.i18n.json` (Stage 13)
  - `inventory.config.json` (Stage 14)

Ces inventaires permettent parfois de mieux contextualiser certaines règles (ex. logique dépendante de flags de config, de textes, de layout).  
Leur absence ne bloque pas, mais peut être mentionnée dans `validation.issues`.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.logic.md`,
- `domain` doit valoir `"logic"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références `targetStructureUcrs` doivent pointer vers des `ucr` valides de `inventory.structure.json`,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.styles.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.i18n.json` (si présent),
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.logic.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - distinguer les nœuds container de logique vs purement présentations si disponible.
   - À partir du bridge :
     - extraire les patterns `logic.*` et les indexer par `dslId`.

4. **Analyser le code Legacy pour la logique**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - les **états locaux** (ex. `useState`, `this.state`, stores locaux),
       - les **états dérivés** (selectors, mémos, calculs dérivés),
       - les **règles métier** (fonctions pures qui décident d’un résultat à partir de données/états),
       - les **fonctions utilitaires de calcul** (validation métier, scoring, formatage complexe).
     - ignorer dans cet inventaire :
       - les détails des conditions (if/else, ternaires → `inventory.conditions`),
       - les interactions avec des effets, événements ou dataflows (référencées mais non détaillées ici).

5. **Construire les items de logique**
   - Créer un `LogicItem` par unité logique significative (voir guide pour le schéma) :
     - définir la **nature** de la logique (localState, derivedState, businessRule, etc.),
     - lier cette logique aux vues concernées (`targetStructureUcrs`),
     - documenter les **inputs** et **outputs** logiques (données/états en entrée / résultat logique en sortie),
     - noter les relations éventuelles avec la config ou la data (sans les détailler).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les unités logiques ne mélangent pas effet/événement/données (sinon le noter dans `validation.issues`).
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.logic.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.logic.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "15",
  "stageName": "inventory.logic",
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

- `Gate ✅` si `inventory.logic.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `16-inventory.conditions.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
