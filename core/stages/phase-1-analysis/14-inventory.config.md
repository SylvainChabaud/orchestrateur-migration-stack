# 🧩 Stage 14 – inventory.config
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 13 – inventory.i18n  
**Next :** 15 – inventory.logic

---

## 🎯 Objectif

Construire l’**inventaire de configuration** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- éventuellement les autres inventaires (layout, styles, i18n),
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `config.*` si disponibles.

L’objectif est de produire un JSON `inventory.config.json` qui décrit, de manière **canonique** et **framework-agnostique** :

- les **paramètres de configuration** utilisés par la page,
- les **sources de configuration** (fichiers, constantes, env vars, feature flags…),
- les **liens entre ces paramètres et les vues** (via les `ucr` de structure),
- les **impacts fonctionnels** majeurs (ex : activer/désactiver une section, changer un endpoint, activer une expérimentation).

Cet inventaire sert de base à :

- la Phase 2 (mappings.config) pour projeter la configuration vers la stack cible,
- la Phase 3 pour recâbler proprement les points de configuration (env, providers, hooks, etc.).

---

## ⚙️ Inputs

> Tous les chemins sont dérivés de `project.config.yaml` via `project.*` et `paths.*`  
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
    - fichiers de configuration (ex. `config/*.js`, `constants/*.ts`, etc.),
    - modules de feature flags,
    - accès aux variables d’environnement ou aux settings globaux.
  - ❌ Ne jamais copier ces fichiers dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire Config**
  - `${paths.core}/guides-internals/inventory/guide.inventory.config.md`
  - Fournit :
    - l’**objectif** du domaine Config,
    - le **schéma JSON contractuel** de `inventory.config.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure, usage des UCR, champs autorisés),
    - les relations avec les autres inventaires (logic, data, routing…).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les références vers les vues (`targetStructureUcrs`, etc.) utilisent des UCR valides,
    - s’assurer que les éventuels UCR propres au domaine Config respectent les même règles globales.

---

### 4. Bridge Legacy → DSL (optionnel mais recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts DSL `config.*`, par exemple :
  - `config.featureFlag`,
  - `config.environmentVariable`,
  - `config.runtimeSetting`,
  - `config.experimentToggle`,
  - `config.apiEndpointSetting`,
  - `config.pageLevelSetting`.
- garantir une interprétation cohérente des usages de configuration,
- éviter de dupliquer des heuristiques spécifiques à la stack Legacy.

> Si le bridge ne définit pas de concepts `config.*`, l’analyse se fait de manière générique (recherche de constantes, env, feature flags…) et cette limitation peut être mentionnée dans `validation.issues`.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre comment les paramètres de configuration devront être projetés (par page, par feature, par module),
    - anticiper les regroupements éventuels dans la stack cible.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions de configuration de la stack cible (env vars, fichiers `config.ts`, providers, etc.),
    - influencer la granularité recommandée des items de configuration.

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues,
    - ancrer les usages de paramètres de configuration à des endroits précis de l’interface.

- **Autres inventaires (optionnels)**
  - `inventory.layout.json` (Stage 11)
  - `inventory.styles.json` (Stage 12)
  - `inventory.i18n.json` (Stage 13)

Ces inventaires ne sont pas bloquants, mais peuvent aider à contextualiser certains paramètres (ex. config d’une région, d’un thème, de textes conditionnels).  
Leur absence peut être mentionnée dans `validation.issues` si cela réduit la finesse de l’analyse.

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.config.md`,
- `domain` doit valoir `"config"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références aux vues (`targetStructureUcrs`, etc.) doivent pointer vers des `ucr` valides de `inventory.structure.json`,
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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.i18n.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.config.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - identifier les vues clés (racine, containers, sections importantes).
   - À partir du bridge :
     - extraire les patterns `config.*` s’ils existent et les indexer par `dslId`.

4. **Analyser le code Legacy pour la configuration**
   - Partir de `${paths.legacySource}` et :
     - repérer :
       - l’utilisation de **variables d’environnement** (ex. `process.env.*`),
       - les **fichiers de configuration** importés (ex. `config`, `settings`, `featureFlags`),
       - les **feature flags** et toggles d’expérimentation,
       - les **constantes globales** qui pilotent le comportement (endpoints, limites, options).
     - relier chaque usage à :
       - une ou plusieurs vues (via `targetStructureUcrs`),
       - une catégorie de config (env, featureFlag, runtimeSetting, experimentToggle…).

5. **Construire les items de configuration**
   - Créer un `ConfigItem` par paramètre logique de configuration (voir le guide pour le schéma) :
     - identifier le **nom logique** (`configName`),
     - la **source** (env var, fichier, module, valeur inline),
     - le **type** (featureFlag, env, runtimeSetting…),
     - les **valeurs importantes** (default, overrides, etc.),
     - les **vues impactées** (`targetStructureUcrs`).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - tous les champs obligatoires sont présents,
     - toutes les références `targetStructureUcrs` sont valides,
     - les duplications ou incohérences de paramètres sont listées dans `validation.issues`.
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.config.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.config.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "14",
  "stageName": "inventory.config",
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

- `Gate ✅` si `inventory.config.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `15-inventory.logic.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
