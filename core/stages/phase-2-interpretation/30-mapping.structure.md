# 🧩 Stage 30 – mapping.structure
**Phase :** Phase 2 — Interprétation  
**Précédent :** 26 – meta-inventory (inventories-summary)  
**Suivant :** 31 – mapping.layout

---

## 🎯 Objectif

Construire le fichier `mapping.structure.json` pour la page `${project.pageName}` en projetant chaque UCR `structure.*` de `inventory.structure.json` vers des artefacts de structure de la stack cible (vues, composants, conteneurs, formulaires, listes, zones de layout, etc.).  

Ce stage **ne relit jamais** le fichier Legacy. Il consomme exclusivement les inventaires de Phase 1 et les artefacts de la Phase 0 pour produire un plan de projection déterministe et traçable de la structure vers la stack cible.

---

## ⚙️ Entrées (Inputs)

> **Toutes les entrées DOIVENT être résolues à partir de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.**

### Configuration

Depuis `core/configs/project.config.yaml` :

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

### Code Legacy

- Aucun accès direct au Legacy.  
  - `${paths.legacySource}` n’est utilisé que comme valeur de référence dans `sourceEntry` au niveau racine de `mapping.structure.json`.

### Artefacts de la Phase 0 (lecture seule)

- **Structure cible du projet**  
  - `${paths.workspace}/projects/${project.name}/stack/project-structure.json`

- **Bridge Legacy → DSL** (contexte de détection des concepts DSL)  
  - `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

- **Guides de stack (structure / UI)**  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.structure.md` (ou équivalent)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.ui-components.md`

### Inventaires de la Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.structure.json` (inventaire primaire du stage 30)  
- `inventories-summary.json` (état global de la Phase 1 pour la page)

### Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping Structure**
  - `${paths.core}/guides-internals/mapping/guide.mapping.structure.md`
  - Fournit :
    - l'objectif du mapping de structure,
    - le schéma JSON contractuel de `mapping.structure.json`,
    - les règles de projection des UCR `structure.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.

---

## 📤 Sorties (Outputs)

> Ce stage produit **exactement un seul fichier** dans le workspace.

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.structure.json`

Racine attendue :

- `domain: "structure"`  
- `pageName: ${project.pageName}`  
- `sourceEntry: ${paths.legacySource}`  
- `items: MappingItem[]`  
- `validation: { status, issues[] }`  

Règles :

- La sortie doit respecter le schéma JSON attendu pour les mappings (par exemple `core/schemas/mapping.structure.schema.json` si disponible).  
- Aucun fichier ne doit être écrit dans `${paths.core}`.  
- Aucun autre fichier de mapping (`mapping.layout.json`, `mapping.logic.json`, etc.) ne doit être créé ou modifié par ce stage.

---

## 🧠 Actions (logique du stage)

### 1. Charger la configuration et le contexte de Phase 0

