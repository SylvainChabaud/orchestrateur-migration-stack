# 📘 Guide Génération — Hooks Logic  
*(Domaine : **hooks-logic** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **hooks-logic** décrit comment générer les **hooks métier** utilisés dans `${project.pageName}`.

Un hook de logique :

- encapsule un **comportement métier complet**
- orchestre **stores + services**
- expose une **API simplifiée** aux composants/pages
- gère **états transitoires**, **synchronisations**, **effets**, **logiques conditionnelles**
- implémente des **UCR complètes**

Le domaine reste **agnostique**, la structure finale dépend des **stack-guides**.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2
Requis :

- `mapping.hooks-logic.json`
- `mapping.logic.json`
- `mapping.stores.json`
- `mapping.services.json`

Ces mappings décrivent :
- quelles actions/services/stores composent chaque hook
- dans quel ordre les exécuter
- quelles UCR et quelles règles métier y sont associées

### 2.2. Structure projet
`project-structure.json`  
→ Définit où doivent être placés les hooks générés.

### 2.3. Bridge Legacy → DSL
`bridge-legacy-to-dsl.json`  
→ Permet d’injecter des UCR, descriptions métier et traces.

### 2.4. Stack-guides obligatoires
- `guide.hooks-logic.md`
- `guide.services.md`
- `guide.stores.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :
- la signature du hook
- la structure interne
- l’usage de stores/services
- les erreurs possibles
- conventions de nommage

### 2.5. Guides internes globaux
- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 3. 🧩 Structure d’entrée typique

### Extrait `mapping.hooks-logic.json`
```jsonc
{
  "hooksLogic": [
    {
      "name": "useCampaignValidation",
      "stateDependencies": ["CampaignStore"],
      "serviceDependencies": ["ValidateCampaignService"],
      "logic": ["loadDraft", "validateFields", "computeErrors"],
      "inputs": [{ "name": "campaignId", "type": "string" }],
      "outputs": [{ "name": "isValid", "type": "boolean" }],
      "ucr": ["UCR-200", "UCR-201"]
    }
  ]
}
```

---

## 4. 📤 Outputs attendus

### 4.1. Fichiers de hooks
Dans :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/hooks/logic/`

Exemples :
- `useCampaignValidation.ext`
- `useBudgetRecompute.ext`

Chaque hook doit contenir :

- documentation complète
- inputs / outputs
- orchestration des services
- interactions avec les stores
- gestion des erreurs
- respect des UCR

### 4.2. Métadonnées du domaine
`generation.hooks-logic.meta.json` :

- liste des hooks générés
- statistiques globales
- état de validation

---

## 5. 🔍 Patterns conceptuels

### 5.1. AST Hook Logic
L’AST contient :

- `hookName`
- `description`
- `inputs[]`
- `outputs[]`
- `stateDependencies[]`
- `serviceDependencies[]`
- `logicSteps[]`
- `ucrTrace[]`

### 5.2. Typologie des hooks

#### **Hook orchestration**
- combine plusieurs stores/services
- gère des scénarios métier complexes

#### **Hook validation**
- calcule des états dérivés
- exécute règles métier

#### **Hook transactionnel**
- effectue une séquence (validate → save → refresh)

### 5.3. API exposée (agnostique)

Exemples de patterns génériques :

- API retournant un objet :
```txt
return { state, actions, computedValues }
```

- API exposant un contrôleur :
```txt
return controller
```

- API événements :
```txt
return { onSubmit, onReset, onValidate }
```

### 5.4. Gestion des erreurs
Selon `guide.error-handling.md`, inclure :

- erreurs fonctionnelles
- erreurs système/transport
- possibilité de retourner un état d’erreur

### 5.5. Documentation UCR
Chaque hook doit exposer :

- règles métier couvertes
- contexte d’utilisation
- limitations éventuelles

---

## 6. ⚠️ Gestion des erreurs

### Erreurs bloquantes
- hook sans `logicSteps`
- référence à un service inexistant
- référence à un store inexistant
- inputs/outputs non typés

### Erreurs non bloquantes
- documentation incomplète
- absence d’UCR
- absence de description

---

## 7. ✔️ Checklist finale

- [ ] `mapping.hooks-logic.json` chargé  
- [ ] AST Hooks Logic construite  
- [ ] dépendances services/stores valides  
- [ ] stack-guides appliqués  
- [ ] fichiers écrits dans `src_new/hooks/logic/`  
- [ ] métadonnée écrite  
- [ ] aucune erreur critique  

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
