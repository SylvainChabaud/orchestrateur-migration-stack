# Stack Guides Summary

## 📋 Vue d'ensemble

Ce document résume les 15 guides de stack générés pour la migration du code legacy vers la nouvelle stack enterprise. Ces guides servent de référence pour les phases d'analyse, d'interprétation et de génération de code.

---

## 🎯 Stack Cible

| Technologie | Version | Rôle |
|-------------|---------|------|
| React | 19.2 | Framework UI |
| TypeScript | 5.9 | Typage statique |
| React Router | 7.9.6 | Routing SPA |
| Zustand | 5.0.8 | State management |
| TanStack Query | 5.90.7 | Data fetching & cache |
| i18next | 25.6.2 | Internationalisation |
| react-hook-form | 7.66.1 | Gestion formulaires |
| Zod | 4.1.x | Validation schemas |
| MUI | 7.3.5 | Design system base |
| @peaksys/design-system | 8.42.0 | Design system entreprise |
| Vitest | 4.0.8 | Testing framework |
| Vite | 7.2.4 | Build tool |

---

## 📚 Index des Guides

### 1. guide.stack.md
**Objectif:** Vue d'ensemble complète de la stack technologique
- Liste de toutes les technologies et versions
- Rôle de chaque outil dans l'architecture
- Liens vers la documentation officielle

### 2. guide.structure.md
**Objectif:** Structure de projet et organisation des fichiers
- Architecture feature-based
- Conventions de nommage
- Organisation des dossiers (`src/features/`, `src/core/`, `src/components/`)

### 3. guide.ui-components.md
**Objectif:** Composants UI avec MUI 7 + Peaksys Design System
- Configuration du ThemeProvider
- Utilisation des composants MUI
- Intégration du design system entreprise
- Patterns de composition

### 4. guide.styles.md
**Objectif:** Styling avec Emotion et MUI
- Utilisation du prop `sx`
- API `styled()` pour composants custom
- Gestion des thèmes
- Variables CSS et tokens

### 5. guide.routing.md
**Objectif:** Navigation avec React Router 7
- Migration de `useHistory` → `useNavigate`
- `useParams`, `useLocation`, `useSearchParams`
- Routes protégées et layouts
- Navigation programmatique

### 6. guide.i18n.md
**Objectif:** Internationalisation avec i18next
- Configuration de i18next
- Hook `useTranslation`
- Composant `<Trans>`
- Gestion des namespaces et pluriels

### 7. guide.state-management.md
**Objectif:** État global avec Zustand + TanStack Query
- Création de stores Zustand
- Selectors et middleware
- useQuery / useMutation patterns
- Invalidation et cache

### 8. guide.api-client.md
**Objectif:** Client API et data fetching
- Configuration du client HTTP
- Query keys factory pattern
- Gestion des erreurs API
- Pagination et infinite queries

### 9. guide.forms-validation.md
**Objectif:** Formulaires avec react-hook-form + Zod
- `useForm` avec `zodResolver`
- Validation synchrone et asynchrone
- Field arrays et wizard forms
- Intégration avec MUI

### 10. guide.tests.md
**Objectif:** Tests avec Vitest + React Testing Library
- Configuration Vitest
- Custom render avec providers
- Testing des hooks React Query
- Mocking avec MSW

### 11. guide.build-and-tooling.md
**Objectif:** Configuration Vite et outillage
- `vite.config.ts` patterns
- Variables d'environnement type-safe
- ESLint et Prettier
- Code splitting et optimisation

### 12. guide.auth.md
**Objectif:** Authentification OIDC
- Configuration OIDC
- Store d'authentification Zustand
- Routes protégées avec rôles/permissions
- Token refresh automatique

### 13. guide.performance-accessibility.md
**Objectif:** Performance et accessibilité
- Memoization (`memo`, `useMemo`, `useCallback`)
- Lazy loading et virtualisation
- ARIA labels et keyboard navigation
- Focus management et live regions

### 14. guide.error-handling.md
**Objectif:** Gestion des erreurs et logging
- Error Boundaries React
- Classes d'erreur custom (ApiError, NetworkError)
- Toast notifications
- Logging service

### 15. guide.monorepo.md
**Objectif:** Structure monorepo (si applicable)
- Organisation des packages
- Shared dependencies
- Build et deployment

---

## 🔄 Correspondances Legacy → New Stack