1.1. Charger `core/configs/project.config.yaml` et résoudre toutes les interpolations `${paths.*}`.  
1.2. Charger `project-structure.json` et construire en mémoire un modèle de l’arborescence cible pour `${project.pageName}` (dossiers de pages, composants, hooks, tests, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour disposer du contexte de classification des concepts `structure.*` (aucun nouveau parsing du Legacy).  
1.4. Charger les guides de stack pertinents (`guide.stack.md`, `guide.structure.md`, `guide.ui-components.md`) pour connaître les patterns et conventions structurels de la stack cible.

### 2. Charger les inventaires de Phase 1

2.1. Charger `inventory.structure.json` et itérer sur tous les items dont le concept DSL (`dsl`) appartient à la famille `structure.*`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `structure` pour `${project.pageName}` est bien présent et marqué comme valide.  
2.3. Si l’inventaire `structure` est manquant ou invalide :
- préparer un objet `mapping.structure` minimal avec `validation.status = "rejected"` et une issue explicite ;
- ne pas poursuivre la projection des UCR ;
- retourner un `Gate ❌`.

### 3. Initialiser l’objet racine du mapping

3.1. Construire un objet en mémoire de la forme :

```jsonc
{
  "domain": "structure",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

3.2. Cet objet sera appelé `mappingRoot` et servira de buffer jusqu’à l’écriture sur disque.

### 4. Projeter chaque UCR `structure.*` vers la stack cible

Pour chaque item de `inventory.structure.json` :

4.1. Lire :

- `item.ucr` (l’UCR inventaire) ;
- `item.dsl` (concept DSL, par ex. `structure.page`, `structure.form`, `structure.list`, etc.) ;
- les métadonnées utiles éventuelles (nom heuristique, hiérarchie parent/enfant, rôle, tags…).

4.2. Déterminer le `toStack.stackKind` approprié en fonction :

- du concept DSL (`structure.page`, `structure.component`, `structure.form`, etc.) ;
- des conventions décrites dans les guides de stack (ex : types de composants, dossiers dédiés, patterns) ;
- de la structure cible (`project-structure.json`) pour localiser la page et ses sous-éléments.

4.3. Construire un `toStack.targetId` :

- lisible par un humain,  
- stable d’une exécution à l’autre,  
- cohérent avec les conventions de nommage issues des guides de stack et de `project-structure.json`  
  (par ex. `CampaignsDetailView`, `CampaignForm`, `CampaignSummaryCard`).

4.4. Déduire `toStack.targetPath` à partir de `project-structure.json` :

- choisir les bons dossiers pour :
  - la page principale ;
  - les composants enfants ;
  - les sous-dossiers `components`, `sections`, etc. lorsque cela est défini ;
- **ne jamais inventer** un chemin qui n’est pas compatible avec la structure cible.

4.5. Fixer `toStack.targetLayer` :

- en général `"presentation"` pour la majorité des artefacts structurels ;
- `"application"` uniquement si la stack cible impose des conteneurs structurels spécifiques.

4.6. Optionnel : renseigner `toStack.targetTechnology`, `toStack.targetPattern`, `toStack.hints[]` selon les guides de stack (par ex. `pageComponent`, `formComponent`, `presentationalComponent`).

4.7. Construire un `MappingItem` :

- `ucr` : identifiant unique du mapping, idéalement dérivé de l’UCR inventaire, par exemple :  
  - `map-structure-${item.ucr}` ou une variante documentée dans `guide.ucr.md` ;
- `fromDsl` : concept DSL d’origine (`structure.page`, `structure.form`, etc.) ;
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.structure.json",
    "domain": "structure",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `toStack` : structure renseignée aux étapes 4.2 à 4.6 ;
- `relations` :
  - `structureUcrs` : peut rester vide ou contenir d’autres UCR du même domaine si plusieurs entrées sont regroupées dans un même artefact de stack ;
  - `routingUcrs`, `actionUcrs`, `eventUcrs` : à remplir si l’information est disponible ou facilement déductible sans re-parsing du Legacy (optionnel, best effort) ;
- `metadata` :
  - `isCritical = true` et `priority = "high"` pour :
    - les vues de page principales ;
    - les formulaires métiers centraux ;
    - les sections clés identifiées dans les guides ou les specs fonctionnelles.

4.8. Ajouter chaque `MappingItem` dans `mappingRoot.items[]`.

### 5. (Optionnel) Enrichir les relations à partir d’autres inventaires

5.1. Facultativement, charger d’autres inventaires (`inventory.routing.json`, `inventory.actions.json`, `inventory.events.json`, etc.) pour compléter les tableaux de `relations` (par ex. `routingUcrs`, `actionUcrs`, `eventUcrs`).  

5.2. Ne jamais créer de nouveaux UCR à ce stade : uniquement référencer des UCR déjà existants.  
5.3. Si certaines relations ne peuvent pas être résolues, laisser les tableaux correspondants vides sans faire échouer le stage.

### 6. Validation interne

6.1. Effectuer les contrôles suivants sur `mappingRoot` :

- `mappingRoot.domain === "structure"` ;
- `mappingRoot.pageName === project.pageName` ;
- tous les `MappingItem.ucr` sont uniques ;
- chaque `sourceInventoryRef.itemUcr` pointe vers un UCR existant dans `inventory.structure.json` ;
- pour chaque `MappingItem`, `toStack.stackKind`, `toStack.targetId`, `toStack.targetPath`, `toStack.targetLayer` sont renseignés.

6.2. **Validation du schéma JSON (optionnelle)**
   - Si `validation.enableSchemaValidation = true` dans la configuration :
     - Charger le schéma depuis `${validation.schemasPath}/mapping.structure.schema.json`
     - Valider `mappingRoot` contre ce schéma
     - En cas d'erreur de validation :
       - Mode `strict` : ajouter les erreurs dans `validation.issues[]`, fixer `status = "rejected"`, préparer `Gate ❌`
       - Mode `warning` : ajouter des warnings dans `validation.issues[]`, continuer normalement
     - Voir `${paths.core}/guides-internals/globals/guide.json-schema-validation.md` pour les détails d'implémentation

6.3. Si un problème bloquant est détecté (inventaire manquant, conflit majeur, erreur de schéma en mode strict) :

- ajouter une entrée descriptive dans `mappingRoot.validation.issues[]` ;
- fixer `mappingRoot.validation.status = "rejected"`.

6.4. Sinon :

- fixer `mappingRoot.validation.status = "valid"` ;
- s’assurer que `mappingRoot.validation.issues` est un tableau (éventuellement vide).

### 7. Écriture de la sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.structure.json`

7.2. Créer les dossiers manquants si nécessaire, sans écraser d’autres fichiers que `mapping.structure.json`.  
7.3. Ne modifier aucun autre fichier du workspace.

---

## ✅ Auto-Checks (résumé retourné par l’IA)

À la fin du stage, l’IA doit retourner dans sa réponse un petit résumé JSON (non écrit sur disque) :

```json
{
  "stageId": "30",
  "stageName": "mapping.structure",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

- `inputsAvailable = false` si une entrée obligatoire est manquante (config, project-structure, inventory.structure, etc.).  
- `schemaValidated = false` si la validation de schéma n’a pas été effectuée ou a échoué.  
- `outputsWritten = false` si le fichier de sortie n’a pas pu être écrit.

---

## 🧩 Gate

En fin de stage, le fichier doit contenir **exactement l’un** des blocs suivants :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` si au moins une erreur bloquante est rencontrée (inventaire manquant/invalide, schéma non respecté, impossibilité d’écrire la sortie, etc.).

---

## 📦 Stage suivant

> Continuer avec `31-mapping-layout.md` **uniquement si** `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
