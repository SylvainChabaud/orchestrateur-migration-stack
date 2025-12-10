# 🔧 Guide Génération — i18n

*(Domaine de génération : **i18n** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de génération

Le domaine **i18n** décrit comment générer les **ressources d’internationalisation** pour `${project.pageName}` en s’appuyant sur :

- les **UCR i18n** (labels, titres, messages, placeholders, erreurs, tooltips…) définis dans le DSL ;
- le **bridge legacy → DSL**, qui explique d’où viennent les textes d’origine ;
- les **mappings de Phase 2** (`mapping.i18n.json` en particulier) qui indiquent comment transformer ces UCR en clés et namespaces ;
- les **stack-guides d’i18n**, qui imposent un format, une structure de clés et une organisation des fichiers adaptée à la stack cible.

Objectif : produire des fichiers de ressources i18n **directement consommables** par les pages, components, hooks et services générés dans les autres stages de la Phase 3, tout en garantissant :

- une **traçabilité claire** UCR → clé i18n ;
- l’absence de chaînes « cachées » en dur là où une clé existe ;
- une **organisation cohérente** (par page, par domaine, par module) selon les stack-guides.

Ce guide est **agnostique de la stack** : il ne parle ni de `react-i18next`, ni d’`ngx-translate`, ni de toute autre lib spécifique. Il décrit uniquement les **concepts** et la **structure** ; les détails d’API et de format exact viennent des stack-guides.

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

Les chemins des ressources i18n générées dérivent de `${paths.*}` + `project-structure.json` + stack-guides.

### 2.2. Artefacts Phase 0 — Stack, structure, bridge

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `bridge-legacy-to-dsl.json`
- `stack-guides/guide.stack.md`
- `stack-guides/guide.i18n.md`
- éventuellement :  
  - `stack-guides/guide.ui-pages.md`  
  - `stack-guides/guide.ui-components.md`  

Les stack-guides i18n doivent préciser :

- la manière dont les **locales** sont gérées (mono-locale vs multi-locales) ;
- la **structure de clés** (flat, nested, par namespace, par domaine) ;
- la **structure des fichiers de ressources** (un fichier par page, par module, par locale…) ;
- comment les **pages / components** doivent consommer ces clés (APIs, helpers, hooks, etc.).

### 2.3. Artefacts Phase 2 — Mappings

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/` :

- `mapping.i18n.json` (source principale des clés et namespaces) ;
- `mapping.structure.json` (association des clés à la structure vue / containers) ;
- `mapping.layout.json` (zone/section associée à certaines clés) ;
- `mapping.routing.json` (clés pour les éléments de navigation) ;
- `mapping.logic.json` (messages liés aux workflows) ;
- `mapping.actions.json` (messages de résultat d’actions) ;
- `mapping.effects.json` (toasts, notifications, logs visibles) ;
- `mapping.config.json` et `mapping.conditions.json` (clés conditionnelles / contextuelles) ;
- `mapping.tests.json` (messages attendus dans les tests, par exemple erreurs de validation) ;
- `mappings-summary.json` (validation globale des mappings).

### 2.4. DSL + UCR + bridge

- `Spec Dsl Orchestrator`
- `Spec Ucr Orchestrator`
- `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json`

Ils permettent de :

- recenser tous les UCR `i18n.*` pertinents pour `${project.pageName}` ;
- comprendre le **type** de chaque UCR (label, helper, error, tooltip, etc.) ;
- vérifier qu’aucun UCR critique (ex. titres de sections) n’est oublié dans le mapping.

---

## 3. 🧠 Règles générales de génération i18n

### 3.1. Tout texte fonctionnel doit être mappé

Le générateur i18n doit viser à ce que :

- tous les textes fonctionnels visibles (titres, labels, messages d’erreur/succès, placeholders…) aient une **clé i18n** ;
- aucune chaîne métier importante ne soit laissée « en dur » dans les pages / components, sauf cas explicitement prévu par la stack.

### 3.2. Traçabilité UCR → clé i18n

Chaque clé générée doit :

- référencer l’UCR d’origine (`i18n.*`, `logic.*`, `actions.*`, etc.) ;  
- être liée, via `mapping.structure`, à un élément de l’UI (vue, section, container, champ).

Cette traçabilité est utile :

- pour les audits de couverture i18n ;
- pour les évolutions futures des textes sans perdre le lien avec les comportements métier.

### 3.3. Respect strict des conventions des stack-guides

Le guide i18n ne fixe pas un **format** ou une **API** ; ce rôle appartient aux stack-guides :

- format de fichier (JSON, YAML, TS, etc.) ;
- localisation des fichiers (dossiers, arborescences) ;
- conventions de namespace (ex. `pages.${project.pageName}`, `common`, `layout`, etc.) ;
- organisation multi-locale.

Le stage de génération doit **appliquer** ce qui est défini dans `guide.i18n.md` sans inventer.

### 3.4. Gestion des locales

En fonction de la stack, deux approches courantes :

1. **Mono-locale** à la génération (on ne gère que la locale de base, les traductions viennent plus tard) ;
2. **Multi-locales** avec plusieurs fichiers dès la génération (ex. `en`, `fr`, `es`, etc.).

Ce guide n’impose pas de stratégie, il se contente de :

- structurer les clés de manière cohérente ;
- laisser la possibilité d’étendre ultérieurement à d’autres locales.

La stratégie exacte est définie dans `guide.i18n.md`.

---

## 4. 🧬 Typologie de contenus i18n

### 4.1. Titres & en-têtes (page, sections, blocs)

- UCR typiques : `i18n.pageTitle.*`, `i18n.sectionTitle.*` ;
- utilisés pour les :
  - titres de pages ;
  - titres de sections (cards, panels, blocs).

Règles :

- toujours prévoir une clé pour :
  - le titre principal de la page ;
  - les sections structurantes (notamment celles visibles dans le layout).

### 4.2. Labels & placeholders

- UCR : `i18n.label.*`, `i18n.placeholder.*` ;
- utilisés pour les composants de formulaire, champs de saisie, boutons d’action principaux.

Règles :

- les labels doivent être distincts des placeholders ;
- prévoir des clés claires et stables (ex. `form.campaignName.label`, `form.campaignName.placeholder`).

### 4.3. Messages d’erreur & de succès

- UCR : `i18n.error.*`, `i18n.success.*`, parfois `i18n.warning.*` ;
- utilisés dans :
  - validations de formulaire ;
  - retours d’actions (ex. création, suppression).

Règles :

- proposer des messages suffisamment explicites pour l’utilisateur final ;
- éviter de mettre les messages d’erreur dans la logique métier en dur ;  
  → toujours les externaliser dans i18n.

### 4.4. Aides contextuelles / tooltips / helper texts

- UCR : `i18n.helper.*`, `i18n.tooltip.*` ;
- apportent des explications supplémentaires sur certaines actions ou champs.

Règles :

- garder ces textes éventuellement plus longs et structurés ;
- prévoir leur possible absence (affichage optionnel).

### 4.5. Textes de navigation & breadcrumbs

- UCR : `i18n.nav.*`, `i18n.breadcrumb.*` ;
- liés à `mapping.routing.json`.

Règles :

- aligner les clés de navigation avec la structure de routing ;
- permettre une composition cohérente des breadcrumbs (ex. `nav.campaigns.list`, `nav.campaigns.detail`).

---

## 5. 🗂 Structure des fichiers de ressources

Les ressources sont écrites sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/`

La structure exacte (fichiers, extensions, répertoires) est définie par :

- `project-structure.json` ;
- `stack-guides/guide.i18n.md`.

Quelques exemples conceptuels (non prescriptifs) :

- `i18n/${locale}/pages.${project.pageName}.json`  
- `i18n/pages/${project.pageName}/${locale}.json`  
- `i18n/pages/${project.pageName}.json` (mono-locale)  

Le guide stack indique aussi :

- comment associer chaque fichier à un namespace ;
- comment les pages / components doivent charger ces ressources.

---

## 6. 📝 `i18n.meta.json`

Le stage génère également :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/i18n/i18n.meta.json`

Ce fichier doit au minimum contenir :

```jsonc
{
  "pageName": "${project.pageName}",
  "namespaces": ["pages.${project.pageName}"],
  "locales": ["fr"],
  "keysCount": 42,
  "missingKeysCount": 3,
  "duplicatedKeysCount": 1,
  "ucr": {
    "i18n": [
      "i18n.pageTitle.MyPage-1",
      "i18n.label.MyPage.submit-1"
    ],
    "structure": [
      "structure.rootView.MyPage-1"
    ],
    "routing": [
      "routing.route.MyPageMain-1"
    ]
  },
  "generatedFiles": [
    "i18n/fr/pages.${project.pageName}.json"
  ],
  "issues": [
    "Missing key for i18n.error.MyPage.validation-1",
    "Duplicated key pages.${project.pageName}.submit"
  ]
}
```

Ce fichier est utile pour :

- les outils de QA i18n ;
- la validation manuelle ;
- les extensions ultérieures (nouvelles locales, audits de complétude…).

---

## 7. ✅ Checklist de génération pour `i18n`

Avant de considérer que la génération i18n est complète pour `${project.pageName}` :

- [ ] `mapping.i18n.json` est présent, lisible et validé dans `mappings-summary.json`  
- [ ] Toutes les UCR critiques (`pageTitle`, sections principales, etc.) ont une clé i18n correspondante  
- [ ] Les labels et placeholders des champs importants sont externalisés  
- [ ] Les messages d’erreur de validation utilisés dans les tests ou la logique sont externalisés  
- [ ] Les textes de navigation (liés au routing) disposent de clés cohérentes  
- [ ] Les ressources i18n sont générées sous `phase-3-generation/i18n/` selon la structure attendue par les stack-guides  
- [ ] Le ou les namespaces sont correctement définis (ex. `pages.${project.pageName}`)  
- [ ] `i18n.meta.json` a été généré avec les compteurs de clés, namespaces, locales et issues  
- [ ] Aucun blocking issue n’empêche l’usage des ressources dans la stack cible

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