| Pattern Legacy | Nouveau Pattern | Guide de référence |
|----------------|-----------------|-------------------|
| `useHistory()` | `useNavigate()` | guide.routing.md |
| `useI18n()` custom | `useTranslation()` | guide.i18n.md |
| Material-UI v4 Grid | MUI 7 Grid2 / Box | guide.ui-components.md |
| `makeStyles()` | `sx` prop / `styled()` | guide.styles.md |
| Redux / Context | Zustand stores | guide.state-management.md |
| Fetch / Axios direct | TanStack Query hooks | guide.api-client.md |
| PropTypes | TypeScript interfaces | guide.stack.md |
| Jest + Enzyme | Vitest + RTL | guide.tests.md |

---

## 🏗️ Architecture Recommandée

```
src/
├── core/                    # Infrastructure partagée
│   ├── api/                 # Client API, queryClient
│   ├── auth/                # Auth store, guards
│   ├── errors/              # Error boundaries, handlers
│   ├── i18n/                # Configuration i18next
│   └── theme/               # MUI theme configuration
│
├── features/                # Modules métier
│   └── campaigns/           # Exemple: CampaignsDetail
│       ├── api/             # API calls + types
│       ├── components/      # Composants UI feature
│       ├── hooks/           # Custom hooks (useQuery wrappers)
│       ├── pages/           # Page components
│       ├── stores/          # Zustand stores locaux
│       └── types/           # TypeScript types
│
├── components/              # Composants UI partagés
│   ├── common/              # Buttons, Cards, etc.
│   └── layout/              # Header, Sidebar, etc.
│
├── routes/                  # Configuration routing
├── utils/                   # Helpers et utilitaires
└── test/                    # Setup tests et mocks
```

---

## 📝 Conventions de Code

### Nommage
- **Composants:** PascalCase (`CampaignCard.tsx`)
- **Hooks:** camelCase avec préfixe `use` (`useCampaigns.ts`)
- **Stores:** camelCase avec suffixe `Store` (`authStore.ts`)
- **Types:** PascalCase (`Campaign`, `CampaignFilters`)

### Imports
```tsx
// 1. React et libraries externes
import { useState, useCallback } from 'react';
import { useNavigate } from 'react-router';

// 2. Components MUI / Design System
import { Box, Typography, Button } from '@mui/material';

// 3. Core / Features internes
import { useAuth } from '@/core/auth';
import { useCampaigns } from '../hooks/useCampaigns';

// 4. Types
import type { Campaign } from '../types';
```

### Structure Composant
```tsx
// Types
interface Props {
  campaign: Campaign;
  onEdit?: (id: string) => void;
}

// Component
export function CampaignCard({ campaign, onEdit }: Props) {
  // 1. Hooks (router, i18n, stores)
  const { t } = useTranslation('campaigns');
  const navigate = useNavigate();
  
  // 2. State local
  const [isOpen, setIsOpen] = useState(false);
  
  // 3. Queries / Mutations
  const { mutate } = useUpdateCampaign();
  
  // 4. Callbacks
  const handleEdit = useCallback(() => {
    onEdit?.(campaign.id);
  }, [campaign.id, onEdit]);
  
  // 5. Render
  return (
    <Card>...</Card>
  );
}
```

---

## ✅ Checklist Migration

Pour chaque composant legacy à migrer :

- [ ] Identifier les imports à transformer (voir tableau correspondances)
- [ ] Convertir en TypeScript avec interfaces
- [ ] Remplacer `useHistory` par `useNavigate`
- [ ] Remplacer `useI18n` custom par `useTranslation`
- [ ] Migrer styles vers `sx` prop ou `styled()`
- [ ] Créer hooks React Query pour les appels API
- [ ] Ajouter Error Boundaries si nécessaire
- [ ] Écrire tests avec Vitest + RTL
- [ ] Valider accessibilité (ARIA, keyboard nav)

---

## 📖 Utilisation par les Stages

| Phase | Guides Principaux |
|-------|-------------------|
| **Phase 1 - Analysis** | guide.stack.md, guide.structure.md |
| **Phase 2 - Interpretation** | Tous les guides selon l'inventaire |
| **Phase 3 - Generation** | guide.ui-components.md, guide.routing.md, guide.state-management.md, guide.tests.md |
| **Phase 4 - Validation** | guide.tests.md, guide.performance-accessibility.md |

---

*Généré par Stage 00 - Stack Guides Builder*  
*© 2025 ai-orchestrator-v4*
