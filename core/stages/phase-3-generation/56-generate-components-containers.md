# 🧩 Stage 56 – generate-components-containers
**Phase :** Phase 3 – Generation  
**Prev :** 55 – generate-components-atoms  
**Next :** 57 – generate-pages

---

## 🎯 Objectif

Générer tous les **components containers** nécessaires à `${project.pageName}`.

Un **container** est un composant UI structurant qui :

- orchestre **hooks (logic + data)**,  
- connecte **atoms** et **UI composites**,  
- prépare l’affichage final,  
- met en forme les données, événements et interactions,  
- applique les conventions d’architecture de la stack finale (via stack-guides).

Un container **ne contient pas** de logique métier brute :  
➡️ Il consomme *uniquement* les hooks générés aux stages 53 et 54.

Ce stage génère :

- les fichiers des containers dans `src_new/components/containers/`
- la métadonnée : `generation.containers.meta.json`

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
- `bridge-legacy-to-dsl.json`

### 3. Mappings Phase 2 (obligatoires)
Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.containers.json` → mapping principal
- `mapping.ui.json`
- `mapping.hooks-logic.json`
- `mapping.hooks-data.json`

Sans `mapping.containers.json` → **Gate ❌**

### 4. Stack-guides (obligatoires)
Depuis `${paths.workspace}/projects/${project.name}/stack/stack-guides/` :

- `guide.ui.containers.md`
- `guide.ui.layout.md`
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent :

- la structure UI d’un container,  
- l’intégration correcte des hooks,  
- l’organisation layout / sections,  
- les conventions de composition UI,  
- les règles d’accessibilité.

### 5. Guides internes globaux
Depuis `${paths.core}/guides-internals/globals/` :
- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 📤 Sorties

### 1. Containers générés
Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/containers/`

Exemples :

- `CampaignFormContainer.ext`
- `BudgetPanelContainer.ext`
- `UserProfileContainer.ext`

Chaque container doit contenir :

- connexion aux hooks concernés  
- injection de données dans les atoms, components, sections  
- gestion de la structure UI  
- absence totale de logique métier  
- gestion des erreurs UI selon stack-guides  
- documentation UCR (via bridge)

### 2. Métadonnées du stage

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.containers.meta.json`

Structure minimale :

```jsonc
{
  "domain": "containers",
  "stageId": "56",
  "pageName": "${project.pageName}",
  "filesGenerated": [],
  "statistics": { "totalFiles": 0 },
  "validation": { "status": "valid", "issues": [] }
}
```

---

## 🧠 Actions

### Étape 1 — Charger la configuration & contexte

Charger :

- `project.config.yaml`
- `project-structure.json`
- `bridge-legacy-to-dsl.json`

### Étape 2 — Charger & valider les mappings

Charger :

- `mapping.containers.json`
- `mapping.ui.json`
- `mapping.hooks-logic.json`
- `mapping.hooks-data.json`

Valider :

- existence des hooks référencés  
- cohérence UI structure / hooks / atoms  
- sections UI définies dans mappings

Si incohérence → **Gate ❌**

### Étape 3 — Charger les stack-guides UI

Charger :

- `guide.ui.containers.md`
- `guide.ui.layout.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils déterminent :

- composition UI  
- structure interne des containers  
- gestion des props / slots / sections  
- intégration agnostique des hooks  

### Étape 4 — Construire l’AST Container

Un nœud AST par container :

- `containerName`
- `hooksLogic[]`
- `hooksData[]`
- `atoms[]` ou autres composants requis
- `sections[]` (layout)
- `uiRole`
- `props[]`
- `ucrTrace[]` via bridge

### Étape 5 — Appliquer les stack-guides

Pour chaque AST :

- appliquer conventions (`guide.conventions.md`)
- nommage (`guide.naming.md`)
- structure UI (`guide.ui.containers.md`)
- disposition (`guide.ui.layout.md`)

Chaque AST → fichier :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/containers/${containerName}.ext`

### Étape 6 — Écrire les fichiers générés

Écrire chaque container, enregistrer :

`filesGenerated[]`

### Étape 7 — Générer la métadonnée

`generation.containers.meta.json` :

- liste des containers  
- stats  
- `status = "valid"`

### Étape 8 — Validation schéma (optionnelle)

Valider :

- structure des containers  
- cohérence avec schémas de composants  
- structure de la métadonnée  

En cas d’erreur → **Gate ❌**

---

## 🧩 Gate

### Gate ✅
- `mapping.containers.json` valide  
- AST complète  
- ≥ 1 container généré  
- métadonnée écrite  
- aucune erreur critique  

### Gate ❌
- mapping incomplet  
- hook manquant  
- structure UI invalide  
- aucun fichier généré  
- échec validation  

---

## 📦 Next

➡️ **Stage 57 – generate-pages**

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
