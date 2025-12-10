# Project Structure Guide

## Overview

This guide defines the folder structure and file organization patterns for the enterprise React application.

## Root Structure

```
project-root/
├── public/                 # Static assets served directly
│   ├── configuration/      # Runtime configuration
│   └── media/              # Static media files
├── src/                    # Source code
│   ├── App.tsx             # Root application component
│   ├── main.tsx            # Application entry point
│   ├── vite-env.d.ts       # Vite type declarations
│   ├── components/         # Shared UI components
│   ├── features/           # Feature modules
│   ├── hooks/              # Shared custom hooks
│   ├── pages/              # Route page components
│   ├── routes/             # Routing configuration
│   ├── services/           # API services
│   ├── stores/             # Global state stores
│   ├── types/              # Shared TypeScript types
│   ├── utils/              # Utility functions
│   ├── locales/            # i18n translation files
│   └── __tests__/          # Integration tests
├── tests/                  # E2E and contract tests
├── vite.config.ts          # Vite configuration
├── tsconfig.json           # TypeScript configuration
├── vitest.config.ts        # Vitest configuration
└── package.json
```

## Feature Module Structure

Each feature should be self-contained:

```
src/features/[feature-name]/
├── components/             # Feature-specific components
│   ├── FeatureComponent.tsx
│   ├── FeatureComponent.test.tsx
│   └── FeatureComponent.styles.ts
├── hooks/                  # Feature-specific hooks
│   ├── useFeatureLogic.ts
│   └── useFeatureLogic.test.ts
├── stores/                 # Feature-specific Zustand stores
│   └── featureStore.ts
├── types/                  # Feature-specific types
│   └── feature.types.ts
├── api/                    # Feature API calls
│   └── featureApi.ts
├── utils/                  # Feature utilities
│   └── featureHelpers.ts
└── index.ts                # Public exports
```

## Component Organization

### Atomic Design Pattern

```
src/components/
├── atoms/                  # Basic building blocks
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.styles.ts
│   │   └── index.ts
│   └── ...
├── molecules/              # Combinations of atoms
│   ├── SearchInput/
│   └── ...
├── organisms/              # Complex UI sections
│   ├── Header/
│   └── ...
└── templates/              # Page layouts
    └── DashboardLayout/
```

## File Naming Conventions

### Components
```
ComponentName.tsx           # React component
ComponentName.test.tsx      # Unit tests
ComponentName.styles.ts     # Styled components / Emotion styles
ComponentName.types.ts      # Component-specific types (optional)
index.ts                    # Barrel export
```

### Hooks
```
useHookName.ts              # Custom hook
useHookName.test.ts         # Hook tests
```

### Stores
```
[domain]Store.ts            # Zustand store
[domain]Store.test.ts       # Store tests
```

### Types
```
[domain].types.ts           # Type definitions
[domain].schemas.ts         # Zod schemas
```

## Import Order Convention

```typescript
// 1. React and core libraries
import React, { useState, useCallback } from 'react';

// 2. Third-party libraries
import { useNavigate } from 'react-router';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';

// 3. Design system / UI components
import { Button, Box, Typography } from '@mui/material';
import { DataGrid } from '@peaksys/design-system';

// 4. Internal shared modules (absolute imports)
import { useAuth } from '@/hooks/useAuth';
import type { User } from '@/types/user.types';

// 5. Feature-specific imports (relative imports)
import { useFeatureStore } from '../stores/featureStore';
import { FeatureComponent } from './FeatureComponent';

// 6. Styles
import { StyledContainer } from './Component.styles';

// 7. Types (if separate)
import type { ComponentProps } from './Component.types';
```

## Barrel Exports Pattern

### index.ts in component folder:
```typescript
export { Button } from './Button';
export type { ButtonProps } from './Button.types';
```

### Feature index.ts:
```typescript
// Public API of the feature
export { FeatureComponent } from './components/FeatureComponent';
export { useFeatureLogic } from './hooks/useFeatureLogic';
export type { FeatureData } from './types/feature.types';
```

## Path Aliases

Configure in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@features/*": ["src/features/*"],
      "@hooks/*": ["src/hooks/*"],
      "@stores/*": ["src/stores/*"],
      "@types/*": ["src/types/*"],
      "@utils/*": ["src/utils/*"],
      "@services/*": ["src/services/*"]
    }
  }
}
```

## Configuration Files

```
project-root/
├── .env                    # Default environment variables
├── .env.development        # Development overrides
├── .env.production         # Production overrides
├── .env.test               # Test environment
├── vite.config.ts          # Vite build configuration
├── tsconfig.json           # TypeScript configuration
├── tsconfig.node.json      # Node-specific TS config
├── vitest.config.ts        # Test configuration
├── .eslintrc.cjs           # ESLint rules
├── .prettierrc             # Prettier formatting
└── .gitignore              # Git ignore patterns
```
