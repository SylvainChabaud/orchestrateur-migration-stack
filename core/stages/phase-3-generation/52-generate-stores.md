# 🧩 Stage 52 – generate-stores
**Phase :** Phase 3 – Generation  
**Prev :** 51 – generate-services  
**Next :** 53 – generate-hooks-logic

---

## 🎯 Objectif

Générer tous les **stores** nécessaires à `${project.pageName}` à partir :

- des mappings de Phase 2 (`mapping.stores.json`, `mapping.state.json`, `mapping.data.json`, `mapping.logic.json`)
- des types générés (Stage 50)
- des services générés (Stage 51)
- des **stack-guides** pour la génération agnostique

Les stores représentent :

- la **source de vérité locale**,
- l'état partagé entre composants/pages/services,
- les mécanismes d'abstraction de la logique métier,
- les mutations autorisées.

Ce stage produit :

- les fichiers de stores dans `src_new/stores/`
- un fichier de métadonnées retraçant les stores générés.

---

## 🔌 Entrées du stage

### 1. Configuration générale

Chargée depuis `project.config.yaml` :

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

- `mapping.stores.json` (mapping principal)
- `mapping.state.json`
- `mapping.data.json`
- `mapping.logic.json`

Si `mapping.stores.json` est absent ou vide → **Gate ❌**.

### 4. Stack-guides (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.stores.md`
- `guide.state.md` (si présent)
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent :

- la forme d’un store,
- la structure de l’état initial,
- les mutations et actions,
- la manière de représenter les effets,
- la façon de connecter services/stores.

### 5. Guides internes globaux

Depuis `${paths.core}/guides-internals/globals/` :

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 📤 Sorties

### 1. Stores générés

Écrits dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/stores/`

Exemples :

- `CampaignStore.ext`
- `UserProfileStore.ext`
- `BudgetStore.ext`

Chaque fichier contient :

- état initial
- mutations
- actions exposées
- éventuels effets asynchrones (selon stack)
- documentation métier + UCR

### 2. Métadonnées du stage

Dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.stores.meta.json`

Structure minimale :

```jsonc
{
  "domain": "stores",
  "stageId": "52",
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

### Étape 1 – Charger configuration et contexte

Charger :

- `project.config.yaml`
- `project-structure.json`
- `bridge-legacy-to-dsl.json`

### Étape 2 – Charger et valider les mappings

Charger :

- `mapping.stores.json`
- `mapping.state.json`
- `mapping.data.json`
- `mapping.logic.json`

Vérifier :

- existence d'au moins un store
- cohérence types/state (type de propriété existant)
- cohérence logic/actions (mutations définies)

Si incohérence → Gate ❌.

### Étape 3 – Charger les stack-guides

Charger :

- `guide.stores.md`
- `guide.state.md` (si présent)
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- comment représenter l’état
- comment organiser les mutations
- comment exposer les actions
- comment gérer les effets asynchrones

### Étape 4 – Construire l’AST Stores

Créer un nœud AST pour chaque store :

- `storeName`
- `state` (issu de `mapping.state.json`)
- `mutations[]`
- `actions[]`
- `effects[]` (si définis dans `mapping.logic.json`)
- `dependencies[]` (services nécessaires)
- `ucrTrace[]` via bridge

### Étape 5 – Appliquer les stack-guides

Transformations effectuées :

- application des conventions (`guide.conventions.md`)
- nommage (`guide.naming.md`)
- structure du fichier (`guide.stores.md`)

Chaque store AST produit un fichier :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/stores/${storeName}.ext`

### Étape 6 – Écrire les fichiers générés

Pour chaque store AST :

- écrire le fichier `.ext`
- enregistrer dans `filesGenerated[]`

### Étape 7 – Générer la métadonnée de stage

Créer :

`generation.stores.meta.json`

Avec :

- fichiers générés
- statistiques (nombre de stores, LOC si calculable)
- validation.status = "valid"

### Étape 8 – Validation schéma (optionnelle)

Si `enableSchemaValidation = true` :

- valider la métadonnée et éventuellement les fichiers
- si échec → Gate ❌

---

## 🧩 Gate

### Gate ✅  
Le stage réussit si :

- `mapping.stores.json` valide
- AST Stores construite
- ≥ 1 store généré
- métadonnée écrite
- aucune erreur critique

### Gate ❌  
Cas typiques :

- mapping stores manquant
- état incohérent ou mutation invalide
- aucun store généré
- échec de validation schéma

---

## 📦 Next

➡️ Passer à : **53 – generate-hooks-logic**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
