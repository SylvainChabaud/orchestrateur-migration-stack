# 🧭 Guide de Mapping — `mapping.i18n`

*(Projection des concepts `i18n.*` du DSL vers le système de traduction de la stack cible)*

---

## 1. 🎯 Rôle du mapping `i18n`

Le domaine `i18n.*` du DSL décrit tout ce qui touche à :

- les textes affichés à l’écran ;
- les clés de traduction ;
- les namespaces / fichiers de traductions ;
- les langues supportées ;
- la structure des messages (interpolations, pluriels, etc.).

La **Phase 1 — Analyse** a produit :

- `inventory.i18n.json` → inventaire des UCR `i18n.*` (textes, clés, messages) ;
- les liens éventuels avec :
  - des UCR de structure (`structure.*`) ;
  - des UCR de layout (`layout.*`) ;
  - des UCR de logique (`logic.*`) qui formatent ou combinent des textes.

La **Phase 2 — Stage 33 — mapping.i18n** doit :

> **Projeter chaque UCR `i18n.*` vers la mécanique i18n de la stack cible**,  
> par exemple :
> - fichiers de traductions (JSON, TS, YAML…) ;
> - namespaces (par page, par domaine fonctionnel…) ;
> - clés de traduction normalisées ;
> - hooks ou helpers (`useTranslation`, `t`, etc.).

L’objectif est d’obtenir un plan I18N clair : chaque texte DSL est relié à une **clé de traduction** dans la stack cible, avec un emplacement précis et une stratégie de structuration.

---

## 2. 📦 Format JSON racine (`mapping.i18n.json`)

