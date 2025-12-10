# 🧩 Stage 50 – generate-types
**Phase :** Phase 3 – Generation  
**Prev :** 46 – mappings-summary  
**Next :** 51 – generate-services

---

## 🎯 Objectif

Générer tous les **types** nécessaires à `${project.pageName}` à partir des mappings de Phase 2 (notamment `mapping.types.json`, `mapping.structure.json`, `mapping.data.json`) en appliquant les **stack-guides** appropriés pour la génération agnostique de types.

Le stage produit une **représentation finale des types** (fichiers `.ext` ou équivalent selon la stack) et une **métadonnée de traçabilité**.

Aucun aspect runtime n’est géré ici :  
➡️ Ce stage génère uniquement des **types statiques** (structures, DTO, interfaces, entités conceptuelles…).

---

## 🔌 Entrées

### 1. Configuration globale

Chargée depuis `core/configs/project.config.yaml` :

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

- `mapping.types.json`
- `mapping.structure.json`
- `mapping.data.json`

Le stage échoue en **Gate ❌** si `mapping.types.json` est absent ou invalide.

### 4. Stack-guides (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.types.md`
- éventuellement : `guide.naming.md`, `guide.conventions.md`

Ces guides définissent :

- la forme des types,
- les conventions de nommage,
- la structure des fichiers,
- les patterns d’export.

### 5. Contrats globaux

Depuis `${paths.core}/guides-internals/globals/` :

- `guide.ucr.md`  
- éventuellement : `guide.schema-validation.md`

Ces contrats permettent d'assurer :

- le respect des UCR dans la génération,
- la cohérence structurelle.

---

## 📤 Sorties

### 1. Types générés

Écrits sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/types/`

Exemples :

- `User.ext`
- `Campaign.ext`
- `Budget.ext`

> ⚠️ L’extension, la syntaxe interne et la forme exacte sont déterminées par les stack-guides.  
Aucun langage n’est imposé ici.

### 2. Métadonnées de stage

Écrit sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.types.meta.json`

Structure minimale :

```jsonc
{
  "domain": "types",
  "stageId": "50",
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

### Étape 1 — Charger la configuration et le contexte

1. Charger `project.config.yaml`  
2. Résoudre `${paths.core}` et `${paths.workspace}`  
3. Charger :  
   - `project-structure.json`  
   - `bridge-legacy-to-dsl.json`  

### Étape 2 — Charger et valider les mappings

Charger :

- `mapping.types.json`  
- `mapping.structure.json`  
- `mapping.data.json`  

Valider :

- structure JSON correcte  
- cohérence entre les mappings (ex. un type référencé doit exister)  

Si échec → Gate ❌.

### Étape 3 — Charger les stack-guides (indispensables)

Charger :

- `guide.types.md`  
- `guide.naming.md` (si présent)  
- `guide.conventions.md` (si présent)

Ces guides définissent :

- les patterns,
- la manière d'écrire les types,
- la syntaxe d’export.

Le stage établit une **AST Types** interne pour pouvoir appliquer ces patterns.

### Étape 4 — Construire l’AST de types

Pour chaque entrée de `mapping.types.json` :

Créer un nœud AST contenant :

- `typeName`  
- `properties[]` (nom, type, nullabilité, documentation)  
- `sourceMapping`  
- `ucrTrace[]` (hérité de `bridge-legacy-to-dsl`)  

Puis enrichir l’AST via `mapping.structure.json` et `mapping.data.json`.

### Étape 5 — Appliquer les stack-guides

À partir de l’AST :

- appliquer les règles de nommage (`guide.naming.md`)  
- appliquer les conventions (`guide.conventions.md`)  
- appliquer le modèle de fichier de `guide.types.md`

Chaque type AST → fichier physique, par exemple :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/types/${typeName}.ext`

### Étape 6 — Écrire les fichiers générés

Pour chaque type transformé :

- écrire le fichier `.ext` dans `src_new/types/`
- collecter dans `filesGenerated[]`

### Étape 7 — Générer la métadonnée de stage

Créer `generation.types.meta.json` contenant :

- liste des fichiers générés
- nombre total de types
- `validation.status = "valid"`
- warnings éventuels

### Étape 8 — Validation du schéma JSON (optionnelle)

- Si `validation.enableSchemaValidation = true` dans la configuration :
  - Charger le schéma depuis `${validation.schemasPath}/generation.types.schema.json`
  - Valider `generation.types.meta.json` contre ce schéma
  - En cas d'erreur de validation :
    - Mode `strict` : ajouter les erreurs dans `validation.issues[]`, fixer `status = "rejected"`, préparer `Gate ❌`
    - Mode `warning` : ajouter des warnings dans `validation.issues[]`, continuer normalement
  - Voir `${paths.core}/guides-internals/globals/guide.json-schema-validation.md` pour les détails d'implémentation

---

## 🧩 Gate

### Gate ✅  
Appliquer si :

- `mapping.types.json` chargé et valide  
- au moins un type généré  
- aucun échec critique  
- la métadonnée a été écrite  

### Gate ❌  
Dans les cas suivants :

- mapping manquant / invalide  
- AST impossible à construire  
- aucun fichier généré  
- échec de validation schéma  

---

## 📦 Next

➡️ Continuer avec : **51 – generate-services**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
