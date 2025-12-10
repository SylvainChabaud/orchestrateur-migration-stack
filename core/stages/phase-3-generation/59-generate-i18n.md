# 🧩 Stage 59 – generate-i18n
**Phase:** Phase 3 – Generation  
**Prev:** 58 – generate-routing  
**Next:** 60 – generate-tests  

---

## 🎯 Objective

Générer les **ressources d’internationalisation (i18n)** pour `${project.pageName}` dans la stack cible, en s’appuyant sur :

- les **mappings de Phase 2** (`mapping.i18n.json` principalement, mais aussi `mapping.structure.json`, `mapping.layout.json`, `mapping.routing.json`, `mapping.config.json`) ;
- la **structure cible** décrite par `project-structure.json` ;
- les **stack-guides i18n / pages / components** produits en Phase 0 ;
- le **DSL + UCR + bridge legacy → DSL** pour comprendre les UCR `i18n.*` et leur lien avec la structure de page.

Le stage reste **agnostique de la stack** : il ne suppose ni un format particulier (JSON, YAML, PO, etc.), ni une bibliothèque précise.  
Il applique les patterns décrits dans les **stack-guides d’i18n** pour produire les fichiers de ressources attendus par la stack cible.

---

## 🔌 Inputs

> Toutes les entrées sont résolues à partir de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration globale

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

### 2. Artefacts Phase 0 – Stack, structure, bridge

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `bridge-legacy-to-dsl.json`  
- `stack-guides/guide.stack.md`  
- `stack-guides/guide.i18n.md` (obligatoire)  
- `stack-guides/guide.ui-pages.md` (recommandé)  
- `stack-guides/guide.ui-components.md` (recommandé, si existant)  

Les stack-guides i18n doivent préciser au minimum :

- les **formats et emplacements** des fichiers de ressources i18n ;
- les **conventions de namespaces** (par page, par domaine, par module, etc.) ;
- la **structure de clés** (flat, nested, par section, etc.) ;
- la manière dont les pages / components consomment ces ressources.

### 3. Artefacts Phase 2 – Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.i18n.json` (obligatoire – source principale)  
- `mapping.structure.json` (pour lier les clés aux views / containers)  
- `mapping.layout.json` (regroupement des clés par zones / sections si nécessaire)  
- `mapping.routing.json` (pour les labels liés à la navigation, breadcrumbs, etc.)  
- `mapping.config.json` (modes / flags pouvant activer différentes variantes de textes)  
- `mapping.conditions.json` (textes conditionnels ou spécifiques à des segments d’utilisateurs)  
- `mapping.logic.json` (messages associés à des workflows, validations, etc.)  
- `mapping.tests.json` (peut contenir des attentes sur des messages d’erreur / succès)  
- `mappings-summary.json` (pour vérifier la readiness de Phase 2, notamment pour `i18n`).  

### 4. DSL, UCR et bridge

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`  

Utilisés pour :

- identifier tous les UCR `i18n.*` associés à `${project.pageName}` ;
- lier ces UCR à la structure de la page (rootView, sections, containers, actions, états) ;
- comprendre l’origine des textes (legacy → DSL → mapping.i18n).  

---

## 📤 Outputs

Ce stage doit produire les ressources i18n pour `${project.pageName}` sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/`

Au minimum :

1. **Un ou plusieurs fichiers de ressources i18n** dans le format attendu par la stack cible :  

   - par exemple, un fichier par locale, par namespace, ou par page, selon `guide.i18n.md` ;  
   - noms, extensions et emplacement exacts définis par `project-structure.json` + stack-guides.

