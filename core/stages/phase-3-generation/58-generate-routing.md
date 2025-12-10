# 🧩 Stage 58 – generate-routing
**Phase:** Phase 3 – Generation  
**Prev:** 57 – generate-pages  
**Next:** 59 – generate-i18n

---

## 🎯 Objective

Générer la **configuration de routing** pour `${project.pageName}` dans la stack cible, en s’appuyant uniquement sur :

- les **mappings de Phase 2** (principalement `mapping.routing.json`, `mapping.structure.json`, `mapping.layout.json`, `mapping.config.json`, `mapping.conditions.json`) ;
- la **structure cible** décrite par `project-structure.json` ;
- les **stack-guides de routing / pages / layout** générés en Phase 0 ;
- le **DSL + UCR + bridge legacy → DSL** pour interpréter correctement les UCR `routing.*`.

Le stage reste **agnostique de la stack** : il applique les patterns décrits dans les stack-guides sans faire d’hypothèse sur le framework (file-based router, config-based router, etc.).

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

### 2. Artefacts Phase 0 (stack / structure / bridge)

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `bridge-legacy-to-dsl.json`  
- `stack-guides/guide.stack.md`  
- `stack-guides/guide.routing.md` (obligatoire)  
- `stack-guides/guide.ui-pages.md` (recommandé)  
- `stack-guides/guide.layout.md` (recommandé, si le routing dépend du layout)  

Ces guides définissent :

- les **patterns de routes** (simples, imbriquées, paramétrées, redirections, guards, lazy, etc.) ;
- la manière de **lier les routes aux artefacts de page** générés par le stage 57 ;
- les **conventions de fichiers / dossiers** pour la configuration de routing dans la stack cible.

### 3. Artefacts Phase 2 – Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.routing.json` (obligatoire)  
- `mapping.structure.json` (obligatoire pour lier routes ↔ pages / rootViews)  
- `mapping.layout.json` (optionnel mais recommandé)  
- `mapping.config.json` (feature flags liés au routing, modes, variantes)  
- `mapping.conditions.json` (guards de route, conditions d’accès)  
- `mapping.logic.json` (facultatif : logique de navigation avancée)  
- `mappings-summary.json` (obligatoire pour vérifier la readiness globale de la Phase 2)  

### 4. DSL, UCR et bridge

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`  

Ils servent à :

- comprendre les **types d’UCR** `routing.*` (route, param, guard, group, fallback, etc.) ;
- lier les routes à la **rootView** de `${project.pageName}` ;
- interpréter correctement les patterns detectés depuis le legacy.

---

## 📤 Outputs

Ce stage doit produire **exclusivement** des artefacts de routing pour `${project.pageName}` sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/`

Au minimum :

1. **Configuration(s) de routing exécutable(s)** dans la stack cible :  

   - un ou plusieurs fichiers dont les noms, extensions et emplacements exacts sont décrits par `guide.routing.md` et/ou `project-structure.json`.  
   - Exemple conceptuel (ne pas coder en dur) :  
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/<stackRoutingMainFile>`

2. Un fichier de **métadonnées** :  

   - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/routing.meta.json`  

   Contenant au minimum :  

   ```jsonc
   {
     "pageName": "${project.pageName}",
     "routesCount": 0,
     "routes": [],
     "hasGuards": false,
     "hasRedirects": false,
     "hasNestedRoutes": false,
     "usesLayoutBinding": false,
     "ucr": {
       "routing": [],
       "structure": []
     },
     "inputs": {
       "mappingRouting": "mapping.routing.json",
       "mappingStructure": "mapping.structure.json"
     },
     "issues": []
   }
   ```

Règles :

- Aucun autre fichier en dehors de `${paths.workspace}` ne doit être modifié.  
- Ne pas toucher aux phases précédentes.  
- Respecter les schémas éventuels définis dans `core/schemas/routing.*.schema.json` si disponibles.

---

## 🧠 Actions

> Décrire ici la séquence exacte d’actions que l’IA doit réaliser pour exécuter ce stage.

### Étape 1 – Charger la configuration et le contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre tous les `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître :  
- les répertoires et patterns prévus pour le routing (ex. `routingRootDir`, `routingEntryFile`, etc.).  
1.3. Charger `bridge-legacy-to-dsl.json` pour retrouver :  
- la rootView UCR associée à `${project.pageName}` ;  
- les UCR `routing.*` pertinents pour cette page.  

### Étape 2 – Vérifier la readiness de la Phase 2

2.1. Charger `mappings-summary.json`.  
2.2. Vérifier que :  

- le domaine `routing` est en `status = "present"` et `validationStatus` non bloquant ;  
- le domaine `structure` est également présent et valide (sinon, les routes n’auraient pas de cible).  

2.3. Si `routing` ou `structure` est manquant / invalid / rejected **et** marqué comme `requiredForPhase3` :  

