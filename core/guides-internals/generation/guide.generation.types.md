# 📘 Guide Génération — Types

*(Domaine : **types** — Phase 3 Generation — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **types** explique comment générer des **types statiques** pour `${project.pageName}` à partir :

- des **mappings de Phase 2**,
- des **stack-guides** de la stack finale,
- des **conventions globales** (naming, UCR, structure).

Les types représentent :

- les structures de données internes,
- les entités manipulées par les services/stores/hooks/pages,
- les schémas d’entrée/sortie des domaines,
- les DTO dérivés.

Aucun langage n’est imposé :  
➡️ Le domaine doit rester **agnostique**, seul le stack-guide décide.

---

## 2. 🔌 Entrées du domaine

### 2.1. Mappings Phase 2 (obligatoires)

Le domaine consomme principalement :

- `mapping.types.json`
- `mapping.structure.json`
- `mapping.data.json`

Chacun contribue à enrichir la description des types.

### 2.2. Structure projet (Phase 0)

- `project-structure.json`

Pour déterminer les emplacements attendus des fichiers types.

### 2.3. Bridge Legacy → DSL

- `bridge-legacy-to-dsl.json`

Pour rattacher chaque propriété à une UCR (User-Centric Requirement).

### 2.4. Stack-guides (Phase 0)

Indispensables :

- `guide.types.md`
- `guide.naming.md`
- `guide.conventions.md`

Ils définissent :

- la syntaxe des types,
- les règles de nommage,
- la structure des fichiers générés.

### 2.5. Guides internes globaux

- `guide.ucr.md`  
- `guide.schema-validation.md` (si présent)

---

## 3. 🧩 Structure d’entrée typique

### Exemple `mapping.types.json`

```jsonc
{
  "types": [
    {
      "name": "User",
      "properties": [
        { "name": "id", "type": "string", "nullable": false },
        { "name": "email", "type": "string", "nullable": false }
      ]
    }
  ]
}
```

### Exemple enrichi via structure et data

`mapping.structure.json` peut apporter :

- des types dérivés,
- des relations,
- des contraintes.

`mapping.data.json` peut apporter :

- des types de réponse API,
- des types d’erreurs,
- des types pour des collections.

---

## 4. 📤 Outputs attendus

### 4.1. Fichiers types

Sous :

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/types/`

Pour chaque type :

- Un fichier `.ext` (extension définie par la stack)
- Contenant :
  - la définition du type
  - les propriétés
  - la doc générée
  - les UCR tracées

### 4.2. Métadonnées de domaine

`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.types.meta.json`

Contient :

- `domain = "types"`
- `filesGenerated[]`
- `statistics.totalFiles`
- `validation.status`

---

## 5. 🔍 Patterns Conceptuels

### 5.1. Transformation AST

Le domaine doit produire une **AST Types** :

- un nœud par type
- un nœud par propriété
- relations via `mapping.structure.json`

Cette AST est ensuite transformée en fichiers via les stack-guides.

### 5.2. Application des conventions

- Naming : PascalCase, camelCase, snake_case… selon la stack
- Ordre des propriétés
- Documentation intégrée (si la stack le permet)

### 5.3. Noms dérivés

Selon `guide.naming.md`, un type peut générer :

- `<TypeName>Input`
- `<TypeName>Output`
- `<TypeName>Error`

### 5.4. Gestion des nullabilités & types composites

Le guide doit préciser :

- comment représenter un type nullable
- comment représenter les listes
- comment représenter les unions

---

## 6. ⚠️ Gestion des erreurs

### Bloquantes

- `mapping.types.json` absent
- type déclaré mais sans propriété
- property sans `type`
- impossibilité d’appliquer un pattern de stack-guide

### Non bloquantes

- propriétés inutilisées
- documentation absente
- warnings de structure

---

## 7. 📚 Exemples conceptuels (notation agnostique)

### 7.1. Type d'entité simple

**Entrée** `mapping.types.json` :
```json
{
  "name": "Campaign",
  "ucr": "type-campaign-1",
  "properties": [
    { "name": "id", "type": "string", "nullable": false },
    { "name": "name", "type": "string", "nullable": false },
    { "name": "budget", "type": "number", "nullable": true },
    { "name": "status", "type": "enum", "enumValues": ["draft", "active", "archived"], "nullable": false }
  ]
}
```

**Sortie conceptuelle** `Campaign.<ext>` :
```
Type Campaign {
  @ucr: "type-campaign-1"
  @generated: "ai-orchestrator-v4 - Stage 50"
  
  id: string (required)
  name: string (required)
  budget: number (optional)
  status: CampaignStatus (required)
}

Enum CampaignStatus {
  draft, active, archived
}
```

*Note* : La syntaxe finale (interface, class, struct, record, etc.) dépend du **stack-guide**.

### 7.2. Type DTO (Data Transfer Object)

**Entrée** `mapping.types.json` :
```json
{
  "name": "CreateCampaignRequest",
  "category": "dto-input",
  "properties": [
    { "name": "name", "type": "string", "nullable": false },
    { "name": "budget", "type": "number", "nullable": true }
  ]
}
```

**Sortie conceptuelle** `CreateCampaignRequest.<ext>` :
```
Type CreateCampaignRequest {
  @category: "dto-input"
  
  name: string (required)
  budget: number (optional)
}

Type CreateCampaignResponse {
  campaign: Campaign (required)
  success: boolean (required)
}
```

### 7.3. Type avec relations

**Entrée** `mapping.types.json` :
```json
{
  "name": "CampaignDetail",
  "properties": [
    { "name": "campaign", "type": "Campaign", "nullable": false },
    { "name": "keywords", "type": "array", "itemType": "Keyword", "nullable": false },
    { "name": "statistics", "type": "CampaignStats", "nullable": true }
  ]
}
```

**Sortie conceptuelle** `CampaignDetail.<ext>` :
```
Type CampaignDetail {
  @dependencies: [Campaign, Keyword, CampaignStats]
  
  campaign: Campaign (required)
  keywords: Array<Keyword> (required)
  statistics: CampaignStats (optional)
}
```

### 7.4. Type avec validation de schéma

**Entrée** `mapping.types.json` avec contraintes :
```json
{
  "name": "Campaign",
  "properties": [
    { "name": "id", "type": "string", "validation": { "format": "uuid" } },
    { "name": "name", "type": "string", "validation": { "minLength": 1, "maxLength": 255 } },
    { "name": "budget", "type": "number", "validation": { "min": 0 }, "nullable": true }
  ]
}
```

**Sortie conceptuelle** avec règles de validation :
```
Type Campaign {
  id: string {
    format: "uuid"
    required: true
  }
  
  name: string {
    minLength: 1
    maxLength: 255
    required: true
  }
  
  budget: number {
    min: 0
    optional: true
  }
}
```

*Note* : Selon la stack, ces validations peuvent être traduites en :
- annotations/decorators
- schémas de validation (Zod, Yup, Joi, etc.)
- contraintes de base de données
- règles métier dans les services

### 7.5. Type avec documentation enrichie

**Sortie conceptuelle** avec métadonnées complètes :
```
Type Campaign {
  @ucr: "type-campaign-1"
  @legacySource: "src/packages/promo-boost/components/campaignsDetail/types.js"
  @generatedBy: "ai-orchestrator-v4"
  @generatedAt: "2025-12-07T10:30:00Z"
  @stage: "50-generate-types"
  @description: "Campaign entity representing a marketing campaign"
  
  id: string {
    @description: "Unique identifier (UUID v4)"
    required: true
  }
  
  name: string {
    @description: "Campaign display name (1-255 characters)"
    required: true
  }
  
  budget: number {
    @description: "Campaign budget in monetary units"
    @constraints: { min: 0 }
    optional: true
  }
  
  status: CampaignStatus {
    @description: "Current campaign status"
    required: true
  }
}
```

### 7.6. Types utilitaires dérivés

**Génération automatique de variantes** :

À partir d'un type de base, le générateur peut produire des variantes selon les besoins :

```
Type Campaign { id, name, budget, status }

→ CampaignUpdate = Campaign sans 'id' + tous champs optionnels
→ CampaignCreate = Campaign sans 'id'
→ CampaignComplete = Campaign avec tous champs requis
→ CampaignReadonly = Campaign en lecture seule (immutable)
→ CampaignPartial = Campaign avec tous champs optionnels
```

Ces variantes dépendent du **stack-guide** qui définit les patterns de transformation disponibles.

---

## 8. 🎨 Patterns conceptuels à respecter

### 8.1. Conventions de nommage génériques

Le **stack-guide** définit les conventions, exemples typiques :

- **Types d'entités** : PascalCase ou UpperCamelCase (`Campaign`, `User`)
- **Types DTO** : PascalCase avec suffixe (`CreateCampaignRequest`, `UserResponse`)
- **Énumérations** : PascalCase pour le type, conventions variables pour les valeurs
- **Fichiers** : Aligné avec le type principal contenu

Exemples de conventions selon les stacks :
- Stack orientée objet : `Campaign`, `CampaignService`
- Stack fonctionnelle : `campaign`, `create_campaign_request`
- Stack mixte : définir explicitement dans le stack-guide

### 8.2. Organisation logique des types

Organisation recommandée (adaptable selon stack-guide) :

```
types/
├── entities/          # Types représentant les entités métier
│   ├── Campaign
│   ├── Keyword
│   └── User
├── dtos/             # Types pour transfert de données (API)
│   ├── CreateCampaignRequest
│   ├── UpdateCampaignRequest
│   └── ApiResponses
├── enums/            # Énumérations et constantes typées
│   ├── CampaignStatus
│   └── UserRole
├── utilities/        # Types helpers et génériques
│   ├── Pagination
│   └── ApiError
└── index            # Point d'entrée centralisé (si applicable)
```

### 8.3. Exports et dépendances

**Principe** : Chaque fichier de type doit :
- Exposer ses définitions via le mécanisme d'export de la stack
- Déclarer explicitement ses dépendances vers d'autres types
- Éviter les dépendances circulaires

**Exemple conceptuel** :
```
// Fichier Campaign expose Campaign et CampaignStatus
exports: [Campaign, CampaignStatus]
dependencies: []

// Fichier CampaignDetail consomme Campaign, Keyword, CampaignStats
exports: [CampaignDetail]
dependencies: [Campaign, Keyword, CampaignStats]

// Fichier index agrège tous les exports
exports: [* from entities, * from dtos, * from enums]
```

### 8.4. Anti-patterns génériques à éviter

❌ **Ne PAS faire** :

- **Types génériques ou faibles** :
  ```
  Type Campaign { data: any }  // ❌ Perte de typage
  ```

- **Types vides ou inutiles** :
  ```
  Type EmptyType {}  // ❌ Aucune valeur ajoutée
  ```

- **Dépendances circulaires** :
  ```
  Campaign → User → Campaign  // ❌ Cycle de dépendances
  ```

- **Duplication de définitions** :
  ```
  Type Campaign { ... }
  Type CampaignEntity { ... }  // ❌ Même structure, noms différents
  ```

✅ **À faire** :

- **Types précis et documentés** :
  ```
  Type Campaign {
    @description: "Marketing campaign entity"
    data: CampaignData (required)
  }
  ```

- **Types avec propriétés significatives** :
  ```
  Type CampaignMetadata {
    createdAt: DateTime (required)
    updatedAt: DateTime (required)
    author: User (required)
  }
  ```

- **Hiérarchie claire** :
  ```
  Type BaseCampaign { id, name }
  Type CampaignDetail extends BaseCampaign { budget, status, keywords }
  ```

- **Réutilisation via composition** :
  ```
  Type Timestamped { createdAt, updatedAt }
  Type Campaign { id, name, ...Timestamped }
  ```

---

## 9. ✔️ Checklist finale du domaine

Avant que la génération soit validée :

- [ ] `mapping.types.json` chargé & valide  
- [ ] AST Types construite correctement  
- [ ] Tous les types ont des propriétés valides et typées  
- [ ] stack-guides appliqués sans erreur  
- [ ] Conventions de nommage respectées (selon stack-guide)
- [ ] Mécanisme d'export/import configuré (selon stack)
- [ ] Documentation complète avec UCR et métadonnées
- [ ] Types utilitaires dérivés générés si nécessaire
- [ ] Schémas de validation générés si stack l'exige
- [ ] Fichiers écrits dans `src_new/types/` avec extension appropriée
- [ ] `generation.types.meta.json` écrit et valide  
- [ ] Aucune dépendance circulaire détectée
- [ ] Validation schéma JSON passée (si `enableSchemaValidation = true`)

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
