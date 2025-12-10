# 📘 Guide Génération — Components Atoms  
*(Domaine : **atoms** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Les **atoms** sont les éléments UI fondamentaux.  
Ils doivent être :

- simples,
- accessibles,
- minimalistes,
- réutilisables,
- sans logique métier.

Ils représentent les briques les plus petites : boutons, inputs, labels, icônes, etc.

Le domaine reste **agnostique** :  
➡️ La forme exacte de l'atom (composant, template, fonction) dépend des **stack-guides UI**.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2

- `mapping.atoms.json` → description principale des atoms
- `mapping.ui.json` → règles UI globales
- `mapping.types.json` → props typées

### 2.2. Structure projet

- `project-structure.json`

### 2.3. Bridge Legacy → DSL

- `bridge-legacy-to-dsl.json`

Pour :

- props héritées,
- labels,
- contraintes UCR.

### 2.4. Stack-guides UI

Obligatoires :

- `guide.ui.atoms.md`
- `guide.ui.props.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- signature des atoms,
- structuration UI,
- composition des props,
- style et conventions,
- accessibilité.

---

## 3. 🧩 Structure d’entrée typique

### Extrait `mapping.atoms.json`
```jsonc
{
  "atoms": [
    {
      "name": "ButtonAtom",
      "props": [
        { "name": "label", "type": "string" },
        { "name": "disabled", "type": "boolean", "default": false }
      ],
      "uiRole": "button",
      "ucr": ["UCR-900"]
    }
  ]
}
```

---

## 4. 📤 Outputs

### 4.1. Composants atoms

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/components/atoms/`

Exemples :

- `ButtonAtom.ext`
- `InputAtom.ext`

Chaque atom doit :

- exposer des props typées
- avoir un rendu UI minimal
- être dépourvu de logique métier
- documenter ses UCR si nécessaire

### 4.2. Métadonnées

`generation.atoms.meta.json` :

- liste des atoms générés
- nombre de fichiers
- statut de validation

---

## 5. 🔍 Patterns conceptuels

### 5.1. AST Atom

Contient :

- `atomName`
- `props[]` (nom, type, default)
- `uiRole`
- `accessibility` (si défini)
- `ucrTrace[]`

### 5.2. Types d’atoms

- **InputAtoms** : champs simples
- **DisplayAtoms** : labels, textes
- **ActionAtoms** : boutons
- **IconAtoms** : icônes ou pictos
- **DecorativeAtoms** : visuels UI

### 5.3. API exposée

Agnostique, typiquement :

```txt
Atom(props)
```

### 5.4. Accessibilité UI

Selon le guide :

- roles
- aria-labels
- tabIndex
- fallback text

### 5.5. Documentation & UCR

Chaque atom doit préciser :

- son rôle UI
- UCR associées (si UI liée à un besoin métier)

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes

- `uiRole` absent
- props non typées
- nom invalide
- type inconnu

### Non bloquantes

- absence d’UCR
- absence d’accessibilité

---

## 7. ✔️ Checklist finale

- [ ] `mapping.atoms.json` chargé  
- [ ] props typées  
- [ ] uiRole défini  
- [ ] rendu UI minimal généré  
- [ ] fichier écrit dans `components/atoms/`  
- [ ] métadonnée générée  
- [ ] aucune erreur critique  

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
