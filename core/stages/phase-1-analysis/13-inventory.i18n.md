# 🧩 Stage 13 – inventory.i18n
**Phase :** Phase 1 – Analyse (Inventaires)  
**Prev :** 12 – inventory.styles  
**Next :** 14 – inventory.config

---

## 🎯 Objectif

Construire l’**inventaire i18n** pour la page `${project.pageName}` en s’appuyant sur :

- le code Legacy situé à `${paths.legacySource}`,
- l’inventaire de structure (`inventory.structure.json`),
- éventuellement l’inventaire de layout (`inventory.layout.json`),
- les guides internes,
- et le bridge Legacy → DSL pour les concepts `i18n.*` si disponibles.

L’objectif est de produire un JSON `inventory.i18n.json` qui décrit, de manière **canonique**, **framework-agnostique** et **centrée sur le domaine i18n** :

- les **chaînes de texte** affichées dans la page,
- les **keys** de traduction (si déjà utilisées dans le Legacy),
- les **namespaces / fichiers** de traduction impliqués,
- les **liens entre textes et vues** (via les `ucr` de structure),
- les éventuelles **variantes linguistiques ou contextuelles** (pluriels, genres, formats, etc.).

Cet inventaire sert de base à :

- la Phase 2 (mappings.i18n) pour projeter la stratégie i18n dans la stack cible,
- la Phase 3 (génération) pour câbler correctement les composants React 19 avec le système i18n d’entreprise.

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
  - Le stage peut suivre les imports pour découvrir :
    - les fichiers de configuration i18n (namespaces, translations),
    - les utilitaires ou hooks i18n (`t`, `useTranslation`, etc.),
    - les composants i18n (ex. `<Trans />`, `<FormattedMessage />`, etc.).
  - ❌ Ne jamais copier les fichiers Legacy dans `${paths.workspace}`.

---

### 3. Guides core (lecture seule)

- **Guide d’inventaire i18n**
  - `${paths.core}/guides-internals/inventory/guide.inventory.i18n.md`
  - Fournit :
    - l’**objectif** du domaine i18n,
    - le **schéma JSON contractuel** de `inventory.i18n.json`,
    - les champs obligatoires / optionnels,
    - les contraintes (cohérence avec Structure, références UCR valides),
    - les relations avec les autres inventaires (logic, config, etc.).

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation dans ce stage :
    - garantir que les références aux vues (`targetStructureUcr`, etc.) utilisent des UCR valides,
    - s’assurer qu’éventuels UCR introduits au niveau i18n (par ex. groupes i18n) respectent le contrat global.

---

### 4. Bridge Legacy → DSL (optionnel mais recommandé)

- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Rôle dans ce stage :

- exploiter les patterns Legacy associés aux concepts i18n du DSL, si présents, par exemple :
  - `i18n.keyUsage`
  - `i18n.namespace`
  - `i18n.pluralization`
  - `i18n.formatting`
- assurer une détection cohérente des usages i18n dans le Legacy,
- éviter de multiplier les heuristiques spécifiques à un framework i18n particulier.

> Si le bridge ne contient pas encore ces concepts, le stage doit tout de même fonctionner en analysant le code de manière générique, et le signaler dans `validation.issues` si nécessaire.

---

### 5. Structure cible & guides de stack (Phase 0)

- **Spécification de structure cible (Stage 01)**
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
  - Utilisation :
    - comprendre la structure cible pour répartir les ressources i18n (par page, par feature, par domaine),
    - anticiper la projection des clés i18n.

- **Guides de stack (Stage 00)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack-*.md` (optionnel mais utile)
  - Utilisation :
    - connaître les conventions i18n de la stack cible (fichiers, namespaces, patterns de clés, etc.),
    - orienter le niveau de granularité attendu pour les keys (page-level, domain-level, etc.).

---

### 6. Outputs précédents requis

- **Inventaire Structure (Stage 10) — obligatoire**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.structure.json`
  - Rôle :
    - fournir les `ucr` des vues,
    - permettre d’ancrer chaque usage i18n sur une vue (ou un groupe de vues).

