# 🧩 Stage 51 – generate-services
**Phase :** Phase 3 – Generation  
**Prev :** 50 – generate-types  
**Next :** 52 – generate-stores

---

## 🎯 Objectif

Générer tous les **services métiers** nécessaires à `${project.pageName}` à partir des mappings de Phase 2 :

- `mapping.services.json` (principal)
- `mapping.types.json`
- `mapping.data.json`
- `mapping.logic.json`

Les services représentent les **unités fonctionnelles** de la page :  
appels API, transformations métiers, orchestration interne, validations, interactions entre domaines.

Ce stage produit :

- une **représentation finale des services** dans `src_new/services/`
- une **métadonnée** retraçant les fichiers générés et les validations appliquées.

---

## 🔌 Entrées du stage

### 1. Configuration générale

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `${paths.core}`
- `${paths.workspace}`

### 2. Artefacts Phase 0 (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `bridge-legacy-to-dsl.json` (pour la traçabilité UCR)

### 3. Mappings Phase 2 (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.services.json`
- `mapping.types.json`
- `mapping.data.json`
- `mapping.logic.json`

Si `mapping.services.json` est absent → **Gate ❌** immédiat.

### 4. Stack-guides (obligatoires)

Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.services.md`
- `guide.types.md`
- `guide.conventions.md`
- `guide.naming.md`

Ces guides indiquent :

- comment structurer un service
- comment documenter un service
- comment représenter les inputs/outputs
- comment appliquer les règles de nommage

### 5. Guides internes globaux

Depuis `${paths.core}/guides-internals/globals/` :

- `guide.ucr.md`
- `guide.error-handling.md` (si présent)
- `guide.schema-validation.md` (si présent)

---

## 📤 Sorties

### 1. Services générés

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/services/`

Exemples de fichiers :

- `CreateCampaignService.ext`
- `LoadUserProfileService.ext`
- `ComputeBudgetService.ext`

L’extension `.ext` reste abstraite et définie par la stack.

Chaque service doit décrire :

- ses inputs structurés
- ses outputs
- son rôle métier
- les UCR associées
- les erreurs attendues
- les dépendances internes (stores, hooks, types)

### 2. Métadonnées du stage

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.services.meta.json`

Structure minimale :

```jsonc
{
  "domain": "services",
  "stageId": "51",
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

1. Charger `project.config.yaml`
2. Résoudre `${paths.core}` et `${paths.workspace}`
3. Charger `project-structure.json`
4. Charger `bridge-legacy-to-dsl.json`

### Étape 2 — Charger et valider les mappings de services

Charger :

- `mapping.services.json`
- `mapping.types.json`
- `mapping.data.json`
- `mapping.logic.json`

Vérifier :

- validité du schéma
- cohérence types ↔ services
- présence d’au moins un service

Si incohérence majeure → Gate ❌.

### Étape 3 — Charger les stack-guides

Charger :

- `guide.services.md`
- `guide.types.md`
- `guide.conventions.md`
- `guide.naming.md`

Les guides déterminent :

- la structure interne des services
- comment déclarer les paramètres
- comment représenter les erreurs
- comment écrire les signatures

### Étape 4 — Construire l’AST Services

Pour chaque entrée de `mapping.services.json` :

Créer un nœud AST contenant :

- `serviceName`
- `inputs[]` (types, nullabilité, documentation)
- `outputs`
- `logicSteps[]` (issus de `mapping.logic.json`)
- `dataRequirements[]` (issus de `mapping.data.json`)
- `ucrTrace[]` (via bridge)

L’AST doit être cohérent avec les types générés en Stage 50.

### Étape 5 — Appliquer les stack-guides

À partir de l’AST :

- appliquer les règles de nommage (`guide.naming.md`)
- appliquer les conventions (`guide.conventions.md`)
- utiliser `guide.services.md` comme template de génération

Chaque nœud AST produce un fichier service dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/services/${serviceName}.ext`

### Étape 6 — Écrire les fichiers générés

Pour chaque service :

- écrire le fichier `.ext`
- enregistrer dans `filesGenerated[]`

### Étape 7 — Générer la métadonnée de stage

Créer `generation.services.meta.json` avec :

- liste des services générés
- statistiques (nombre, éventuelles LOC)
- validation : `"valid"`

### Étape 8 — Validation schéma (optionnelle)

Si `enableSchemaValidation = true` :

- valider la métadonnée et les fichiers générés
- en cas d’erreur → Gate ❌

---

## 🧩 Gate

### Gate ✅  
Conditions :

- `mapping.services.json` valide
- AST services construite
- ≥ 1 service généré
- aucune erreur critique
- métadonnée écrite

### Gate ❌  
Causes possibles :

- absence du mapping services
- service sans inputs/outputs valides
- impossibilité d’appliquer un guide
- aucun fichier généré
- validation schéma échouée

---

## 📦 Next

➡️ Continuer avec : **52 – generate-stores**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
