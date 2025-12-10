# 🧩 Stage 33 — mapping.i18n

**Phase :** Phase 2 — Interprétation  
**Précédent :** 32 — mapping.styles  
**Suivant :** 34 — mapping.config

---

## 🎯 Objectif

Construire le fichier `mapping.i18n.json` pour la page `${project.pageName}` en projetant chaque UCR `i18n.*` issu de `inventory.i18n.json` vers :

- des clés de traduction ;
- des namespaces ;
- des fichiers de locales ;
- et, plus largement, la mécanique I18N de la stack cible.

Aucune relecture du Legacy n’est autorisée dans ce stage.

---

## ⚙️ Entrées requises

> Toutes les entrées sont dérivées de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### Configuration

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

### Artefacts de la Phase 0 (lecture seule)

- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.stack.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.i18n.md` (si présent)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.locales.md` (ou équivalent)

### Inventaires de la Phase 1 (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/` :

- `inventory.i18n.json`
- `inventories-summary.json`
### Guides internes (lecture seule, core)

Depuis `${paths.core}/guides-internals/` :

- **Guide de mapping i18n**
  - `${paths.core}/guides-internals/mapping/guide.mapping.i18n.md`
  - Fournit :
    - l'objectif du mapping i18n,
    - le schéma JSON contractuel de `mapping.i18n.json`,
    - les règles de projection des UCR `i18n.*` vers la stack cible,
    - les relations avec les autres mappings.

- **Guide UCR global**
  - `${paths.core}/guides-internals/globals/guide.ucr.md`
  - Utilisation :
    - garantir que les UCR de mapping sont uniques et cohérents,
    - assurer la traçabilité entre inventaires et mappings via les UCR.
### Mappings Phase 2 déjà produits (lecture seule)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json`
- `mapping.layout.json` (si existant)
- éventuellement d’autres mappings utiles (ex : `mapping.logic.json`)

Si l’un des inputs obligatoires est manquant ou invalide → le stage se termine en **Gate ❌**.

---

## 📤 Sortie

