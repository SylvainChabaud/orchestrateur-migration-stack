# 🧩 Stage 53 – generate-hooks-logic
**Phase :** Phase 3 – Generation  
**Prev :** 52 – generate-stores  
**Next :** 54 – generate-hooks-data

---

## 🎯 Objectif

Générer tous les **hooks de logique métier** nécessaires à `${project.pageName}` à partir :

- des mappings de Phase 2 (`mapping.logic.json`, `mapping.hooks-logic.json`, `mapping.stores.json`, `mapping.services.json`)
- des artefacts produits aux stages 50 (types), 51 (services), 52 (stores)
- des **stack-guides** décrivant comment doit être implémenté un hook de logique dans la stack cible

Un hook de logique :

- encapsule un **comportement métier**,
- orchestre **services + stores**,
- expose une **API simple** aux composants/pages,
- peut gérer des **effets**, des **événements**, ou des **scénarios métier complexes**.

Ce stage génère :

- des fichiers de hooks dans `src_new/hooks/logic/`
- une métadonnée retraçant les hooks générés

Aucune stack n’est imposée :  
➡️ Les stack-guides décident de la structure, des signatures et du style final.

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
- `mapping.hooks-logic.json` (mapping principal)
- `mapping.logic.json`
- `mapping.services.json`
- `mapping.stores.json`

Si `mapping.hooks-logic.json` est absent → **Gate ❌**

### 4. Stack-guides (obligatoires)
Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :
- `guide.hooks-logic.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

### 5. Guides internes globaux
Depuis `${paths.core}/guides-internals/globals/` :
- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md` (si présent)

---

## 📤 Sorties

### 1. Hooks générés
Dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/logic/`

Exemples :

- `useCampaignValidation.ext`
- `useBudgetRecompute.ext`
- `useSubmitCampaignForm.ext`

Chaque hook doit :
- composer plusieurs stores/services
- gérer transitions d’état
- encapsuler un comportement métier
- exposer une API stable (`return {...}` ou équivalent agnostique)
- documenter UCR, règles métier, erreurs

### 2. Métadonnées du stage
`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.hooks-logic.meta.json`

Structure minimale :
```jsonc
{
  "domain": "hooks-logic",
  "stageId": "53",
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
- `mapping.hooks-logic.json`
- `mapping.logic.json`
- `mapping.stores.json`
- `mapping.services.json`

Valider :
- cohérence des actions/services/stores référencés
- existence des stores nécessaires
- existence des services requis

Si incohérence majeure → **Gate ❌**

### Étape 3 – Charger les stack-guides
Charger :
- `guide.hooks-logic.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent :
- la structure du hook logique
- comment orchestrer stores/services
- comment gérer effets, erreurs, dépendances
- règles de nommage

### Étape 4 – Construire l’AST Hooks Logic
Pour chaque hook dans `mapping.hooks-logic.json`, produire un nœud AST contenant :
- `hookName`
- `description`
- `inputs[]`
- `outputs[]`
- `stateDependencies[]` (stores)
- `serviceDependencies[]`
- `logicSteps[]` (issus de mapping.logic)
- `ucrTrace[]` via bridge

### Étape 5 – Appliquer les stack-guides
Transformation :
- conventions → `guide.conventions.md`
- nommage → `guide.naming.md`
- structure interne → `guide.hooks-logic.md`

Chaque AST → fichier final :
`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/logic/${hookName}.ext`

### Étape 6 – Écrire les fichiers générés
Pour chaque hook :
- écrire `.ext`
- ajouter son entrée dans `filesGenerated[]`

### Étape 7 – Générer la métadonnée du stage
Créer :
`generation.hooks-logic.meta.json`

Avec :
- fichiers générés
- statistiques
- validation.status = "valid"

### Étape 8 – Validation schéma (optionnelle)
Si activée :
- valider structure JSON de la meta
- valider structure des hooks selon schémas (si dispo)
- sinon Gate ❌

---

## 🧩 Gate

### Gate ✅
Si :
- `mapping.hooks-logic.json` valide
- AST Hooks Logic construite
- ≥ 1 hook généré
- métadonnée écrite
- aucune erreur critique

### Gate ❌
Causes :
- mapping principal absent
- hooks introuvables ou incohérents
- références à stores/services inexistants
- aucun hook généré
- échec de validation schéma

---

## 📦 Next

➡️ Continuer avec **Stage 54 – generate-hooks-data**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
