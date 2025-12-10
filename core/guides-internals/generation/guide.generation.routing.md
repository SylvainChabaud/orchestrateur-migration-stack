# 🔧 Guide Génération — Routing

*(Domaine de génération : **routing** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de génération

Le domaine **routing** décrit comment générer la **configuration de navigation** de `${project.pageName}` dans la stack cible, à partir :

1. des **mappings de routing** (`mapping.routing.json`) qui définissent les routes, paramètres, guards, redirections, etc. ;
2. de la **structure cible** (`project-structure.json`) qui indique où vivent les fichiers de routing ;
3. des **stack-guides de routing / pages / layout** (`guide.routing.md`, `guide.ui-pages.md`, `guide.layout.md`) qui décrivent les patterns concrets de la stack ;
4. du **DSL + UCR + bridge legacy → DSL**, qui donnent le sens sémantique des UCR `routing.*` et leur lien avec les vues/pages.

Ce guide reste **agnostique de la stack** : il décrit les *concepts* de routing (routes, segments, guards, nested routes) mais délègue la syntaxe exacte et les primitives de router aux **stack-guides**.

Objectif final : produire une ou plusieurs **configurations de routing exécutables** pour `${project.pageName}`, plus un fichier `routing.meta.json` qui donne une vue synthétique des routes générées.

---

## 2. 🔌 Entrées de génération

### 2.1. Configuration & chemins

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `stack.custom`

Les chemins concrets vers les fichiers générés **ne sont jamais codés en dur** :  
ils dérivent de `${paths.*}`, de `project-structure.json` et des stack-guides.

### 2.2. Artefacts Phase 0 — Stack & structure

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `bridge-legacy-to-dsl.json`
- `stack-guides/guide.stack.md`
- `stack-guides/guide.routing.md` (obligatoire)
- `stack-guides/guide.ui-pages.md` (recommandé)
- `stack-guides/guide.layout.md` (recommandé)

Les stack-guides de routing doivent préciser au minimum :

- la **forme des fichiers** (un fichier unique, plusieurs fichiers, file-based routing, etc.) ;
- les **patterns supportés** :  
  - route simple ;
  - route paramétrée ;
  - nested route ;
  - redirects ;
  - guards ;
  - fallback / 404 ;
  - lazy loading (si applicable) ;
- les **conventions de nommage** des routes et des fichiers.

### 2.3. Artefacts Phase 2 — Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.routing.json` (domaine principal) ;
- `mapping.structure.json` (lien route ↔ page / rootView) ;
- `mapping.layout.json` (association layout ↔ route, si la stack le prévoit) ;
- `mapping.conditions.json` (guards / habilitations / préconditions d’accès) ;
- `mapping.config.json` (feature flags ou modes impactant les routes) ;
- `mapping.logic.json` (logique de navigation, transitions entre vues) ;
- `mappings-summary.json` (readiness globale de Phase 2).

### 2.4. DSL, UCR, bridge

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`  

Utilisés pour :

- connaître les **types d’UCR routing** :  
  - `routing.route.*`,  
  - `routing.routeParam.*`,  
  - `routing.guard.*`,  
  - `routing.redirect.*`,  
  - `routing.fallback.*`, etc. ;
- retrouver la **rootView** de `${project.pageName}` ;
- comprendre le lien entre legacy et routes DSL avant mapping.

---

## 3. 🧠 Règles générales de génération de routing

### 3.1. Une route = un UCR `routing.route.*` enrichi

Chaque route générée doit avoir une **origine claire** dans le DSL :

- UCR principal : `routing.route.*` ;
- UCR secondaires :  
  - `routing.routeParam.*` (segments dynamiques),  
  - `routing.guard.*` (conditions d’accès),  
  - `routing.redirect.*`,  
  - `routing.fallback.*`.

Cette traçabilité doit être conservée dans `routing.meta.json` et, si la stack le permet, dans des commentaires ou métadonnées du fichier de routing.

### 3.2. Routes ↔ Pages

Toute route doit pointer vers :

- une **page générée** par le stage 57 (`generate-pages`),  
- ou vers une autre ressource explicitement prévue par le DSL (par ex. une page externe ou un module différent, selon les mappings).

Le mapping `structure` fournit le lien formel :

- `structure.rootView.*` ↔ `${project.pageName}` ↔ route(s) principales ;
- `structure.section.*` ↔ sous-routes éventuelles / nested routes.

### 3.3. Usage strict des stack-guides

Ce guide ne spécifie **pas** :

- les noms de fonctions de routing ;
- la syntaxe des routes ;
- les objets stack-spécifiques.

Ces éléments sont fournis par :

- `guide.routing.md` (patterns & API router) ;
- `guide.ui-pages.md` (comment référencer une page) ;
- `guide.layout.md` (comment binder un layout à une route).

Le générateur doit **lire** ces guides et appliquer fidèlement leurs conventions.

### 3.4. Pas de relecture du legacy

Les décisions de routing se basent **exclusivement** sur :

- DSL + UCR + bridge ;
- mappings Phase 2 ;
- stack-guides ;
- structure projet.

Le legacy (`${paths.legacySource}`) ne doit jamais être relu à ce stade.

---

## 4. 🧬 Patterns de routing à supporter (au niveau conceptuel)

### 4.1. Route simple

Cas le plus basique :  
- `routing.route.list-1` → path `/campaigns` → page `CampaignsList`.

Règles :

- path littéral ;  
- pas de params ;  
- pas de guard ;  
- pas de nested children.

### 4.2. Route paramétrée

UCR typiques :

- `routing.route.detail-1` ;
- `routing.routeParam.detail-id-1` (paramètre `:id`).

Règles :

- la route doit inclure le segment dynamique ;
- le paramètre doit être transmis à la page/la logique métier via les mécanismes prévus par la stack (abstraction décrite dans `guide.routing.md`) ;
- les cas param manquant / invalide doivent être gérés (fallback / redirection).

### 4.3. Nested routes / children

UCR typiques :

- `routing.route.parent-1` ;
- `routing.route.child-1` avec parent indiqué.

Règles :

- reproduire la hiérarchie décrite dans `mapping.routing.json` ;
- si la stack prévoit un composant ou un wrapper spécial pour les nested routes, suivre `guide.routing.md`.

### 4.4. Guards

UCR : `routing.guard.*` + `conditions.*` + `config.*`.

Règles :

- générer des **hooks / fonctions / objets de guard** selon les stack-guides ;
- relier ces guards à la route correspondante (ex. champs `canActivate`, `beforeEnter`, etc. définis dans `guide.routing.md`) ;
- ne pas implémenter ici la logique métier de guard :  
  - elle doit être déléguée à des services/hooks générés par d’autres stages (logic, services, hooks-logic).

### 4.5. Redirects

UCR : `routing.redirect.*`.

Règles :

- générer les redirections conformément aux capacités du router de la stack ;
- ne pas coder des redirects « à la main » (ex. `window.location`) ;
- toujours privilégier les primitives du router décrites dans `guide.routing.md`.

### 4.6. Fallback / 404

UCR : `routing.fallback.*`.

Règles :

- déclarer la route de fallback / 404 selon les patterns de la stack ;
- veiller à ce qu’elle n’interfère pas avec les routes existantes (ordre, priorités, catch-all, etc.).

---

## 5. 🗂 Structure des fichiers de routing générés

Les fichiers de routing doivent être créés sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/`

La structure exacte est déterminée par :

- `project-structure.json`  
- `stack-guides/guide.routing.md`  

Exemples conceptuels (à adapter via les guides, ne pas coder en dur) :

- fichier unique :  
  - `routing/mainRoutingFile` (nom, extension et path décrits par `guide.routing.md`) ;
- plusieurs fichiers :  
  - `routing/routes.<segment>.ext` par groupe de routes,  
  - `routing/nested/<feature>.ext`, etc.

Le guide de routing doit spécifier :

- comment **exporter** la config (nom de la constante, default export, etc.) ;
- comment **composer** ces fichiers avec le router racine de l’application ;
- comment **brancher les pages** générées par le stage 57.

---

## 6. 📝 `routing.meta.json`

En plus des fichiers de config de routing, le stage doit produire :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/routing/routing.meta.json`

Ce fichier contient au minimum :

```jsonc
{
  "pageName": "${project.pageName}",
  "routesCount": 3,
  "routes": [
    {
      "ucr": "routing.route.list-1",
      "path": "/campaigns",
      "targetPageId": "CampaignsListPage",
      "hasParams": false,
      "hasGuard": false,
      "hasRedirect": false,
      "isNested": false
    }
  ],
  "hasGuards": true,
  "hasRedirects": false,
  "hasNestedRoutes": true,
  "usesLayoutBinding": true,
  "ucr": {
    "routing": ["routing.route.list-1", "routing.route.detail-1"],
    "structure": ["structure.rootView.CampaignsList-1"]
  },
  "inputs": {
    "mappingRouting": "mapping.routing.json",
    "mappingStructure": "mapping.structure.json"
  },
  "issues": []
}
```

Ce fichier est utilisé par :

- les outils de diagnostic ;  
- les étapes ultérieures (par exemple, en Phase 4, pour valider la cohérence navigation / tests).

---

## 7. ✅ Checklist de génération pour `routing`

Avant de considérer que la génération de routing pour `${project.pageName}` est complète :

- [ ] `mapping.routing.json` est présent, lisible et validé via `mappings-summary.json`  
- [ ] `mapping.structure.json` permet de lier chaque route à une page / rootView  
- [ ] `guide.routing.md` est disponible et pris en compte (patterns, structure fichiers)  
- [ ] Au moins une route est générée si des UCR `routing.route.*` existent  
- [ ] Les paramètres de route sont correctement reliés à la logique / dataflows des pages  
- [ ] Les guards sont déclarés en s’appuyant sur des hooks / services, pas en logique ad-hoc dans les fichiers de routing  
- [ ] Les redirects sont déclarés via les primitives du router  
- [ ] Une route de fallback / 404 est présente si définie dans le DSL  
- [ ] Tous les fichiers de routing sont écrits sous `phase-3-generation/routing/` selon la structure décrite par les stack-guides  
- [ ] `routing.meta.json` a été généré et liste correctement les routes, UCR et éventuels issues

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
