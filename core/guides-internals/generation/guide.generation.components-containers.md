# 📘 Guide Génération — Components Containers  
*(Domaine : **containers** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Les **containers** sont les composants UI de haut niveau pour `${project.pageName}`.

Ils :

- orchestrent **hooks logique + hooks data**,  
- organisent l’affichage via un layout,  
- transmettent données et callbacks aux atoms/components,  
- structurent clairement la page sans logique métier.

Un container est **agnostique** :  
sa forme finale dépend des **stack-guides UI**.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2

- `mapping.containers.json` → mapping principal  
- `mapping.ui.json` → règles UI globales  
- `mapping.hooks-logic.json`  
- `mapping.hooks-data.json`

### 2.2. Structure projet
`project-structure.json`

### 2.3. Bridge Legacy → DSL
`bridge-legacy-to-dsl.json` :  

Permet de :

- rattacher sections UI aux UCR  
- hériter des intentions UI  
- justifier la structure du container

### 2.4. Stack-guides UI

- `guide.ui.containers.md`
- `guide.ui.layout.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- structure interne du container  
- patterns layout  
- conventions des props et sections  
- nommage des composants  

### 2.5. Guides internes globaux

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 3. 🧩 Structure d’entrée typique

### mapping.containers.json
```jsonc
{
  "containers": [
    {
      "name": "CampaignFormContainer",
      "hooksLogic": ["useCampaignValidation"],
      "hooksData": ["useCampaignData"],
      "sections": ["header", "form", "footer"],
      "uiRole": "formContainer",
      "ucr": ["UCR-1200"]
    }
  ]
}
```

---

## 4. 📤 Outputs

### 4.1. Composants containers générés

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/containers/`

Exemples :

- `CampaignFormContainer.ext`
- `BudgetPanelContainer.ext`

Un container doit :

- connecter hooks logique + data  
- organiser l’affichage via layout  
- exposer une structure UI stable  
- documenter ses UCR  

### 4.2. Métadonnées

`generation.containers.meta.json` :

- `filesGenerated`  
- statistiques  
- validation  

---

## 5. 🔍 Patterns conceptuels

### 5.1. AST Container

Un container doit contenir :

- `containerName`
- `hooksLogic[]`
- `hooksData[]`
- `atoms[]` (dérivés du mapping UI)
- `sections[]`
- `uiRole`
- `props[]`
- `ucrTrace[]`

### 5.2. Types de containers

#### **FormContainer**
- orchestre un formulaire  
- sections : header / form / footer  

#### **PanelContainer**
- gère affichage de données, pagination, actions  

#### **LayoutContainer**
- structure un groupe d’éléments UI  

### 5.3. API exposée

Un container retourne une construction UI, format agnostique, du type :

```txt
render(layoutStructure)
```

### 5.4. Accessibilité

Le guide peut exiger :

- roles  
- aria-*  
- navigabilité clavier  

### 5.5. UCR Documentation

Chaque container doit préciser :

- règles UI et intentions  
- interactions prises en charge  
- contexte métier (via bridge)

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes

- hook référencé mais inexistant  
- section UI manquante  
- rôle UI incohérent  
- props non définies  

### Non bloquantes

- documentation incomplète  
- absence d’UCR  

---

## 7. ✔️ Checklist finale

- [ ] mapping chargé et valide  
- [ ] hooks existants  
- [ ] sections conformes  
- [ ] container généré dans `components/containers/`  
- [ ] métadonnée écrite  
- [ ] aucune erreur critique  

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
