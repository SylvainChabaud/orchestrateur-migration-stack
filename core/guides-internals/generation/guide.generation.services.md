# 📘 Guide Génération — Services  
*(Domaine : **services** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **services** explique comment transformer les résultats de la Phase 2 en **services métiers complets** pour `${project.pageName}`.

Un service représente :

- une unité fonctionnelle (chargement, création, mise à jour, traitement)
- une interaction avec une API
- une orchestration de données
- des validations métier
- un point central permettant aux stores, hooks et pages d’obtenir des données cohérentes

Le domaine reste **agnostique** :  
➡️ La forme exacte du service dépend des **stack-guides**.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2 (obligatoires)

- `mapping.services.json` → cœur du domaine
- `mapping.types.json` → pour définir inputs/outputs typés
- `mapping.data.json` → pour les dépendances API/DTO
- `mapping.logic.json` → pour la structure logique interne

Chaque mapping enrichit la compréhension de ce que doit contenir un service.

### 2.2. Structure projet (Phase 0)

- `project-structure.json`

Définit où doivent vivre les services.

### 2.3. Bridge Legacy → DSL

- `bridge-legacy-to-dsl.json`

Permet :

- la traçabilité UCR
- la documentation automatique
- la justification métier

### 2.4. Stack-guides (Phase 0)

- `guide.services.md`
- `guide.types.md`
- `guide.naming.md`
- `guide.conventions.md`

Ces guides définissent *l’implémentation finale* d’un service.

### 2.5. Guides internes globaux

- `guide.ucr.md`
- `guide.error-handling.md`
- `guide.schema-validation.md`

---

## 3. 🧩 Structure d’entrée typique

### Exemple minimal de `mapping.services.json`

```jsonc
{
  "services": [
    {
      "name": "CreateUser",
      "inputs": [
        { "name": "email", "type": "string" },
        { "name": "password", "type": "string" }
      ],
      "output": { "type": "User" },
      "logic": ["validateInput", "callApi", "mapResult"],
      "ucr": ["UCR-123", "UCR-456"]
    }
  ]
}
```

---

## 4. 📤 Outputs attendus

### 4.1. Fichiers services

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/services/`

Exemple :

`CreateUserService.ext`

Un fichier de service doit contenir :

- documentation détaillée (UCR, description)
- inputs typés
- outputs typés
- logique interne
- gestion d’erreurs
- dépendances éventuelles
- modèle de retour standardisé selon la stack

### 4.2. Métadonnées du domaine

`generation.services.meta.json`

Contient :

- `domain = "services"`
- `filesGenerated[]`
- `statistics.totalFiles`
- `validation.status`

---

## 5. 🔍 Patterns Conceptuels

### 5.1. AST Services

Chaque service doit être représenté par une AST contenant :

- `serviceName`
- `inputParams[]`
- `outputType`
- `logicSteps[]`
- `dataDependencies[]`
- `errors[]` (si fournis)
- `ucrTrace[]`

### 5.2. Patterns typiques de services

Selon les stack-guides :

- **API Fetcher**  
  - input → validation → appel API → mapping output

- **Computational Service**  
  - input → transformation pure → output  

- **Aggregating Service**  
  - combines data from multiple APIs or stores

- **Validation Service**  
  - returns structured validation results  

### 5.3. Gestion des erreurs

Le guide peut décrire :

- comment représenter les erreurs  
- comment structurer les exceptions  
- comment documenter les codes d’erreur  

### 5.4. Documentation & UCR

Chaque service doit inclure :

- un bloc de documentation
- la liste des UCR
- l’explication métier du rôle du service

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes

- service sans inputs ou outputs
- service référencé mais absent du mapping
- type introuvable
- logique interne incohérente
- échec d’un pattern imposé par `guide.services.md`

### Non bloquantes

- documentation partielle
- absence d’UCR
- absence de logique secondaire

---

## 7. ✔️ Checklist finale du domaine

Avant validation :

- [ ] `mapping.services.json` chargé & valide  
- [ ] AST Services construite  
- [ ] types d’inputs/outputs valides  
- [ ] stack-guides appliqués  
- [ ] fichiers écrits dans `src_new/services/`  
- [ ] métadonnée de génération écrite  
- [ ] aucune erreur critique  

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