- ajouter une entrée dans `issues` (qui sera recopiée dans `routing.meta.json`) ;  
- fixer le **Gate** du stage à ❌ ;  
- ne générer que, éventuellement, un `routing.meta.json` minimal décrivant l’échec.  

### Étape 3 – Charger les mappings nécessaires

3.1. Charger `mapping.routing.json` (obligatoire).  
3.2. Charger `mapping.structure.json` (pour lier routes ↔ pages / rootViews).  
3.3. Charger :  

- `mapping.layout.json` (liaison route ↔ layout, si applicable),  
- `mapping.config.json` (feature flags influençant le routing),  
- `mapping.conditions.json` (guards, habilitations),  
- `mapping.logic.json` (logique avancée de navigation si décrite).  

3.4. Construire en mémoire des index simples :  

- UCR → route (depuis `mapping.routing`) ;  
- route → page / rootView (via `mapping.structure`) ;  
- route → conditions / guards (via `mapping.conditions` et `mapping.config`).  

### Étape 4 – Lire les stack-guides de routing

4.1. Charger `stack-guides/guide.routing.md`.  
4.2. Si disponible, charger également :  

- `stack-guides/guide.ui-pages.md`  
- `stack-guides/guide.layout.md`  

4.3. En extraire :  

- les **patterns de routes** (simples, imbriquées, paramétrées, redirections, guards, lazy…) ;  
- la **forme attendue des fichiers** (config unique, arbre de fichiers, segments, etc.) ;  
- les **règles de nommage** (ex. `PageRoute`, `routes.<something>`, etc.).  

> Le stage ne doit pas inventer ces détails : il doit suivre strictement ce que décrivent les stack-guides.

### Étape 5 – Construire un AST de routing en mémoire

5.1. Pour chaque entrée de `mapping.routing.items[]` :  

- lire l’UCR d’origine (`sourceInventoryRef.itemUcr`) ;  
- lire le `toStack.*` associé (path, params, type de route, guards, redirections, meta).  

5.2. Enrichir chaque route avec :  

- la page / rootView cible (via `mapping.structure`) ;  
- le layout éventuellement associé (via `mapping.layout`) ;  
- les conditions / guards (via `mapping.conditions`, `mapping.config`) ;  
- les informations de navigation / logique (via `mapping.logic`, si présent).  

5.3. Construire en mémoire une structure d’AST de routing hiérarchique :  

- routes racines ;  
- routes enfants ;  
- segments dynamiques (params) ;  
- redirections ;  
- fallback / 404 / catch-all.  

Cette structure en mémoire sert ensuite à la génération du code concret. Elle n’est pas forcément écrite telle quelle dans un fichier dédié (sauf si les stack-guides le demandent).

### Étape 6 – Générer la configuration de routing concrète

6.1. Pour chaque pattern de route de l’AST, choisir le **pattern stack adapté** selon `guide.routing.md` :  

- route simple ;  
- nested route ;  
- route paramétrée ;  
- route avec guard ;  
- route avec lazy loading, etc.  

6.2. Déterminer le ou les fichiers cibles, à partir de :  

- `project-structure.json` (chemins de base),  
- les conventions de `guide.routing.md` (dossiers, noms, extensions).  

6.3. Générer le contenu du fichier de routing **dans la syntaxe de la stack cible** en :  

- injectant les chemins / identificants de pages fournis par le stage 57 ;  
- reliant chaque route à sa page / rootView ;  
- appliquant les guards / conditions / redirections décrits dans les mappings.  

6.4. Ecrire les fichiers de routing dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/`  
- en respectant strictement la structure décrite dans `project-structure.json` + stack-guides.

### Étape 7 – Générer `routing.meta.json`

7.1. Construire un objet `routingMeta` contenant au minimum :  

- `pageName` ;  
- `routesCount` ;  
- une liste synthétique des routes (path, cible, hasGuard, hasRedirect, isNested, etc.) ;  
- les UCR `routing.*` et `structure.*` utilisés ;  
- la liste des fichiers générés ;  
- un tableau `issues[]` (warnings + erreurs non bloquantes).  

7.2. Sérialiser `routingMeta` dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/routing.meta.json`

### Étape 8 – Validation et Gate

8.1. Si `mapping.routing.json` est manquant ou illisible → **Gate ❌**.  
8.2. Si aucun fichier de routing n’a pu être généré alors que des routes étaient attendues → **Gate ❌**.  
8.3. Sinon, si au moins une route valide a été générée, que les fichiers cibles existent et que `routing.meta.json` a été écrit : → **Gate ✅**.

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

Utiliser `Gate ❌` en cas de problème bloquant (inputs manquants, impossibilité d’écrire les sorties minimales, parsing JSON invalide sur `mapping.routing.json`, etc.).

---

## 📦 Next

> Continuer avec `59-generate-i18n.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