Le fichier est écrit dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.i18n.json`

Structure racine attendue :

```json
{
  "domain": "i18n",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Champs principaux :

- `domain` : `"i18n"`  
- `pageName` : `${project.pageName}`  
- `sourceEntry` : `${paths.legacySource}` (référence informative)  
- `items` : tableau de `MappingItem`  
- `validation` : état global du mapping et liste d’issues

---

## 3. 🔗 Schéma d’un `MappingItem` I18N

### 3.1. Schéma générique

Chaque `MappingItem` relie :

- un UCR `i18n.*` issu de `inventory.i18n.json` ;
- à une clé / ressource i18n dans la stack cible.

```jsonc
{
  "ucr": "string",
  "fromDsl": "string",
  "sourceInventoryRef": {
    "file": "inventory.i18n.json",
    "domain": "i18n",
    "itemUcr": "string"
  },
  "toStack": {
    "stackKind": "string",
    "targetId": "string",
    "targetPath": "string",
    "targetLayer": "string",
    "targetTechnology": "string",
    "targetPattern": "string",
    "hints": []
  },
  "relations": {
    "structureUcrs": [],
    "layoutUcrs": [],
    "logicUcrs": [],
    "namespaces": [],
    "languages": []
  },
  "metadata": {
    "isCritical": false,
    "priority": "normal",
    "notes": ""
  }
}
```

### 3.2. Champs obligatoires

- `ucr`  
  - Identifiant de mapping unique dans `mapping.i18n.json`.  
  - Préfixe recommandé : `map-i18n-…` dérivé de l’UCR inventaire.

- `fromDsl`  
  - Concept de la famille `i18n.*` (par exemple : `i18n.text`, `i18n.label`, `i18n.message`, `i18n.errorMessage`, `i18n.placeholder`, etc.).

- `sourceInventoryRef.file`  
  - Toujours `"inventory.i18n.json"`.

- `sourceInventoryRef.domain`  
  - Toujours `"i18n"`.

- `sourceInventoryRef.itemUcr`  
  - UCR exact dans l’inventaire I18N.

- `toStack.stackKind`  
  - Type d’artefact i18n, par exemple :
    - `"translationKey"` ;
    - `"namespaceFile"` ;
    - `"sharedMessage"` ;
    - `"validationMessage"` ;
    - `"errorDictionaryEntry"`.

- `toStack.targetId`  
  - Clé ou identifiant final, par ex. :
    - `campaignsDetail.title` ;
    - `campaignsDetail.errors.budgetTooLow` ;
    - `common.buttons.save`.

- `toStack.targetPath`  
  - Fichier / ressource de traduction, dérivé de `project-structure.json`, par ex. :  
    - `src/locales/fr/campaignsDetail.json` ;  
    - `src/locales/en/campaignsDetail.json` ;  
    - ou toute organisation décidée par la stack.

- `toStack.targetLayer`  
  - `"i18n"` ou `"presentation"` selon la convention retenue.

### 3.3. Champs optionnels recommandés

- `toStack.targetTechnology`  
  - Technologie i18n : `"i18next"`, `"react-intl"`, `"next-intl"`, `"custom-i18n"`, etc.

- `toStack.targetPattern`  
  - Pattern : `"namespacedJson"`, `"flatJson"`, `"messagesFile"`, `"dictionaryObject"`, etc.

- `toStack.hints[]`  
  - Conseils pratiques, ex. : `"Utiliser le namespace 'campaignsDetail'"`, `"Prévoir pluriels"`…

- `relations.structureUcrs`  
  - UCR de structure attachés à ce texte (composant, section, vue).

- `relations.layoutUcrs`  
  - UCR de layout si le texte est lié à une zone spécifique.

- `relations.logicUcrs`  
  - UCR de logique si le texte est produit ou combiné par une règle métier.

- `relations.namespaces`  
  - Noms de namespace i18n (ex. `["campaignsDetail"]`).

- `relations.languages`  
  - Langues cibles, ex. `["fr", "en"]`.

---

## 4. ⚙️ Entrées requises pour `mapping.i18n`

### 4.1. Configuration (obligatoire)

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `stack.custom`
- `gates.*`
- `stages.*`

### 4.2. Artefacts de la Phase 0

- Structure cible du projet :  
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- Bridge Legacy → DSL :  
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- Guides de stack (i18n / locales) :  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.i18n.md` (si présent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.locales.md` (ou équivalent)

### 4.3. Inventaires de la Phase 1

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.i18n.json` (inventaire primaire)  
- `inventories-summary.json`

### 4.4. Mappings Phase 2 déjà produits

Dans `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`  
- `mapping.layout.json` (si existant)  
- éventuellement `mapping.logic.json` (si les règles de formatage sont importantes)

---

## 5. 🧠 Règles d’interprétation

1. **Pas de relecture du Legacy**  
   - Le stage ne lit jamais le code Legacy.  
   - Tout se base sur l’inventaire I18N, la structure cible, les guides et les mappings existants.

2. **Convergence vers une stratégie i18n cohérente**  
   - Clés de traduction normalisées ;  
   - Namespaces structurés (par page, domaine fonctionnel…) ;  
   - Respect des conventions de la stack (snakeCase, camelCase, dot.notation, etc.).

3. **Respect de `project-structure.json`**  
   - Les fichiers cibles de traduction (JSON, TS, etc.) doivent correspondre à l’organisation de la stack.

4. **Regroupement logique**  
   - Il est possible (et souhaitable) de regrouper plusieurs UCR I18N dans une même ressource (par ex. un namespace).

5. **Traçabilité totale**  
   - Les relations vers `structure.*`, `layout.*` et éventuellement `logic.*` permettent de savoir :  
     - quel composant / section affiche le texte ;
     - dans quelle zone de page ;
     - via quelle logique métier.

---

## 6. Exemple simplifié

```json
{
  "domain": "i18n",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/legacy/pages/CampaignsDetail/index.js",
  "items": [
    {
      "ucr": "map-i18n-title-CampaignsDetail-1",
      "fromDsl": "i18n.text",
      "sourceInventoryRef": {
        "file": "inventory.i18n.json",
        "domain": "i18n",
        "itemUcr": "i18n-text-CampaignsDetail-title-1"
      },
      "toStack": {
        "stackKind": "translationKey",
        "targetId": "campaignsDetail.title",
        "targetPath": "src/locales/fr/campaignsDetail.json",
        "targetLayer": "i18n",
        "targetTechnology": "i18next",
        "targetPattern": "namespacedJson",
        "hints": [
          "Prévoir clé miroir en 'en' dans src/locales/en/campaignsDetail.json"
        ]
      },
      "relations": {
        "structureUcrs": ["view-page-CampaignsDetail-1"],
        "layoutUcrs": ["layout-zone-header-1"],
        "logicUcrs": [],
        "namespaces": ["campaignsDetail"],
        "languages": ["fr", "en"]
      },
      "metadata": {
        "isCritical": true,
        "priority": "high",
        "notes": "Titre principal de la page de détail campagne."
      }
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

## 7. ✅ Checklist de validation

- [ ] `inventory.i18n.json` est présent et déclaré valide dans `inventories-summary.json`  
- [ ] `mapping.structure.json` est accessible  
- [ ] Les fichiers de targetPath sont compatibles avec la stratégie i18n / locales définie dans les guides  
- [ ] Chaque UCR I18N important a une projection vers une clé de traduction  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] Les `relations` sont renseignées lorsque l’information est disponible (structure/layout/logic)  

---

© 2025 — ai-orchestrator-v4  
*Guide concret pour le domaine `mapping.i18n` (Stage 33 — Phase 2 : Interprétation)*
