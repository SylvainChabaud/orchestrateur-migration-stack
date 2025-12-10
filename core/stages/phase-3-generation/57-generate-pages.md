# 🧩 Stage 57 – generate-pages
**Phase:** Phase 3 – Generation  
**Prev:** 56 – generate-components-containers  
**Next:** 58 – generate-routing  

---

## 🎯 Objective

Générer les **artefacts de pages** pour `${project.pageName}` dans la stack cible, en orchestrant :

- les **components-containers** et **components-atoms** générés par les stages 55–56,
- les **hooks-data** et **hooks-logic** (stages 53–54),
- les **stores** (stage 52) et **services** (stage 51),
- les **mappings de Phase 2** (structure, layout, routing, i18n, logic, actions, conditions, effects…),
- les **stack-guides de pages / layout / i18n / routing** produits en Phase 0,
- le **DSL + UCR + bridge legacy → DSL** comme source de vérité sémantique.

Le stage reste **agnostique de la stack** : il n’impose ni React, ni Vue, ni Angular.  
Il applique uniquement les patterns décrits dans les **stack-guides** pour générer les pages finales.

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

### 2. Artefacts Phase 0 – Stack / structure / bridge

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `bridge-legacy-to-dsl.json`  
- `stack-guides/guide.stack.md`  
- `stack-guides/guide.ui-pages.md` (obligatoire)  
- `stack-guides/guide.layout.md` (recommandé)  
- `stack-guides/guide.routing.md` (recommandé)  
- `stack-guides/guide.i18n.md` (recommandé)  

Les stack-guides définissent :

- la forme abstraite d’une **page** dans la stack cible (nommage, emplacement, API),
- la manière de **composer** containers, hooks, services,
- comment **appliquer un layout** et **brancher l’i18n**,
- comment lier la page au **routing** (sans encore générer la config de routing, qui est du ressort du stage 58).

### 3. Artefacts Phase 2 – Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json` (obligatoire – rootView, sections, containers)  
- `mapping.layout.json` (obligatoire – zones, templates de page)  
- `mapping.routing.json` (recommandé – infos de route associées à la page)  
- `mapping.i18n.json` (obligatoire – titres, labels, messages pour la page)  
- `mapping.logic.json` (logique métier de vue / workflow)  
- `mapping.hooks.json` (hooks transverses de vue)  
- `mapping.dataflows.json` (flux de données principaux de la page)  
- `mapping.services.json` (services consommés sur la page)  
- `mapping.actions.json` (actions métiers exposées sur la page)  
- `mapping.conditions.json` (conditions d’affichage / habilitations)  
- `mapping.effects.json` (toasts, logs, tracking)  
- `mapping.config.json` (feature flags / modes)  
- `mapping.tests.json` (stratégie de tests de la page – non bloquant)  
- `mappings-summary.json` (pour vérifier la readiness globale de Phase 2)  

### 4. Artefacts Phase 3 déjà générés (services / stores / hooks / components)

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

- `services/` (stage 51)  
- `stores/` (stage 52)  
- `hooks-logic/` (stage 53)  
- `hooks-data/` (stage 54)  
- `components/atoms/` (stage 55)  
- `components/containers/` (stage 56)  

Les chemins exacts et patterns de fichiers sont définis par les stack-guides et `project-structure.json`.

### 5. DSL, UCR et bridge

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`  

Ils servent à :

- identifier la **rootView UCR** `structure.rootView.*` associée à `${project.pageName}`,
- comprendre les UCR `logic.viewLifecycle.*`, `logic.workflow.*` liés à la vue,
- interpréter les différents UCR des domaines `routing`, `i18n`, `effects`, etc.

---

## 📤 Outputs

Ce stage doit produire **exclusivement** des artefacts de page pour `${project.pageName}` sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/`

Outputs minimum :

1. **Un ou plusieurs fichiers de page exécutables** dans la stack cible :  

   - noms, extensions et emplacements exacts définis par `guide.ui-pages.md` et `project-structure.json`.  
   - Exemples conceptuels (ne pas coder en dur) :  
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/<stackPageMainFile>`  
     - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/<stackPageVariants...>`  