- **Inventaire Layout (Stage 11) — optionnel**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.layout.json`
  - Rôle :
    - contextualiser certaines zones textuelles par région (header, footer, overlays, etc.).

- **Inventaire Styles (Stage 12) — optionnel**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.styles.json`
  - Rôle :
    - éventuellement repérer des textes précis liés à des styles particuliers (ex. badges, tags, labels stylés).

Sans `inventory.structure.json`, le stage doit conclure sur un `Gate ❌`.  
L’absence de `inventory.layout.json` ou `inventory.styles.json` ne bloque pas, mais doit être notée dans `validation.issues` si cela limite l’analyse.

---

## 📤 Outputs

Tous les outputs sont écrits dans `${paths.workspace}`.

### 1. Inventaire principal

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.i18n.json`

Contraintes :

- respecte le schéma défini dans `guide.inventory.i18n.md`,
- `domain` doit valoir `"i18n"`,
- `pageName` doit correspondre à `${project.pageName}`,
- `sourceEntry` doit pointer vers `${paths.legacySource}`,
- les références aux vues (`targetStructureUcr`, etc.) doivent pointer uniquement vers des `ucr` valides,
- JSON strictement sérialisable, sans clés inconnues.

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
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.styles.json` (si présent).
   - Lire les guides core :
     - `${paths.core}/guides-internals/inventory/guide.inventory.i18n.md`,
     - `${paths.core}/guides-internals/globals/guide.ucr.md`.

3. **Préparer les index en mémoire**
   - À partir de `inventory.structure.json` :
     - construire un index `structureUcr → StructureNode`,
     - éventuellement un index par type de vue (root, container, présentational…) pour localiser les textes.
   - À partir de `inventory.layout.json` (si présent) :
     - construire un index `layoutUcr → LayoutItem` pour connecter certains textes à des régions.
   - À partir du bridge :
     - extraire les patterns `i18n.*` s’ils existent et les indexer par `dslId`.

4. **Analyser le code Legacy pour les usages i18n**
   - Partir de `${paths.legacySource}` et :
     - repérer les appels aux APIs i18n (`t('key')`, `intl.formatMessage`, etc.),
     - repérer les composants/translations (`<Trans>`, `<FormattedMessage>`, etc.),
     - relever les textes littéraux qui semblent destinés à être externalisés en i18n.
   - Pour chaque occurrence, déterminer :
     - la **key** utilisée (si existante),
     - le **namespace / fichier** auquel elle appartient (si identifiable),
     - la **vue de structure** concernée (via un `targetStructureUcr`),
     - le type d’usage (label, titre, message d’erreur, help text, tooltip, etc.).

5. **Construire les items i18n**
   - Regrouper les usages en `I18nItem` logiques (par key, par vue, ou par namespace selon le schéma défini dans le guide) :
     - renseigner les champs obligatoires (key, namespace, targetStructureUcrs, etc.),
     - documenter les variantes (singulier/pluriel, formats, contextes).

6. **Assembler le JSON final**
   - Construire la racine :
     - `domain`, `pageName`, `sourceEntry`, `items[]`, `validation`.
   - Vérifier la conformité au schéma contractuel.

7. **Validation interne**
   - Vérifier que :
     - toutes les références `targetStructureUcr` sont valides,
     - les champs obligatoires sont présents pour chaque item,
     - les patterns incohérents (keys non résolues, textes en dur critiques) sont consignés dans `validation.issues`.
   - Mettre à jour :
     - `validation.status` (`"valid"` ou `"rejected"`),
     - `validation.issues[]`.

8. **Écriture de l’output**
   - Écrire `inventory.i18n.json` dans :
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.i18n.json`
   - Ne pas modifier les autres inventaires.

---

## ✅ Auto-Checks

Exemple de résumé à produire en fin d’étape (dans la réponse IA, pas sur disque) :

```json
{
  "stageId": "13",
  "stageName": "inventory.i18n",
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

Si `inventory.layout.json` ou `inventory.styles.json` sont absents, le stage peut rester `valid` mais doit l’indiquer dans `validation.issues`.

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

- `Gate ✅` si `inventory.i18n.json` a été généré et validé.
- `Gate ❌` si une erreur bloquante empêche la production de l’inventaire (ex : `inventory.structure.json` absent ou invalide, schéma violé).

---

## 📦 Next

> Continuer avec `14-inventory.config.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
