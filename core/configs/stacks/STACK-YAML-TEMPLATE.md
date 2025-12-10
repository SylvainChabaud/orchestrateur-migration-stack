# 📘 Guide de remplissage : my-custom-enterprise.stack.yaml
**Version :** v2 - Consolidée pour génération de guides  
**Date :** 8 décembre 2025

---

## 🎯 Objectif de ce fichier

Ce fichier YAML est **LA SOURCE DE VÉRITÉ** pour décrire ta stack finale.  
Il sera utilisé par l'orchestrateur pour générer **26 guides de stack** cohérents.

---

## 📋 Structure du fichier

```
my-custom-enterprise.stack.yaml
├── stackId, label, description     ← Identité de la stack
├── metadata                        ← 🆕 Métadonnées pour génération guides
│   ├── architecture               ← Type projet, structure dossiers
│   ├── naming                     ← Conventions de nommage
│   ├── projectStructure           ← Arborescence src/
│   ├── performance                ← Libs lourdes, optimisations, métriques
│   ├── accessibility              ← Standards WCAG, outils, métriques
│   ├── qualityThresholds          ← Seuils de validation Phase 4
│   └── layouts                    ← Layouts disponibles, responsive
└── tools                          ← Technologies de la stack
    ├── runtime                    ← TypeScript
    ├── frontend                   ← React
    ├── routing                    ← React Router
    ├── i18n                       ← i18next
    ├── design                     ← Design System, MUI, Emotion
    ├── stateManagement            ← Zustand, React Query
    ├── api                        ← Axios, Orval
    ├── validation                 ← Zod
    ├── forms                      ← react-hook-form
    ├── build                      ← Vite
    ├── tests                      ← Vitest, RTL, Cypress
    ├── auth                       ← OIDC
    ├── devTools                   ← ESLint, Prettier, Husky
    ├── packageManagers            ← Yarn, pnpm, bun
    ├── monorepo                   ← Nx, Workspaces
    ├── automation                 ← Renovate
    └── documentation              ← Storybook, Docusaurus
```

---

## 🆕 Section `metadata` (NOUVEAU)

Cette section contient les informations **structurelles et conventionnelles** de ton projet.

### 1. `metadata.architecture`

**Objectif :** Décrire l'architecture globale du projet.

```yaml
metadata:
  architecture:
    type: "monorepo"              # monorepo | single-app | microfrontend
    folderStructure: "nx-workspaces"
    packageManagement: "yarn-classic"
```

**Valeurs possibles :**
- `type` : `monorepo`, `single-app`, `microfrontend`
- `folderStructure` : `nx-workspaces`, `lerna`, `turborepo`, `flat`, `custom`
- `packageManagement` : `yarn-classic`, `yarn-berry`, `pnpm`, `npm`, `bun`

---

### 2. `metadata.naming`

**Objectif :** Définir les conventions de nommage du projet.

```yaml
metadata:
  naming:
    files:
      components: "PascalCase"    # Button.tsx, UserProfile.tsx
      hooks: "camelCase"          # useAuth.ts, useForm.ts
      pages: "PascalCase"         # Dashboard.tsx
      utils: "camelCase"          # formatDate.ts
    functions:
      handlers: "handleXxx"       # handleClick, handleSubmit
      events: "onXxx"             # onClick, onSubmit
    constants:
      style: "UPPER_SNAKE_CASE"   # API_BASE_URL, MAX_RETRIES
```

**Valeurs possibles :**
- `PascalCase` : `MyComponent`, `UserProfile`
- `camelCase` : `myFunction`, `userData`
- `kebab-case` : `my-component`, `user-data`
- `snake_case` : `my_function`, `user_data`
- `UPPER_SNAKE_CASE` : `MY_CONSTANT`, `API_URL`

---

### 3. `metadata.projectStructure`

**Objectif :** Décrire l'arborescence des dossiers `src/`.

```yaml
metadata:
  projectStructure:
    srcLayout: |
      src/
      ├── pages/              # Pages applicatives
      ├── components/         # Composants réutilisables
      │   ├── atoms/          # Composants atomiques
      │   ├── molecules/      # Composants composés
      │   └── organisms/      # Composants complexes
      ├── hooks/              # Custom hooks
      ├── services/           # Services API
      ├── stores/             # Stores Zustand
      ├── types/              # Définitions TypeScript
      ├── utils/              # Utilitaires
      ├── locales/            # i18n
      └── routes/             # Configuration routing
```

**Conseil :** Utilise un bloc texte multi-ligne (`|`) pour dessiner ton arborescence.

---

### 4. `metadata.performance`

**Objectif :** Définir les stratégies de performance et les métriques cibles.

