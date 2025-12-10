# 🔧 Guide Génération — Pages

*(Domaine de génération : **pages** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de génération

Le domaine **pages** décrit comment générer les **vues racines** de `${project.pageName}` dans la stack cible.

Une *page* est l’unité de composition la plus haute pour une fonctionnalité UI :

- elle orchestre les **containers** et **atoms** ;
- consomme les **hooks de données** et de **logique métier** ;
- applique un **layout** ;
- se connecte au **routing** (mais la config router est gérée par le stage 58) ;
- branche l’**internationalisation** (i18n) ;
- peut déclencher des **actions** et des **effects** (toasts, tracking, logs, etc.).

Ce guide reste **agnostique de la stack** : il ne parle pas de composants React, de templates Vue, de modules Angular, etc.  
Les détails concrets sont décrits par les **stack-guides** (`guide.ui-pages.md`, `guide.layout.md`, `guide.i18n.md`, `guide.routing.md`).

---

## 2. 🔌 Entrées du domaine de génération

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

Aucun chemin absolu ne doit être codé en dur.  
Les chemins des pages générées dérivent de `${paths.*}` + `project-structure.json` + stack-guides.

### 2.2. Artefacts Phase 0 — Stack / Structure / Bridge

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `bridge-legacy-to-dsl.json`
- `stack-guides/guide.stack.md`
- `stack-guides/guide.ui-pages.md`
- `stack-guides/guide.layout.md` (si le layout est géré par la stack)
- `stack-guides/guide.routing.md` (pour les liens route ↔ page)
- `stack-guides/guide.i18n.md` (pour l’internationalisation)

Le guide de pages (`guide.ui-pages.md`) doit préciser :

- où vivent les pages (répertoires, patterns de fichiers) ;
- comment elles se déclarent dans la stack (export, signature, props/context) ;
- comment elles référencent les containers, hooks, stores, services ;
- comment elles intègrent layout / i18n / routing.

### 2.3. Artefacts Phase 2 — Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.structure.json` (rootView, sections, containers) ;
- `mapping.layout.json` (zones, templates de page) ;
- `mapping.routing.json` (routes associées à la page) ;
- `mapping.i18n.json` (titres, labels, messages de la page) ;
- `mapping.logic.json` (logique de vue, workflows) ;
- `mapping.hooks.json` (hooks transverses côté UI) ;
- `mapping.dataflows.json` (flux de données gérés) ;
- `mapping.services.json` (services consommés) ;
- `mapping.actions.json` (actions métiers déclenchées) ;
- `mapping.effects.json` (toasts, logs, analytics) ;
- `mapping.conditions.json` (conditions d’affichage / habilitations) ;
- `mapping.config.json` (modes, feature flags) ;
- `mapping.tests.json` (stratégie de tests liée à la page) ;
- `mappings-summary.json` (readiness globale de la Phase 2).

### 2.4. Artefacts Phase 3 — Prérequis

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

- `services/` (stage 51) ;
- `stores/` (stage 52) ;
- `hooks-logic/` (stage 53) ;
- `hooks-data/` (stage 54) ;
- `components/atoms/` (stage 55) ;
- `components/containers/` (stage 56).

Les pages **ne doivent pas** réimplémenter ces briques ; elles doivent seulement les **orchestrer**.

### 2.5. DSL + UCR + bridge

- `Spec Dsl Orchestrator`
- `Spec Ucr Orchestrator`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Ils indiquent, entre autres :

- les UCR `structure.rootView.*` (rootView de la page) ;
- les UCR `logic.viewLifecycle.*` et `logic.workflow.*` (comportement de la page) ;
- les UCR `routing.*` (routes associées) ;
- les UCR `i18n.*` (textes spécifiques de la page).

---

## 3. 🧠 Règles générales de génération des pages

### 3.1. Page = rootView UCR

Chaque page générée correspond à une **rootView** dans le DSL :

- UCR typique : `structure.rootView.<PageName>-1` ;  
- cette rootView est reliée à `${project.pageName}` via le bridge et `mapping.structure`.

La page doit donc être conçue comme **l’orchestrateur UI** de cette rootView :

- elle instancie les containers et atoms nécessaires ;
- elle branche la logique métier et les flux de données ;
- elle intègre le layout et l’i18n ;
- elle expose les points d’entrée vers les actions et effets.

### 3.2. Pages agnostiques de la stack

Le guide ne présuppose pas :

- l’existence d’un composant `Page` spécifique,
- la forme des imports,
- la syntaxe exacte (JSX, templates, etc.).

Les détails concrets sont fournis par les `stack-guides`.  
Ce guide décrit seulement les **rôles** et **relations** entre les éléments (pages, containers, hooks, etc.).

### 3.3. Pages = orchestration, pas logique métier lourde

Principes :

- la logique métier vit dans :
  - `hooks-logic`,
  - `actions`,
  - `services`,
  - éventuellement certains `stores` ;
- la page se contente de :
  - instancier les hooks ;
  - transmettre les données / callbacks aux containers ;
  - gérer la composition visuelle (ordre, groupes, layout).

### 3.4. Traçabilité UCR

La page doit conserver, de façon structurée (métadonnées, commentaires, annotations stack-spécifiques) :

- les UCR principaux qu’elle orchestre :
  - `structure.rootView.*`
  - `layout.pageLayout.*`
  - `routing.route.*`
  - `logic.viewLifecycle.*`
  - `actions.*`
  - `i18n.*`

Ce lien facilite :

- les évolutions ultérieures,
- les diagnostiques automatisés,
- la régénération future.

---

## 4. 🧬 Patterns de pages (au niveau conceptuel)

### 4.1. Page simple

Caractéristiques :

- 1 rootView ;
- 1 container principal ;
- layout minimal (ou implicite) ;
- pas de paramètres de route critiques ;
- peu ou pas de logique de workflow.

Usage typique :

- page d’accueil simple,
- page d’info statique légèrement dynamique.

### 4.2. Page paramétrée

Caractéristiques :

- dépend de `routing.routeParam.*` (ex. `:id`, `:slug`) ;
- consomme au moins un `hook-data` dépendant de ces params ;
- gère les états `loading`, `notFound`, `error` et `success`.

La page doit :

- récupérer les paramètres de route via les abstractions définies dans `guide.routing.md` ;
- les transmettre aux hooks-data / hooks-logic responsables du chargement ;
- adapter le rendu en fonction des états.

### 4.3. Page multi-containers / composite

Caractéristiques :

- plusieurs sections cohérentes mais indépendantes ;
- plusieurs containers racines ou de premier niveau ;
- plusieurs flux de données simultanés.

La page doit :

- orchestrer la disposition de ces containers via `mapping.layout` ;
- s’assurer que les dépendances entre containers sont gérées (via hooks-logic / stores) ;
- éviter de mettre la logique métier d’orchestration dans la page.

### 4.4. Page type « workflow » / wizard

Caractéristiques :

- UCR `logic.workflow.*` présents ;
- plusieurs étapes de formulaire / navigation ;
- transitions entre étapes.

La page doit :

- déléguer l’état du workflow à des hooks-logic / stores ;
- se contenter de :
  - afficher l’étape courante ;
  - brancher les actions `next`, `previous`, `submit` ;
  - gérer l’affichage conditionnel et le layout d’étape.

### 4.5. Page avec layout avancé

Caractéristiques :

- layout explicitement mappé (header, sidebar, footer, main, etc.) ;
- containers mappés à des zones spécifiques.

La page doit :

- instancier le layout recommandé par `guide.layout.md` ;
- utiliser les zones définies (`mapping.layout`) pour insérer les containers ;
- respecter les conventions d’accessibilité (landmarks, structure d’en-tête…).

---

## 5. 🌍 Internationalisation, accessibilité, état & effets

### 5.1. Internationalisation (i18n)

Basé sur `mapping.i18n` et `guide.i18n.md` :

- utiliser les primitives d’i18n de la stack (composants / fonctions) ;
- éviter les chaînes en dur lorsqu’une clé existe ;
- regrouper les clés de page dans un namespace cohérent (ex. `pages.${project.pageName}.*`).

### 5.2. Accessibilité (a11y)

Même si la mise en œuvre dépend de la stack, la page doit respecter les principes :

- hiérarchie de titres (`h1`, `h2`, etc.) alignée avec le layout ;
- rôles ARIA pour les parties importantes ;
- messages d’erreur/états rendus lisibles par les lecteurs d’écran.

### 5.3. Effets et feedback utilisateur

À partir de `mapping.effects` :

- déclencher les effets via les **abstractions** de la stack (hooks d’effets, services dédiés, etc.) ;
- ne pas coder les effets directement dans la page de manière ad-hoc ;
- distinguer :
  - effets bloquants (rare, ex. modals critiques) ;
  - effets non bloquants (toasts, logs, analytics).

---

## 6. 🗂 Structure des fichiers de pages générés

Les pages sont générées dans :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/`

La structure interne et les noms sont déterminés par :

- `project-structure.json` ;
- `stack-guides/guide.ui-pages.md` ;
- éventuellement `guide.routing.md` (si la stack attend une co-localisation des pages et routes).

Exemples conceptuels (à adapter via stack-guides, ne jamais coder en dur) :

- `pages/<PageName>Page.<ext>`  
- `pages/<PageName>/index.<ext>`  
- `pages/<routeSegment>/page.<ext>`  

Le guide doit indiquer :

- comment la page est exportée (default export, named export, factory, etc.) ;
- comment elle est reliée au router (directement ou via des fichiers séparés).

---

## 7. 📝 `pages.meta.json`

En plus des fichiers de page, le stage doit produire :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/pages/pages.meta.json`

Ce fichier doit contenir :

```jsonc
{
  "pageName": "${project.pageName}",
  "rootViewUcr": "structure.rootView.MyPage-1",
  "containersCount": 3,
  "hasLayout": true,
  "hasRoutingInfo": true,
  "hooksDataUsed": ["useMyPageData"],
  "hooksLogicUsed": ["useMyPageLogic"],
  "storesUsed": ["useMyPageStore"],
  "actionsUsed": ["action.saveMyPage"],
  "effectsUsed": ["effect.showSuccessToast"],
  "ucr": {
    "structure": ["structure.rootView.MyPage-1"],
    "layout": ["layout.pageLayout.MyPage-1"],
    "routing": ["routing.route.MyPageMain-1"],
    "i18n": ["i18n.page.MyPage.title-1"],
    "logic": ["logic.viewLifecycle.MyPage-1"],
    "actions": ["actions.saveMyPage-1"]
  },
  "generatedFiles": [
    "pages/MyPagePage.ext"
  ],
  "issues": []
}
```

Ce fichier :

- documente la configuration réelle de la page ;
- permet à d’autres étapes/ou outils de diagnostiquer la page ;
- facilite l’analyse de couverture (tests vs pages vs routes).

---

## 8. ✅ Checklist de génération pour `pages`

Avant de considérer que la génération de la page `${project.pageName}` est terminée :

- [ ] `structure.rootView.*` est identifié pour la page via DSL + bridge + `mapping.structure`  
- [ ] `mapping.structure.json` et `mapping.layout.json` sont présents, lisibles et validés  
- [ ] Les stack-guides de pages (`guide.ui-pages.md`) sont disponibles et interprétés  
- [ ] Les containers et atoms nécessaires existent déjà (stages 55–56)  
- [ ] Les hooks-data, hooks-logic, stores, services nécessaires existent (stages 51–54)  
- [ ] La page ne contient pas de logique métier lourde (déléguée aux hooks/ services/ actions)  
- [ ] L’i18n est correctement branchée, sans chaînes critiques en dur  
- [ ] Les effets principaux sont déclenchés via les abstractions prévues  
- [ ] Au moins un fichier de page a été généré sous `phase-3-generation/pages/`  
- [ ] `pages.meta.json` a été généré et liste correctement UCR, hooks, containers et fichiers

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