2. Un fichier de **métadonnées i18n** :  

   - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/i18n.meta.json`  

   Contenant au minimum :

   ```jsonc
   {
     "pageName": "${project.pageName}",
     "namespaces": [],
     "locales": [],
     "keysCount": 0,
     "missingKeysCount": 0,
     "duplicatedKeysCount": 0,
     "ucr": {
       "i18n": [],
       "structure": [],
       "routing": []
     },
     "generatedFiles": [],
     "issues": []
   }
   ```

Règles :

- ne modifier **aucun autre répertoire** que `phase-3-generation/i18n/` pour cette page ;
- respecter les schémas éventuels `core/schemas/i18n.*.schema.json` si disponibles.

---

## 🧠 Actions

### Étape 1 – Charger configuration et contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître les conventions i18n (répertoires, structure).  
1.3. Charger `bridge-legacy-to-dsl.json` pour retrouver les UCR `i18n.*` associés à `${project.pageName}` et leur lien avec les vues / containers.  

### Étape 2 – Vérifier readiness de Phase 2

2.1. Charger `mappings-summary.json`.  
2.2. Vérifier que le domaine `i18n` est :  

- présent (`status = "present"`) ;  
- non bloquant (`validationStatus` différent de `rejected` si `requiredForPhase3`).  

2.3. Si le domaine `i18n` est manquant / invalid / rejected **et** `requiredForPhase3` :

- consigner le problème dans `issues` ;  
- produire un `i18n.meta.json` minimal ;  
- arrêter le stage avec **Gate ❌**.

### Étape 3 – Charger et analyser `mapping.i18n.json`

3.1. Charger `mapping.i18n.json`.  
3.2. Pour chaque item, extraire :  

- le ou les UCR d’origine (`sourceInventoryRef.itemUcr`) ;  
- le type de texte (`label`, `title`, `placeholder`, `errorMessage`, `helperText`, `tooltip`, etc.) ;  
- la clé cible et éventuellement le namespace (`toStack.key`, `toStack.namespace`, etc.) ;  
- les éventuelles infos de locale (si déjà définies dans le mapping).  

3.3. Construire une collection en mémoire de **descripteurs de clés** de la forme :

```jsonc
{
  "ucr": "i18n.label.MyPage.submit-1",
  "type": "label",
  "namespace": "pages.${project.pageName}",
  "key": "submit",
  "defaultValue": "",
  "locales": {}
}
```

Les détails (structure exacte, champs) peuvent être adaptés en fonction du schéma JSON prévu pour les ressources i18n.

### Étape 4 – Enrichir avec la structure et le routing

4.1. À partir de `mapping.structure.json` et `bridge-legacy-to-dsl.json` :  

- lier les clés à des vues / containers / sections ;  
- détecter les clés orphelines (sans structure associée).  

4.2. À partir de `mapping.routing.json` :  

- détecter les clés de navigation (breadcrumbs, titres de routes, liens, boutons de navigation).  

4.3. À partir de `mapping.config.json` et `mapping.conditions.json` :  

- identifier les clés conditionnelles (ex. textes affichés uniquement dans certains modes ou pour certains profils).  

Cette étape permet :  

- de vérifier la cohérence globale (toutes les vues clés ont un titre, etc.) ;  
- de mieux organiser les namespaces si la stack le permet (par section, par layout…).  

### Étape 5 – Appliquer les conventions des stack-guides i18n

5.1. Charger `stack-guides/guide.i18n.md`.  
5.2. Déterminer :  

- comment regrouper les clés (par page, par module, par domaine…) ;  
- quels namespaces utiliser (ex. `pages.${project.pageName}`, `layout`, `common`, etc.) ;  
- la structure des fichiers :  
  - un fichier par locale + namespace ;  
  - ou un fichier global par locale avec sous-clés ;  
  - ou autre pattern défini par le stack-guide.  

5.3. Organiser la collection de descripteurs de clés en **resources** prêtes à être sérialisées dans les formats définis par la stack.  

### Étape 6 – Générer les fichiers de ressources i18n

6.1. Pour chaque combinaison (locale, namespace, ou autre partition définie par le stack-guide) :  

- créer un objet de ressources (ex. arbre de clés → valeurs) ;  
- y injecter toutes les clés correspondantes.  

6.2. Si le mapping fournit déjà des valeurs par défaut (ex. extraites du legacy), les utiliser comme `defaultValue` :  

- sinon, laisser des placeholders vides ou des valeurs explicites (ex. `"__MISSING__"`) si le stack-guide le recommande.  

6.3. Sérialiser ces ressources dans les fichiers décrits par :  

- `project-structure.json` ;  
- `guide.i18n.md`.  

6.4. Écrire les fichiers sous :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/`

En respectant les conventions (extensions, nommage, contenu) décrites par les stack-guides.

### Étape 7 – Construire `i18n.meta.json`

7.1. Construire un objet `i18nMeta` contenant au minimum :  

- `pageName` ;  
- `namespaces` utilisés ;  
- `locales` générées (si la stack gère plusieurs locales à ce stade) ;  
- `keysCount` (total de clés dans les resources) ;  
- `missingKeysCount` (clés requises selon DSL/UCR mais non mappées) ;  
- `duplicatedKeysCount` (mêmes clé + namespace définies plusieurs fois) ;  
- UCR utilisés :
  - `ucr.i18n` ;
  - `ucr.structure` ;
  - `ucr.routing` ;
- `generatedFiles` (liste des fichiers de ressources écrits) ;  
- `issues` (warnings, erreurs non bloquantes).  

7.2. Écrire le fichier :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/i18n.meta.json`

### Étape 8 – Validation et Gate

8.1. Conditions typiques pour **Gate ❌** :  

- `mapping.i18n.json` manquant ou illisible ;  
- aucun fichier de ressource i18n n’a pu être écrit alors que des UCR `i18n.*` existent ;  
- erreurs critiques lors de la sérialisation selon les schémas éventuels.  

8.2. Conditions pour **Gate ✅** :  

- au moins un fichier de ressource i18n généré avec des clés ;  
- `i18n.meta.json` correctement écrit ;  
- pas d’erreur bloquante dans les issues.  

---

## 🧩 Gate

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` en cas de problème bloquant (inputs indispensables manquants, sortie non écrite, JSON invalide…).

---

## 📦 Next

> Continuer avec `60-generate-tests.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