```yaml
metadata:
  performance:
    heavyLibraries:
      - name: "chart.js"
        strategy: "dynamic-import"
      - name: "@mui/x-data-grid"
        strategy: "lazy-load"
    optimization:
      lazyLoading: true
      codeSplitting: true
      treeShaing: true
    targets:
      tti: "< 3s"             # Time to Interactive
      fcp: "< 1.5s"           # First Contentful Paint
      lighthouseScore: "> 90"
```

**Stratégies possibles :**
- `dynamic-import` : `import()` dynamique
- `lazy-load` : `React.lazy()`
- `code-splitting` : Split par route ou feature
- `vendor-splitting` : Vendor bundle séparé

---

### 5. `metadata.accessibility`

**Objectif :** Définir les standards d'accessibilité et les métriques.

```yaml
metadata:
  accessibility:
    standard: "WCAG 2.1 Level AA"
    requirements:
      - "Keyboard navigation complète"
      - "Screen reader compatible"
      - "Contraste minimum 4.5:1"
      - "Alt text obligatoire"
      - "ARIA labels sur interactive"
    tools:
      - "axe DevTools"
      - "Lighthouse Accessibility"
    targets:
      lighthouseScore: "> 95"
      axeCriticalViolations: 0
```

**Standards possibles :**
- `WCAG 2.0 Level A`
- `WCAG 2.0 Level AA`
- `WCAG 2.1 Level AA` (recommandé)
- `WCAG 2.2 Level AA`

---

### 6. `metadata.qualityThresholds`

**Objectif :** Définir les seuils de validation pour la Phase 4.

```yaml
metadata:
  qualityThresholds:
    validation:
      staticConsistency:
        minScore: 70
        maxUnresolvedImports: 0
        maxCircularDeps: 0
      testsAudit:
        minCoverage: 60
        minCriticalCoverage: 100
      functionalEquivalence:
        minScore: 70
        criticalUcrMustBeCovered: true
      dependencies:
        maxUnresolved: 0
        maxUnexpected: 3
        minScore: 70
      integration:
        maxRouteConflicts: 0
        maxLayoutConflicts: 0
        maxMissingEntryFiles: 0
        minScore: 70
      accessibility:
        minScore: 70
        maxInteractiveWithoutLabel: 0
        maxImagesWithoutAlt: 0
      performance:
        minScore: 70
        maxHeavyImportsWithoutLazyLoading: 0
        maxLargeListsWithoutOptimization: 0
      qualityCompliance:
        minScore: 80
        maxForbiddenPatterns: 0
    globalScore:
      weights:
        static: 0.15
        tests: 0.20
        functional: 0.25
        dependencies: 0.10
        integration: 0.15
        accessibility: 0.05
        performance: 0.05
        quality: 0.05
      minGlobalScore: 70
```

**Conseil :** Ajuste les seuils selon ton niveau d'exigence qualité.

---

### 7. `metadata.layouts`

**Objectif :** Lister les layouts disponibles dans ton application.

```yaml
metadata:
  layouts:
    available:
      - name: "MainLayout"
        description: "Layout principal avec header, sidebar, footer"
        zones: ["header", "sidebar", "content", "footer"]
      - name: "AuthLayout"
        description: "Layout pour pages d'authentification"
        zones: ["content"]
      - name: "AdminLayout"
        description: "Layout admin avec navigation latérale étendue"
        zones: ["header", "sidebar-expanded", "content"]
    responsive:
      breakpoints:
        xs: "0px"
        sm: "600px"
        md: "960px"
        lg: "1280px"
        xl: "1920px"
```

---

## 🔧 Section `tools` (AMÉLIORÉE)

Cette section liste toutes les technologies de ta stack avec **versions précises** et **patterns recommandés**.

### Exemple enrichi pour `tools.stateManagement`

```yaml
tools:
  stateManagement:
    globalState:
      library: "Zustand"
      version: "5.0.8"
      patterns:
        - "Slice pattern par feature"
        - "Selectors avec shallow equality"
        - "Devtools enabled"
    serverState:
      library: "React Query (TanStack Query)"
      version: "5.90.7"
      patterns:
        - "Cache par query key"
        - "Stale time: 5 minutes"
        - "Retry: 3 attempts"
```

**Nouveauté :** Ajoute un champ `patterns` pour documenter les bonnes pratiques.

---

## ✅ Checklist de remplissage

### Étape 1 : Identité de la stack
- [ ] `stackId` : identifiant unique (kebab-case)
- [ ] `label` : nom lisible
- [ ] `description` : résumé de la stack

