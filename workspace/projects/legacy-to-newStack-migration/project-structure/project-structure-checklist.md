# Project Structure Checklist — CampaignsDetail

## 📋 Vue d'ensemble

Ce document décrit les règles de structure de projet dérivées des guides de stack pour la migration de **CampaignsDetail**.

---

## 🏗️ Architecture

| Paramètre | Valeur |
|-----------|--------|
| **Type** | Feature-based |
| **Racine du projet** | `src/` |
| **Page cible** | CampaignsDetail |
| **Dossier feature** | `src/features/campaigns/` |

---

## 📂 Structure des Dossiers

### Feature CampaignsDetail

```
src/features/campaigns/
├── api/                    # Appels API + types
├── components/             # Composants UI de la feature
├── hooks/                  # Custom hooks (useQuery wrappers)
├── pages/                  # Composants de page
├── stores/                 # Zustand stores locaux
├── types/                  # TypeScript types
├── utils/                  # Helpers spécifiques
├── __tests__/              # Tests unitaires/intégration
└── index.ts                # Exports publics (barrel)
```

### Dossiers Partagés

```
src/
├── core/                   # Infrastructure partagée
│   ├── api/                # Client API, queryClient
│   ├── auth/               # Auth store, guards
│   ├── errors/             # Error boundaries, handlers
│   ├── i18n/               # Configuration i18next
│   └── theme/              # MUI theme configuration
├── components/             # Composants UI partagés
│   ├── atoms/              # Building blocks (Button, Input...)
│   ├── molecules/          # Combinaisons d'atomes
│   ├── organisms/          # Sections UI complexes
│   └── templates/          # Layouts de page
├── routes/                 # Configuration routing
├── locales/                # Fichiers de traduction
├── utils/                  # Helpers globaux
├── types/                  # Types globaux
└── test/                   # Setup tests et mocks
```

---

## 📝 Conventions de Nommage

### Fichiers & Dossiers

| Type | Convention | Exemple |
|------|------------|---------|
| **Composant** | PascalCase.tsx | `CampaignCard.tsx` |
| **Hook** | useCamelCase.ts | `useCampaigns.ts` |
| **Store** | camelCaseStore.ts | `campaignStore.ts` |
| **Types** | PascalCase.types.ts | `Campaign.types.ts` |
| **Test** | ComponentName.test.tsx | `CampaignCard.test.tsx` |
| **Styles** | ComponentName.styles.ts | `CampaignCard.styles.ts` |

### Structure d'un Composant

```
ComponentName/
├── ComponentName.tsx           # Composant React
├── ComponentName.test.tsx      # Tests unitaires
├── ComponentName.styles.ts     # Styles Emotion/MUI
├── ComponentName.types.ts      # Types (optionnel)
└── index.ts                    # Barrel export
```

---

## 📦 Ordre des Imports

```typescript
// 1. React et core libraries
import { useState, useCallback } from 'react';

// 2. Third-party libraries
import { useNavigate } from 'react-router';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';

// 3. Design system / UI components
import { Button, Box, Typography } from '@mui/material';
import { DataGrid } from '@peaksys/design-system';

// 4. Internal shared modules (absolute imports avec alias)
import { useAuth } from '@/hooks/useAuth';
import type { User } from '@/types/user.types';

// 5. Feature-specific imports (relative imports)
import { useFeatureStore } from '../stores/featureStore';
import { FeatureComponent } from './FeatureComponent';

// 6. Styles
import { StyledContainer } from './Component.styles';

// 7. Types (si séparés)
import type { ComponentProps } from './Component.types';
```

---

## 🔗 Alias de Chemins

| Alias | Chemin |
|-------|--------|
| `@/*` | `src/*` |
| `@components/*` | `src/components/*` |
| `@features/*` | `src/features/*` |
| `@hooks/*` | `src/hooks/*` |
| `@stores/*` | `src/stores/*` |
| `@types/*` | `src/types/*` |
| `@utils/*` | `src/utils/*` |
| `@services/*` | `src/services/*` |
| `@core/*` | `src/core/*` |

---

## 🌐 Internationalisation (i18n)

| Paramètre | Valeur |
|-----------|--------|
| **Dossier** | `src/locales/` |
| **Structure** | `locales/{lang}/{namespace}.json` |
| **Namespaces** | `common`, `campaigns`, `translation` |
| **Namespace par défaut** | `translation` |
| **Langue de fallback** | `en` |

---

## 🛣️ Routing

| Paramètre | Valeur |
|-----------|--------|
| **Dossier config** | `src/routes/` |
| **Pattern** | React Router 7 avec `createBrowserRouter` |
| **Routes protégées** | ✅ Oui |
| **Constantes de routes** | `src/routes/paths.ts` |

---

## 🧪 Testing

| Paramètre | Valeur |
|-----------|--------|
| **Framework** | Vitest + React Testing Library |
| **Config** | `vitest.config.ts` |
| **Setup** | `src/test/setup.ts` |
| **Pattern** | `**/*.{test,spec}.{ts,tsx}` |
| **Mocks** | `src/test/mocks/` |

---

## ⚙️ Règles de Génération

- [x] **Créer les dossiers manquants** automatiquement
- [x] **Utiliser les barrel exports** (`index.ts`)
- [x] **Générer les fichiers index** pour chaque dossier
- [x] **Enforcer TypeScript** (pas de .js)
- [ ] Ne pas supprimer les dossiers vides

---

## 🛠️ Stack Technologique

| Technologie | Version |
|-------------|---------|
| React | 19.2 |
| TypeScript | 5.9 |
| React Router | 7.9.6 |
| Zustand | 5.0.8 |
| TanStack Query | 5.90.7 |
| i18next | 25.6.2 |
| react-hook-form | 7.66.1 |
| Zod | 4.1.x |
| MUI | 7.3.5 |
| @peaksys/design-system | 8.42.0 |
| Vitest | 4.0.8 |
| Vite | 7.2.4 |

---

## ✅ Checklist de Validation

- [x] Guides de stack chargés
- [x] Summary lisible
- [x] JSON généré (`project-structure.json`)
- [x] Checklist générée (`project-structure-checklist.md`)
- [x] Structure cohérente avec les guides
- [x] Conventions de nommage définies
- [x] Alias de chemins configurés
- [x] Règles i18n définies
- [x] Règles de routing définies
- [x] Configuration de tests définie

---

*Généré par Stage 01 - Project Structure Spec Builder*  
*© 2025 ai-orchestrator-v4*
