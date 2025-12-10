# 🔗 Guide Bridge — Legacy Stack ↔ DSL Interne (Template)
*(À instancier en Stage 02 — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Ce guide sert de **pont entre la stack Legacy réelle** du projet et le **DSL interne générique** utilisé par tous les inventaires et mappings.

Il permet :

- d’interpréter correctement le code Legacy,
- de déterminer comment identifier les concepts DSL dans ce code,
- d’assurer que toutes les étapes d’analyse (Inventaires) et d’interprétation (Mappings) utilisent une correspondance cohérente et stable,
- de rendre le pipeline **agnostique du framework Legacy** (React, Vue, Angular, Twig…).

---

## 2. 📚 Structure du guide

Le guide doit contenir :

- `legacyStack`: le nom de la stack Legacy détectée ou déclarée (ex: "react-17")
- `dslConcepts[]`: liste des concepts génériques utilisés dans toute la pipeline
- pour chaque concept :
  - une **description générique**
  - une **liste de patterns Legacy** permettant à l'IA de reconnaître ce concept dans le code original

---

## 3. 📦 Schéma JSON attendu

```json
{
  "domain": "bridge-legacy-to-dsl",
  "legacyStack": "string",
  "dslConcepts": [
    {
      "id": "structure.viewNode",
      "description": "Unité de vue / composant / template participant à la hiérarchie",
      "legacyPatterns": [
        "Pattern concret stack-specific (ex: function component returning markup)"
      ]
    }
  ]
}
```

---

## 4. 🧩 Catalogue DSL complet (à instancier dans le guide généré)

> ℹ️ **Important** : tous les `id` listés ici doivent avoir une entrée dans `dslConcepts[]` du JSON final, même si `legacyPatterns` est vide dans un premier temps.  
> Le Stage 02 peut compléter progressivement les `legacyPatterns` en fonction de la stack et de l’évolution du projet.

### Structure
- structure.viewNode
- structure.rootView
- structure.childView
- structure.containerView
- structure.presentationalView
- structure.fragment
- structure.slot
- structure.portal
- structure.viewHierarchy

### Logic
- logic.localState
- logic.derivedState
- logic.effect
- logic.sideEffect
- logic.businessRule
- logic.computedSelector
- logic.errorBoundary

### Effects
- effect.logicTriggered  
  *(effet déclenché par un changement logique : state/props/conditions ; souvent un useEffect ou watcher dépendant de variables)*
- effect.async  
  *(polling, timers, intervals, timeouts, jobs async récurrents)*
- effect.ui.focus  
  *(mise au focus d’un élément, autofocus, gestion du focus à l’affichage)*
- effect.ui.scroll  
  *(scroll automatique, scrollIntoView, repositionnement de la vue)*
- effect.ui.animation  
  *(animations déclenchées par effet : ajout/retrait de classes CSS, Web Animations API, etc.)*
- effect.ui.measure  
  *(mesures DOM : getBoundingClientRect, resize observers, mesure de taille/position)*
- effect.lifecycle.mount  
  *(effet exécuté au montage de la vue/composant : équivalent componentDidMount / onMounted)*
- effect.lifecycle.unmount  
  *(effet exécuté au démontage / cleanup : désabonnements, clearInterval, removeEventListener, etc.)*
- effect.navigation  
  *(effet provoquant une navigation : redirect, pushState, navigate(), changement d’URL)*
- effect.eventDriven  
  *(effet déclenché par des événements globaux ou custom : window, document, EventBus, etc.)*

### Data
- data.dataSource
- data.query
- data.mutation
- data.command
- data.endpoint
- data.requestPayload
- data.responsePayload
- data.pagination
- data.filter
- data.sort
- data.cacheKey
- data.errorHandling

### Conditions
- condition.visibility
- condition.enabledDisabled
- condition.branchIf
- condition.branchSwitch
- condition.logicalAndOr
- condition.featureFlag
- condition.roleGuard
- condition.dataGuard

### Events
- event.handler
- event.click
- event.submit
- event.change
- event.input
- event.focusBlur
- event.keyboard
- event.mouse
- event.navigation
- event.custom

### Layout
- layout.region
- layout.section
- layout.grid
- layout.stack
- layout.card
- layout.list
- layout.table
- layout.modal
- layout.drawer
- layout.panel
- layout.overlay
- layout.responsiveBreakpoint

### Forms
- form.formRoot
- form.field
- form.fieldGroup
- form.validationRule
- form.validationSchema
- form.errorMessage
- form.submitAction
- form.resetAction
- form.wizardStep

### i18n
- i18n.namespace
- i18n.key
- i18n.message
- i18n.pluralRule
- i18n.interpolationVar
- i18n.localeFile

### Routing
- routing.routeConfig
- routing.routePath
- routing.routeParam
- routing.queryParam
- routing.routeGuard
- routing.redirect
- routing.nestedRoute
- routing.routeLayout

### Tests
- test.unitCase
- test.integrationCase
- test.e2eCase
- test.selector
- test.fixture
- test.mock
- test.assertion
- test.scenario

### Accessibilité
- a11y.role
- a11y.ariaAttribute
- a11y.landmark
- a11y.focusTrap
- a11y.keyboardNav

### Analytics / Perf
- analytics.event
- analytics.tag
- performance.lazyChunk
- performance.memoization

### Styles
- styles.source  
  *(fichier CSS, CSS-in-JS, styled-components, thème, tokens…)*
- styles.rule  
  *(règle CSS significative, classe, selector…)*
- styles.theme  
  *(thème global : light/dark, enterprise/consumer…)*
- styles.token  
  *(design token : couleurs, espacements, typographie…)*
- styles.variant  
  *(variante de composant : primary/secondary, small/large…)*
- styles.responsive  
  *(media queries, breakpoints, responsive utilities)*
- styles.animation  
  *(animations CSS, keyframes, transitions…)*

### Config
- config.source  
  *(fichier de config, constantes, env vars…)*
- config.parameter  
  *(paramètre de configuration : API endpoint, feature flag, timeout…)*
- config.featureFlag  
  *(feature flag, A/B test, expérimentation…)*
- config.environment  
  *(variables d'environnement : NODE_ENV, API_URL…)*
- config.constant  
  *(constante applicative : limites, formats, règles…)*

### Hooks
- hooks.standard  
  *(hooks React/Vue standards : useState, useEffect, computed…)*
- hooks.custom  
  *(hooks custom métier : useAuth, useCampaigns…)*
- hooks.composition  
  *(composition de hooks : hook appelant d'autres hooks)*
- hooks.context  
  *(hooks de contexte : useContext, useTheme…)*
- hooks.lifecycle  
  *(hooks de lifecycle : onMounted, onBeforeUnmount…)*

### Dataflows
- dataflows.query  
  *(lecture de données : GET, fetch initial, queries…)*
- dataflows.mutation  
  *(écriture de données : POST/PUT/DELETE, updates…)*
- dataflows.cache  
  *(stratégie de cache : stale-while-revalidate, cache-first…)*
- dataflows.optimistic  
  *(optimistic updates, rollback…)*
- dataflows.subscription  
  *(subscriptions, real-time, WebSocket…)*
- dataflows.pagination  
  *(pagination : offset, cursor, infinite scroll…)*
- dataflows.filtering  
  *(filtrage de données : search, filters, facets…)*

### Async
- async.promise  
  *(promises, async/await…)*
- async.retry  
  *(retry logic, exponential backoff…)*
- async.timeout  
  *(timeouts, cancellation…)*
- async.throttle  
  *(throttling, rate limiting…)*
- async.debounce  
  *(debouncing, delayed execution…)*
- async.polling  
  *(polling, periodic checks…)*
- async.concurrency  
  *(parallel, sequential, race, all…)*

### Services
- services.apiClient  
  *(client HTTP : axios, fetch wrapper…)*
- services.repository  
  *(repository pattern, data access layer…)*
- services.adapter  
  *(adapter, mapper, transformer…)*
- services.facade  
  *(facade, orchestration de plusieurs services…)*
- services.integration  
  *(intégration externe : third-party, SDK…)*

### Actions
- actions.userAction  
  *(action utilisateur : click, submit, selection…)*
- actions.systemAction  
  *(action système : auto-save, auto-refresh, timer…)*
- actions.workflow  
  *(workflow métier : séquence d'étapes, wizard…)*
- actions.transaction  
  *(transaction multi-étapes : rollback, commit…)*
- actions.orchestration  
  *(orchestration : coordination de plusieurs actions…)*

---

## 5. 🧩 Notes IA

- Ne jamais inventer de patterns Legacy totalement déconnectés de la stack réelle.
- Toujours maintenir la correspondance stable pour toutes les phases d’analyse et d’interprétation.
- Le guide doit être exhaustif : **tous** les ids DSL listés doivent avoir une entrée dans `dslConcepts[]`.
- Si certains `legacyPatterns` ne peuvent pas encore être déterminés, laisser un tableau vide `[]` documenté dans la description.
- Le JSON doit être sérialisable, propre, lisible, sans commentaires.

---

© 2025 — ai-orchestrator-v4