### Étape 2 : Metadata (OBLIGATOIRE)
- [ ] `metadata.architecture` : type, structure, package manager
- [ ] `metadata.naming` : conventions de nommage complètes
- [ ] `metadata.projectStructure` : arborescence `src/`
- [ ] `metadata.performance` : libs lourdes, optimisations, métriques
- [ ] `metadata.accessibility` : standards, requirements, outils, métriques
- [ ] `metadata.qualityThresholds` : seuils pour Phase 4
- [ ] `metadata.layouts` : layouts disponibles, breakpoints

### Étape 3 : Tools (OBLIGATOIRE)
- [ ] `tools.runtime` : TypeScript version et config
- [ ] `tools.frontend` : React version et patterns
- [ ] `tools.routing` : Router et features
- [ ] `tools.i18n` : i18next et stratégie
- [ ] `tools.design` : Design System, MUI, Emotion
- [ ] `tools.stateManagement` : Zustand + React Query avec patterns
- [ ] `tools.api` : Axios + Orval avec config
- [ ] `tools.validation` : Zod avec usages
- [ ] `tools.forms` : react-hook-form avec patterns
- [ ] `tools.build` : Vite avec plugins
- [ ] `tools.tests` : Vitest, RTL, Cypress avec coverage
- [ ] `tools.auth` : OIDC avec stratégie
- [ ] `tools.devTools` : ESLint, Prettier, Husky
- [ ] `tools.packageManagers` : Yarn, pnpm, bun
- [ ] `tools.monorepo` : Nx, Workspaces
- [ ] `tools.automation` : Renovate
- [ ] `tools.documentation` : Storybook, Docusaurus

---

## 🎯 Mapping : Section YAML → Guides générés

| Section YAML | Guide(s) | Obligatoire |
|-------------|----------|-------------|
| `metadata.architecture` | `guide.stack.md`, `guide.monorepo.md` | ✅ |
| `metadata.naming` | `guide.naming.md` | ✅ |
| `metadata.projectStructure` | `guide.structure.md` | ✅ |
| `metadata.performance` | `guide.performance.md` | ✅ |
| `metadata.accessibility` | `guide.accessibility.md` | ✅ |
| `metadata.qualityThresholds` | `guide.quality-thresholds.md` | ✅ |
| `metadata.layouts` | `guide.layout.md` | ✅ |
| `tools.runtime` | `guide.types.md`, `guide.conventions.md` | ✅ |
| `tools.frontend` | `guide.ui-components.md`, `guide.ui-pages.md` | ✅ |
| `tools.design` | `guide.ui.atoms.md`, `guide.ui.props.md`, `guide.styles.md` | ✅ |
| `tools.routing` | `guide.routing.md` | ✅ |
| `tools.i18n` | `guide.i18n.md` | ✅ |
| `tools.stateManagement` | `guide.state-management.md`, `guide.stores.md`, `guide.state.md` | ✅ |
| `tools.api` | `guide.api-client.md` | ✅ |
| `tools.validation` | `guide.validation.md` | ✅ |
| `tools.forms` | `guide.forms.md` | ✅ |
| `tools.tests` | `guide.tests.md` | ✅ |
| `tools.build` | `guide.build-and-tooling.md` | ✅ |
| `tools.auth` | `guide.auth.md` | ✅ |

**Total : 26 guides seront générés**

---

## ⚠️ Erreurs fréquentes

### 1. Metadata incomplet
```yaml
❌ metadata:
     architecture:
       type: "monorepo"
     # MANQUE : naming, projectStructure, performance, etc.
```

### 2. Versions manquantes
```yaml
❌ tools:
     frontend:
       library: "React"
       # MANQUE : libraryVersion
```

### 3. Patterns non documentés
```yaml
❌ tools:
     stateManagement:
       globalState:
         library: "Zustand"
         version: "5.0.8"
         # MANQUE : patterns
```

---

## 🔍 Validation avant lancement

Avant de lancer l'orchestrateur, vérifie :

1. ✅ Le fichier YAML est syntaxiquement valide
2. ✅ Toutes les sections `metadata` sont remplies
3. ✅ Toutes les sections `tools` obligatoires sont présentes
4. ✅ Toutes les versions sont renseignées
5. ✅ Les patterns sont documentés pour les outils principaux

**Outil de validation :**  
L'orchestrateur affichera un résumé détaillé à la Question 3 pour validation.

---

## 📚 Ressources

- Template complet : [STACK-DEFINITION-CONSOLIDATION.md](STACK-DEFINITION-CONSOLIDATION.md)
- Config orchestrateur : [project-config.yaml](../project-config.yaml)
- Main orchestrator : [main-orchestrator.md](../../main-orchestrator.md)

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4  
**Guide créé le :** 8 décembre 2025  
**Status :** ✅ Template v2 pour génération complète de guides
