# 📘 Guide Génération — Stores  
*(Domaine : **stores** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **stores** décrit comment générer les **sources de vérité locales** pour `${project.pageName}`.

Un store fournit :

- un **état initial structuré**
- des **mutations clairement définies**
- des **actions** consommables par les hooks et services
- des **effets** (optionnels, selon stack)
- un **point central d'orchestration métier**

Il est agnostique : aucune stack n’est imposée.  
Les stack-guides définissent *comment* le store final doit être écrit.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2 (obligatoires)

- `mapping.stores.json` → liste des stores
- `mapping.state.json` → structure détaillée de l’état
- `mapping.data.json` → interactions API
- `mapping.logic.json` → logique interne (mutations, actions, effets)

### 2.2. Structure projet (Phase 0)

- `project-structure.json`

Définit l’emplacement des stores.

### 2.3. Bridge Legacy → DSL

- `bridge-legacy-to-dsl.json`

Permet de :

- rattacher chaque mutation aux UCR
- justifier les transformations d’état
- documenter les règles métiers

### 2.4. Stack-guides

- `guide.stores.md`
- `guide.state.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- la forme syntaxique des stores
- les patterns pour l’état
- la structure des actions/mutations
- la manière d’écrire les effets

### 2.5. Guides internes globaux

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 3. 🧩 Structure d’entrée typique

### Exemple `mapping.stores.json`

```jsonc
{
  "stores": [
    {
      "name": "CampaignStore",
      "state": "CampaignState",
      "actions": ["loadCampaign", "updateBudget"],
      "mutations": ["setCampaign", "setBudget"],
      "effects": ["fetchCampaignFromApi"]
    }
  ]
}
```

### Exemple `mapping.state.json`

```jsonc
{
  "state": [
    {
      "name": "CampaignState",
      "properties": [
        { "name": "campaign", "type": "Campaign", "nullable": true },
        { "name": "loading", "type": "boolean", "nullable": false }
      ]
    }
  ]
}
```

---

## 4. 📤 Outputs attendus

### 4.1. Fichiers stores

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/stores/`

Exemples :

- `CampaignStore.ext`
- `BudgetStore.ext`

Un store doit inclure :

- un état initial
- les mutations
- les actions
- les effets
- la documentation & UCR

### 4.2. Métadonnées du domaine

`generation.stores.meta.json`

Contient :

- liste des stores générés
- statistiques globales
- état de validation

---

## 5. 🔍 Patterns Conceptuels

### 5.1. AST Store

Un nœud AST par store :

- `storeName`
- `state.properties[]`
- `mutations[]`
- `actions[]`
- `effects[]`
- `dependencies[]` (services)
- `ucrTrace[]`

### 5.2. Patterns typiques

Selon les stack-guides :

- **Store simple**  
  - état minimal + mutations

- **Store avec effets**  
  - interaction forte avec les services

- **Store orchestration**  
  - combine plusieurs sources de données

### 5.3. Mutations

Chaque mutation doit :

- être pure (si la stack le demande)
- modifier une propriété existante du state
- respecter le type

### 5.4. Actions

Les actions peuvent :

- appeler des services
- déclencher une ou plusieurs mutations
- produire des effets asynchrones

### 5.5. Documentation UCR

Le store doit exposer :

- quelles UCR sont couvertes
- quelles règles métiers justifient l’état
- quelles interactions sont possibles

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes

- store sans state associé
- mutation ciblant une propriété inexistante
- action référencée mais non définie
- state introuvable dans `mapping.state.json`

### Non bloquantes

- documentation incomplète
- propriétés inutilisées
- absence d’effets secondaires

---

## 7. ✔️ Checklist finale du domaine

Avant validation :

- [ ] `mapping.stores.json` chargé  
- [ ] chaque store possède un state valide  
- [ ] toutes les mutations ciblent des propriétés existantes  
- [ ] actions cohérentes avec les services  
- [ ] fichiers écrits dans `src_new/stores/`  
- [ ] métadonnée générée  
- [ ] aucune erreur critique

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
