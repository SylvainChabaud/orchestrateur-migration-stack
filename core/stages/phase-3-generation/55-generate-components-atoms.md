# 🧩 Stage 55 – generate-components-atoms
**Phase :** Phase 3 – Generation  
**Prev :** 54 – generate-hooks-data  
**Next :** 56 – generate-components-containers

---

## 🎯 Objectif

Générer tous les **components atoms** nécessaires à `${project.pageName}`, à partir :

- des mappings Phase 2 (`mapping.atoms.json`, `mapping.ui.json`, `mapping.types.json`)
- des types (Stage 50)
- des services/stores/hooks générés aux stages 51–54
- des **stack-guides UI** dictant la structure, les conventions et le style agnostique

Un **atom** est un composant UI fondamental, minimal et réutilisable.  
Il ne contient **aucune logique métier**, seulement :

- props typées,
- rendu UI simple,
- styles agnostiques dictés par les stack-guides,
- accessibilité si définie dans les guides.

---

## 🔌 Entrées du stage

### 1. Configuration générale
Depuis `project.config.yaml` :
- `project.name`
- `project.pageName`
- `${paths.workspace}`
- `${paths.core}`

### 2. Artefacts Phase 0
Depuis `${paths.workspace}/projects/${project.name}/stack/` :
- `project-structure.json`
- `bridge-legacy-to-dsl.json` (pour la traçabilité UCR → labels UI, props héritées, contraintes)

### 3. Mappings Phase 2 (obligatoires)
Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.atoms.json` → mapping principal
- `mapping.ui.json`
- `mapping.types.json`

Sans `mapping.atoms.json` → **Gate ❌**

### 4. Stack-guides (obligatoires)
Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.ui.atoms.md`
- `guide.ui.props.md`
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent :

- structure d’un atom,
- règles d’accessibilité,
- conventions de props,
- règles de nommage,
- structure UI agnostique (pas de stack imposée).

### 5. Guides internes globaux
Depuis `${paths.core}/guides-internals/globals/` :

- `guide.ucr.md`
- `guide.error-handling.md` (si props validation)
- `guide.schema-validation.md`

---

## 📤 Sorties

### 1. Atoms générés
Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/atoms/`

Exemples :

- `ButtonAtom.ext`
- `InputAtom.ext`
- `IconAtom.ext`

Chaque atom doit inclure :

- props typées
- rendu UI minimal
- absence totale de logique métier
- documentation UCR (si héritée)
- conventions de structure définies par `guide.ui.atoms.md`

### 2. Métadonnées
Dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.atoms.meta.json`

Structure minimale :

```jsonc
{
  "domain": "atoms",
  "stageId": "55",
  "pageName": "${project.pageName}",
  "filesGenerated": [],
  "statistics": {
    "totalFiles": 0
  },
  "validation": { "status": "valid", "issues": [] }
}
```

---

## 🧠 Actions

### Étape 1 — Charger configuration & contexte

Charger :

- `project.config.yaml`
- `project-structure.json`
- `bridge-legacy-to-dsl.json`

### Étape 2 — Charger et valider les mappings

Charger :

- `mapping.atoms.json`
- `mapping.ui.json`
- `mapping.types.json`

Vérifier :

- chaque atom a une `uiRole`
- props valides et typées
- mapping cohérent avec types

Si erreur → **Gate ❌**

### Étape 3 — Charger les stack-guides

Charger :

- `guide.ui.atoms.md`
- `guide.ui.props.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils déterminent :

- structure UI finale
- syntaxe agnostique du composant
- règles de props
- style/conventions

### Étape 4 — Construire l’AST Atom

Par atom :

- `atomName`
- `props[]`
- `uiRole`
- `accessibility`
- `defaultValues`
- `ucrTrace[]`

### Étape 5 — Appliquer les stack-guides

Pour chaque AST :

- appliquer les règles de nommage (`guide.naming.md`)
- structurer les props (`guide.ui.props.md`)
- construire le rendu (`guide.ui.atoms.md`)
- appliquer conventions (`guide.conventions.md`)

### Étape 6 — Écrire les fichiers générés

Dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/atoms/${atomName}.ext`

Ajouter à `filesGenerated[]`.

### Étape 7 — Générer la métadonnée

Créer `generation.atoms.meta.json`.

### Étape 8 — Validation schéma (optionnelle)

Si activée, valider :

- props structure  
- métadonnée JSON  
- structure UI si schémas fournis

---

## 🧩 Gate

### Gate ✅
- `mapping.atoms.json` valide
- ≥ 1 atom généré
- métadonnée écrite
- aucune incohérence critique

### Gate ❌
- mapping manquant ou invalide
- props non typées
- uiRole non défini
- aucun fichier généré

---

## 📦 Next

➡️ **Stage 56 – generate-components-containers**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
