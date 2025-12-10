# 📘 Guide Génération — Hooks Data  
*(Domaine : **hooks-data** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **hooks-data** décrit comment générer les **hooks dédiés à la gestion et préparation des données**.

Un hook data :

- obtient des données depuis des services/stores
- les transforme, normalise ou combine
- gère leur cycle de vie (loading, error, success)
- expose une API simple aux composants/pages
- peut gérer refresh, polling, revalidation
- applique des UCR liées à l’obtention/présentation des données

Le domaine reste **agnostique**, la stack finale dicte les patterns via les **stack-guides**.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2

Les mappings utilisés :

- `mapping.hooks-data.json` → description principale du hook data
- `mapping.data.json` → transformations, normalisation, DTO
- `mapping.services.json` → dépendances externes
- `mapping.stores.json` → sources locales d’état

### 2.2. Structure projet

`project-structure.json` → indique où placer les fichiers générés.

### 2.3. Bridge Legacy → DSL

`bridge-legacy-to-dsl.json` :

- rattacher les règles métier
- appliquer les UCR
- expliciter les transformations

### 2.4. Stack-guides

Obligatoires :

- `guide.hooks-data.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

Les stack-guides définissent :

- comment représenter la donnée exposée
- comment structurer loading/error
- comment intégrer polling, refresh
- comment nommer le hook et ses retours
- comment structurer les imports

### 2.5. Guides internes globaux

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 3. 🧩 Structure d’entrée typique

### Exemple `mapping.hooks-data.json`
```jsonc
{
  "hooksData": [
    {
      "name": "useCampaignData",
      "dataSources": ["GetCampaignService"],
      "stateDependencies": ["CampaignStore"],
      "transformations": ["mapCampaignDto", "computeDerivedState"],
      "outputs": ["campaign", "isLoading", "error"],
      "polling": { "intervalMs": 5000 },
      "ucr": ["UCR-400"]
    }
  ]
}
```

---

## 4. 📤 Outputs attendus

### 4.1. Fichiers de hooks data

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/data/`

Exemple attendu (agnostique) :

- `useCampaignData.ext`
- `useUserData.ext`

Chaque hook doit exposer :

- les données prêtes à l’emploi
- les états dérivés (computed)
- gestion loading/error
- mécanismes de rafraîchissement
- documentation UCR

### 4.2. Métadonnées du domaine

`generation.hooks-data.meta.json` :

- fichiers générés
- stats globales
- statut de validation

---

## 5. 🔍 Patterns conceptuels

### 5.1. AST Hook Data

Chaque hook possède :

- `hookName`
- `dataSources[]` (services de lecture)
- `stateDependencies[]`
- `transformations[]`
- `pollingOptions`
- `refreshRules`
- `outputs[]`
- `ucrTrace[]`

### 5.2. Types de hooks data

#### **Simple Fetch Hook**
- service derrière
- loading/error/data

#### **Composite Data Hook**
- combine plusieurs services ou stores

#### **Derived Data Hook**
- applique transformations complexes

#### **Polling Hook**
- rafraîchit périodiquement

### 5.3. API exposée

Agnostique, mais typiquement :

```txt
return { data, isLoading, error, refresh }
```

ou :

```txt
return { state, computed, actions }
```

### 5.4. Gestion des erreurs

Selon `guide.error-handling.md` :

- erreurs de transport
- erreurs métier
- erreurs de transformation

### 5.5. Documentation UCR

Chaque hook doit mentionner :

- les règles métier couvertes
- les scénarios d’usage
- les limitations

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes
- dataSource inexistant
- transformation inconnue
- absence d’output
- hook non défini dans mapping principal

### Non bloquantes
- documentation incomplète
- absence de polling
- absence de UCR

---

## 7. ✔️ Checklist finale

- [ ] `mapping.hooks-data.json` chargé  
- [ ] AST construite  
- [ ] dépendances valides  
- [ ] fichiers écrits dans `src_new/hooks/data/`  
- [ ] métadonnée générée  
- [ ] aucune erreur critique  

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
