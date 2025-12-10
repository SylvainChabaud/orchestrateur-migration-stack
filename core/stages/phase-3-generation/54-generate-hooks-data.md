# 🧩 Stage 54 – generate-hooks-data
**Phase :** Phase 3 – Generation  
**Prev :** 53 – generate-hooks-logic  
**Next :** 55 – generate-components-atoms

---

## 🎯 Objectif

Générer tous les **hooks de données** (*data hooks*) nécessaires à `${project.pageName}`, à partir :

- des mappings Phase 2 (`mapping.hooks-data.json`, `mapping.data.json`, `mapping.services.json`, `mapping.stores.json`)
- des types générés (Stage 50)
- des services générés (Stage 51)
- des stores générés (Stage 52)
- des **stack-guides** définissant comment les data hooks doivent être structurés

Un hook de données est centré sur :

- l’accès et la **mise à disposition des données** (lecture, polling, refresh, préchargements)
- la **synchronisation** entre stores, services et transformations
- la **mise en cache**, la **normalisation**, ou la **dérivation** de données
- la constitution d’une **API simple** pour les components/pages

Ce stage génère :

- un fichier de hook par entrée de `mapping.hooks-data.json`
- une métadonnée retraçant les fichiers générés (`generation.hooks-data.meta.json`)

Aucune stack n’est imposée :  
➡️ Les stack-guides dictent la structure finale, signatures et patterns internes.

---

## 🔌 Entrées du stage

### 1. Configuration générale

Depuis `core/configs/project.config.yaml` :
- `project.name`
- `project.pageName`
- `${paths.core}`
- `${paths.workspace}`

### 2. Artefacts Phase 0

Depuis `${paths.workspace}/projects/${project.name}/stack/` :
- `project-structure.json`
- `bridge-legacy-to-dsl.json`

### 3. Mappings Phase 2 (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.hooks-data.json` (mapping principal)
- `mapping.data.json`
- `mapping.services.json`
- `mapping.stores.json`

Si `mapping.hooks-data.json` est absent → **Gate ❌**

### 4. Stack-guides (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.hooks-data.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent :

- comment un data hook expose les données
- comment structurer les états dérivés
- comment orchestrer stores/services côté data
- comment gérer loading/error states
- comment représenter caching, revalidation, refresh ou polling

### 5. Guides internes globaux

Depuis `${paths.core}/guides-internals/globals/` :

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 📤 Sorties

### 1. Hooks Data générés

Écrits dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/data/`

Exemples :

- `useCampaignData.ext`
- `useUserProfile.ext`
- `useBudgetData.ext`

Chaque hook doit fournir :

- les données prêtes à l’emploi
- le statut (loading, error, success)
- les transformations définies dans les mappings
- les UCR liées à l’obtention des données
- une API claire (`return { data, status, refresh }` ou équivalent)

### 2. Métadonnées du stage

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.hooks-data.meta.json`

Structure minimale :

```jsonc
{
  "domain": "hooks-data",
  "stageId": "54",
  "pageName": "${project.pageName}",
  "filesGenerated": [],
  "statistics": {
    "totalFiles": 0
  },
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 🧠 Actions

### Étape 1 — Charger configuration et contexte

Charger :

- `project.config.yaml`
- `project-structure.json`
- `bridge-legacy-to-dsl.json`

### Étape 2 — Charger et valider les mappings

Charger :

- `mapping.hooks-data.json`
- `mapping.data.json`
- `mapping.services.json`
- `mapping.stores.json`

Valider :

- que chaque hook définit une source de données claire
- que les stores/services référencés existent
- que les transformations sont définies

Si incohérence majeure → Gate ❌

### Étape 3 — Charger les stack-guides

Charger :

- `guide.hooks-data.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- comment structurer les hooks de données
- comment dériver l’état
- comment gérer erreurs et chargement
- comment exposer une API stable

### Étape 4 — Construire l’AST Hooks Data

Pour chaque entrée de `mapping.hooks-data.json`, construire un nœud AST contenant :

- `hookName`
- `dataSources[]` (services ou stores)
- `stateDependencies[]`
- `transformations[]` (issus de `mapping.data.json`)
- `pollingOptions` (si fournis)
- `refreshRules`
- `ucrTrace[]` via bridge
- `outputs[]` (data exposée)

### Étape 5 — Appliquer les stack-guides

Appliquer :

- conventions de fichier (`guide.conventions.md`)
- nommage (`guide.naming.md`)
- modèle de hook (`guide.hooks-data.md`)

Chaque AST produit un fichier :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/data/${hookName}.ext`

### Étape 6 — Écrire les fichiers générés

Pour chaque hook :

- écrire `.ext`
- ajouter son entrée dans `filesGenerated[]`

### Étape 7 — Générer la métadonnée du stage

Créer `generation.hooks-data.meta.json` :

- liste des fichiers générés
- totalFiles
- validation.status = "valid"

### Étape 8 — Validation schéma (optionnelle)

Si activée :

- valider la métadonnée selon schémas
- sinon Gate ❌

---

## 🧩 Gate

### Gate ✅

Si :

- `mapping.hooks-data.json` valide
- AST construite sans erreur
- ≥ 1 fichier généré
- métadonnée écrite
- aucune erreur critique

### Gate ❌

Cas :

- mapping principal absent
- dépendances introuvables
- transformation invalide
- aucun fichier généré
- échec de validation schéma

---

## 📦 Next

➡️ Continuer avec **Stage 55 – generate-components-atoms**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