Ce stage produit **exactement un fichier** :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.i18n.json`

Racine attendue :

```jsonc
{
  "domain": "i18n",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

---

## 🧠 Actions (logique du stage)

### 1. Charger configuration et contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre toutes les interpolations `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître l’organisation des fichiers, y compris les dossiers de locales s’ils sont définis.  
1.3. Charger `bridge-legacy-to-dsl.json` (contexte, sans re-analyse du Legacy).  
1.4. Charger les guides de stack relatifs à l’I18N (organisation des dossiers, namespaces, conventions de clé).

### 2. Vérifier la présence et l’état de l’inventaire I18N

2.1. Charger `inventory.i18n.json`.  
2.2. Charger `inventories-summary.json` et vérifier que l’inventaire `i18n` est présent et valide pour `${project.pageName}`.  

2.3. Si l’inventaire `i18n` est manquant ou invalide :

- initialiser `mappingRoot` (voir étape 3) ;
- ajouter une issue explicite dans `mappingRoot.validation.issues` ;
- fixer `mappingRoot.validation.status = "rejected"` ;
- écrire quand même le fichier de sortie minimal ;
- retourner **Gate ❌**.

### 3. Initialiser l’objet racine `mappingRoot`

3.1. Construire en mémoire :

```jsonc
{
  "domain": "i18n",
  "pageName": "${project.pageName}",
  "sourceEntry": "${paths.legacySource}",
  "items": [],
  "validation": {
    "status": "pending",
    "issues": []
  }
}
```

3.2. Conserver cet objet en mémoire jusqu’à l’écriture du fichier de sortie.

### 4. Charger les mappings précédents pour les relations

4.1. Charger `mapping.structure.json` (obligatoire).  
4.2. Charger `mapping.layout.json` si disponible.  
4.3. Construire des index en mémoire pour relier rapidement :  
- UCR I18N → UCR structure ;
- UCR I18N → UCR layout ;
- éventuellement UCR I18N → UCR logic (si accessible).

### 5. Projeter chaque UCR `i18n.*`

5.1. Parcourir toutes les entrées de `inventory.i18n.json`.  
5.2. Pour chaque entrée, lire :
- `item.ucr` ;
- `item.dsl` (ex : `i18n.text`, `i18n.errorMessage`, `i18n.placeholder`, etc.) ;
- les métadonnées utiles (texte source, contexte d’utilisation, langue d’origine, etc.).

5.3. Déterminer `toStack.stackKind` en fonction du concept DSL et des guides de stack, par ex. :
- `translationKey` pour un texte standard ;
- `validationMessage` pour une erreur de validation ;
- `sharedMessage` pour un message réutilisable ;
- `namespaceFile` pour des UCR de haut niveau.

5.4. Construire `toStack.targetId` :  

- appliquer les conventions de nommage des clés :  
  - ex : `"<namespace>.<section>.<name>"`  
  - ou tout autre pattern défini dans les guides I18N.

5.5. Déduire `toStack.targetPath` :  

- à partir de `project-structure.json` + guides i18n :
  - par ex. un dossier `src/locales/<lang>/<namespace>.json` ;
  - ou une structure `src/i18n/<namespace>.ts`.

5.6. Déterminer `toStack.targetLayer` (`"i18n"` ou `"presentation"`).

5.7. Optionnel :  

- `toStack.targetTechnology` (ex : `i18next`, `react-intl`, etc.) ;  
- `toStack.targetPattern` (`namespacedJson`, `messagesFile`, etc.) ;  
- `toStack.hints[]` (conseils sur pluriels, formats, etc.).

5.8. Construire un `MappingItem` :

- `ucr` : identifiant unique, par ex. `map-i18n-${item.ucr}` ;  
- `fromDsl` : concept `i18n.*` ;  
- `sourceInventoryRef` :
  ```jsonc
  {
    "file": "inventory.i18n.json",
    "domain": "i18n",
    "itemUcr": "<ucr de l'inventaire>"
  }
  ```
- `relations.structureUcrs` : UCR de structure associés (si connus) ;  
- `relations.layoutUcrs` : UCR de layout associés (si connus) ;  
- `relations.logicUcrs` : UCR de logique qui manipulent le texte (si pertinents) ;  
- `relations.namespaces` : nom(s) de namespace cible (ex : `["campaignsDetail"]`) ;  
- `relations.languages` : liste de langues dans lesquelles le texte doit être projeté (ex : `["fr", "en"]`) ;  
- `metadata` : marquer `isCritical = true` et `priority = "high"` pour :
  - les titres ;
  - les CTA ;
  - les messages d’erreur bloquants ;
  - les textes légaux ou sensibles.

5.9. Ajouter chaque `MappingItem` dans `mappingRoot.items[]`.

### 6. Validation interne

6.1. Contrôles génériques :  

- `mappingRoot.domain === "i18n"` ;  
- `mappingRoot.pageName === project.pageName` ;  
- tous les `ucr` de mapping sont uniques ;  
- chaque `sourceInventoryRef.itemUcr` existe dans `inventory.i18n.json` ;  
- pour chaque item, `toStack.stackKind`, `targetId`, `targetPath`, `targetLayer` sont renseignés.

6.2. Validation de cohérence i18n (best effort) :  

- les namespaces utilisés sont cohérents avec les guides ;  
- les langues mentionnées dans `relations.languages` existent réellement dans la stack (si l’info est disponible).

6.3. Si un schéma JSON formel existe pour `mapping.i18n.json`, valider `mappingRoot` contre ce schéma.

6.4. En cas de problème bloquant :  

- ajouter une issue explicite dans `mappingRoot.validation.issues` ;  
- fixer `mappingRoot.validation.status = "rejected"`.

6.5. Sinon :  

- fixer `mappingRoot.validation.status = "valid"` ;  
- s’assurer que `mappingRoot.validation.issues` est un tableau (éventuellement vide).

### 7. Écriture de la sortie

7.1. Sérialiser `mappingRoot` vers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.i18n.json`

7.2. Créer les dossiers manquants si nécessaire.  
7.3. Ne modifier aucun autre fichier.

---

## ✅ Résumé de fin de stage (retourné par l’IA)

L’IA doit renvoyer (dans sa réponse, pas sur disque) :

```json
{
  "stageId": "33",
  "stageName": "mapping.i18n",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

- `inputsAvailable` = `false` si une entrée obligatoire est manquante.  
- `schemaValidated` = `false` si la validation JSON n’a pas été effectuée ou a échoué.  
- `outputsWritten` = `false` si le fichier de sortie n’a pas pu être écrit.

---

## 🧩 Gate

Le stage doit se terminer par **exactement l’un** des blocs :

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` en cas d’erreur bloquante (input manquant, inventaire invalide, schéma non respecté, sortie non écrite…).

---

## 📦 Stage suivant

> Continuer avec le stage suivant (par ex. `34-mapping-logic-...`) uniquement si `Gate ✅`.

---

© 2025 — ai-orchestrator-v4