2. Un fichier de **métadonnées de page** :  

   - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/pages.meta.json`  

   Contenant au minimum :

   ```jsonc
   {
     "pageName": "${project.pageName}",
     "rootViewUcr": "",
     "containersCount": 0,
     "hasLayout": false,
     "hasRoutingInfo": false,
     "hooksDataUsed": [],
     "hooksLogicUsed": [],
     "storesUsed": [],
     "actionsUsed": [],
     "effectsUsed": [],
     "ucr": {
       "structure": [],
       "layout": [],
       "routing": [],
       "i18n": [],
       "logic": [],
       "actions": []
     },
     "generatedFiles": [],
     "issues": []
   }
   ```

Règles :

- ne modifier **aucun autre dossier** que `phase-3-generation/pages/` pour cette page ;
- respecter les schémas éventuels `core/schemas/pages.*.schema.json` si disponibles.

---

## 🧠 Actions

### Étape 1 – Charger la configuration et le contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre tous les `${paths.*}`.  
1.2. Charger `project-structure.json` pour connaître les dossiers / patterns de pages.  
1.3. Charger `bridge-legacy-to-dsl.json` pour retrouver la **rootView UCR** de `${project.pageName}`.  

### Étape 2 – Vérifier la readiness de Phase 2

2.1. Charger `mappings-summary.json`.  
2.2. Vérifier que les domaines suivants sont présents et non bloquants :  

- `structure`,  
- `layout`,  
- `i18n`,  
- `logic` (si la page a des comportements dynamiques),  
- `actions` (si la page déclenche des use-cases).  

2.3. Si `structure` ou `layout` est manquant / invalid / rejected et `requiredForPhase3` :

- consigner l’erreur dans `issues` ;
- produire un `pages.meta.json` minimal ;
- arrêter le stage avec **Gate ❌**.

### Étape 3 – Charger les mappings nécessaires

3.1. Charger `mapping.structure.json` et extraire :  

- la vue racine (`rootView`) pour `${project.pageName}` ;  
- les containers associés ;  
- les sections / zones pertinentes.  

3.2. Charger `mapping.layout.json` pour savoir :  

- quel layout (global ou spécifique) s’applique à la page ;  
- comment les containers sont répartis dans les zones.  

3.3. Charger `mapping.routing.json` (si présent) pour :  

- identifier le path principal de la page ;  
- savoir si des params sont associés (utile mais le stage 58 reste maître du routing).  

3.4. Charger `mapping.i18n.json` pour :  

- récupérer les clés et namespaces associées à la page (titre, sous-titres, labels principaux).  

3.5. Charger `mapping.logic.json`, `mapping.actions.json`, `mapping.hooks.json`, `mapping.dataflows.json`, `mapping.effects.json`, `mapping.config.json` pour :  

- identifier les hooks, stores, services, actions et effets à brancher dans la page ;  
- connaître les conditions d’affichage ou de variation de la page (modes, flags).  

### Étape 4 – Lire et interpréter les stack-guides de pages

4.1. Charger `stack-guides/guide.ui-pages.md`.  
4.2. Si disponibles, charger aussi :  

- `stack-guides/guide.layout.md` ;  
- `stack-guides/guide.i18n.md` ;  
- `stack-guides/guide.routing.md`.  

4.3. En extraire :  

- les **patterns de page** (page simple, paramétrée, layout complexe, wizard, etc.) ;  
- les signatures attendues pour les pages (nom, type, contract de props, etc.) ;  
- les conventions pour brancher :  
  - layout,  
  - i18n,  
  - hooks-data,  
  - hooks-logic,  
  - stores,  
  - services,  
  - effets.  

Le stage ne doit **jamais inventer** ces conventions : il doit suivre strictement les stack-guides.

### Étape 5 – Construire un modèle de page en mémoire

5.1. À partir de `mapping.structure` + `bridge-legacy-to-dsl` + UCR :  

- identifier le type de page (liste, détail, formulaire, composite, wizard…) ;  
- identifier les containers racines et secondaires ;  
- lier chaque container aux sections / zones.  

5.2. Enrichir ce modèle avec :  

- layout (d’après `mapping.layout`) ;  
- i18n (d’après `mapping.i18n`) ;  
- routing (d’après `mapping.routing`, juste comme métadonnée) ;  
- hooks (d’après `mapping.hooks`, `mapping.dataflows`) ;  
- actions / effets (d’après `mapping.actions`, `mapping.effects`).  

5.3. Obtenir une représentation abstraite de la page :

```jsonc
{
  "pageId": "${project.pageName}",
  "rootViewUcr": "structure.rootView.MyPage-1",
  "layout": { "id": "...", "zones": [] },
  "containers": [],
  "hooksData": [],
  "hooksLogic": [],
  "stores": [],
  "actions": [],
  "effects": [],
  "i18nKeys": [],
  "routingHints": {}
}
```

Cette structure reste interne au stage (sauf si un schéma d’AST de page est prévu plus tard).

### Étape 6 – Générer les fichiers de page concrets

6.1. Choisir le **pattern de page** adapté en fonction de :  

- rootView UCR ;  
- layout ;  
- présence de params de route ;  
- type d’interaction (simple, wizard, multi-sections).  

6.2. À partir des stack-guides :  

- déterminer le **nom de fichier**, le **chemin**, et la **forme** de la page ;  
- injecter la composition des containers (issus de `phase-3-generation/components/containers/`) ;  
- brancher les hooks-data, hooks-logic, stores, services et effets ;  
- brancher l’i18n (titre, labels principaux), sans laisser de chaînes en dur si des clés existent.  

6.3. Écrire les fichiers générés dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/`  
- en respectant la structure définie par `project-structure.json` + stack-guides.

### Étape 7 – Générer `pages.meta.json`

7.1. Construire `pagesMeta` avec :  

- `pageName` ;  
- `rootViewUcr` ;  
- `containersCount` ;  
- `hasLayout`, `hasRoutingInfo` ;  
- listes des hooks, stores, actions, effets utilisés ;  
- UCR par domaine ;  
- liste des fichiers générés ;  
- éventuels `issues[]` (warnings et erreurs non bloquantes).  

7.2. Écrire :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/pages.meta.json`

### Étape 8 – Validation et Gate

8.1. Gate ❌ si :  

- rootView UCR introuvable ;  
- `mapping.structure.json` ou `mapping.layout.json` manquant / invalide ;  
- aucun fichier de page n’a pu être généré malgré la présence d’une rootView déclarée.  

8.2. Gate ✅ si :  

- au moins une page a été générée ;  
- `pages.meta.json` est écrit ;  
- les dépendances critiques sont satisfaites.

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

Utiliser `Gate ❌` en cas de problème bloquant (invalid mappings, inputs indispensables manquants, sortie non écrite).

---

## 📦 Next

> Continuer avec `58-generate-routing.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
