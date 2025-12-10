# Monorepo Guide

## Overview

This guide covers monorepo structure and patterns using npm workspaces or similar tools.

## Monorepo Structure

### Directory Layout

```
├── apps/
│   ├── cars-client/              # Main application
│   │   ├── src/
│   │   ├── public/
│   │   ├── package.json
│   │   └── vite.config.ts
│   ├── admin-portal/             # Admin application
│   │   ├── src/
│   │   └── package.json
│   └── storybook/                # Storybook instance
│       ├── .storybook/
│       └── package.json
├── packages/
│   ├── ui/                       # Shared UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── api-client/               # Shared API client
│   │   ├── src/
│   │   └── package.json
│   ├── config/                   # Shared configs
│   │   ├── eslint/
│   │   ├── typescript/
│   │   └── package.json
│   └── types/                    # Shared TypeScript types
│       ├── src/
│       └── package.json
├── package.json                  # Root package.json
├── tsconfig.base.json           # Base TypeScript config
└── turbo.json                   # Turborepo config (if using)
```

## Package Configuration

### Root package.json

```json
{
  "name": "cars-platform",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.9.0"
  }
}
```

### Shared Package Example

```json
// packages/ui/package.json
{
  "name": "@cars/ui",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": "./src/index.ts",
    "./components": "./src/components/index.ts",
    "./hooks": "./src/hooks/index.ts"
  },
  "scripts": {
    "build": "tsc",
    "lint": "eslint src",
    "test": "vitest run"
  },
  "peerDependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@cars/config": "workspace:*",
    "@types/react": "^19.0.0"
  }
}
```

### App Package Consuming Shared Packages

```json
// apps/cars-client/package.json
{
  "name": "@cars/client",
  "private": true,
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest run"
  },
  "dependencies": {
    "@cars/ui": "workspace:*",
    "@cars/api-client": "workspace:*",
    "@cars/types": "workspace:*",
    "react": "^19.2.0",
    "react-dom": "^19.2.0"
  },
  "devDependencies": {
    "@cars/config": "workspace:*"
  }
}
```

## Turborepo Configuration

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```

## Shared Configuration Packages

### TypeScript Base Config

```json
// packages/config/typescript/base.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true
  }
}

// packages/config/typescript/react.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "jsx": "react-jsx",
    "noEmit": true
  }
}
```

### App TypeScript Config

```json
// apps/cars-client/tsconfig.json
{
  "extends": "@cars/config/typescript/react.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@cars/ui": ["../../packages/ui/src"],
      "@cars/api-client": ["../../packages/api-client/src"],
      "@cars/types": ["../../packages/types/src"]
    }
  },
  "include": ["src"],
  "references": [
    { "path": "../../packages/ui" },
    { "path": "../../packages/api-client" },
    { "path": "../../packages/types" }
  ]
}
```

## Shared UI Components Package

```tsx
// packages/ui/src/components/Button/Button.tsx
import { forwardRef, ButtonHTMLAttributes } from 'react';
import { Button as MuiButton, ButtonProps as MuiButtonProps } from '@mui/material';

export interface ButtonProps extends MuiButtonProps {
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ isLoading, disabled, children, ...props }, ref) => {
    return (
      <MuiButton
        ref={ref}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading ? 'Loading...' : children}
      </MuiButton>
    );
  }
);

Button.displayName = 'Button';

// packages/ui/src/components/index.ts
export * from './Button';
export * from './Card';
export * from './DataTable';
export * from './Form';

// packages/ui/src/index.ts
export * from './components';
export * from './hooks';
export * from './theme';
```

## Shared API Client Package

```tsx
// packages/api-client/src/client.ts
export class ApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async get<T>(endpoint: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    return response.json();
  }

  async post<T>(endpoint: string, data: unknown): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// packages/api-client/src/hooks/index.ts
export { useCampaigns, useCreateCampaign } from './campaigns';
export { useProducts, useCreateProduct } from './products';

// packages/api-client/src/index.ts
export { ApiClient } from './client';
export * from './hooks';
export * from './types';
```

## Shared Types Package

```tsx
// packages/types/src/campaign.ts
export interface Campaign {
  id: string;
  name: string;
  status: CampaignStatus;
  budget: number;
  startDate: string;
  endDate?: string;
  createdAt: string;
  updatedAt: string;
}

export type CampaignStatus = 'draft' | 'active' | 'paused' | 'completed';

export interface CampaignFilters {
  status?: CampaignStatus;
  search?: string;
  page?: number;
  limit?: number;
}

// packages/types/src/index.ts
export * from './campaign';
export * from './product';
export * from './user';
export * from './api';
```

## Using Shared Packages in Apps

```tsx
// apps/cars-client/src/features/campaigns/CampaignsList.tsx
import { Button, DataTable, Card } from '@cars/ui';
import { useCampaigns, useCreateCampaign } from '@cars/api-client';
import type { Campaign, CampaignFilters } from '@cars/types';

export function CampaignsList() {
  const [filters, setFilters] = useState<CampaignFilters>({});
  const { data, isLoading } = useCampaigns(filters);
  const createMutation = useCreateCampaign();

  return (
    <Card>
      <Button onClick={() => createMutation.mutate(newCampaign)}>
        Create Campaign
      </Button>
      <DataTable
        data={data?.items ?? []}
        loading={isLoading}
        columns={columns}
      />
    </Card>
  );
}
```

## Vite Configuration for Monorepo

```ts
// apps/cars-client/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';
import path from 'path';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      // Explicitly resolve workspace packages to source
      '@cars/ui': path.resolve(__dirname, '../../packages/ui/src'),
      '@cars/api-client': path.resolve(__dirname, '../../packages/api-client/src'),
      '@cars/types': path.resolve(__dirname, '../../packages/types/src'),
    },
  },
  
  // Optimize deps for monorepo packages
  optimizeDeps: {
    include: ['@cars/ui', '@cars/api-client'],
  },
  
  server: {
    port: 3000,
    // Watch for changes in workspace packages
    watch: {
      ignored: ['!**/node_modules/@cars/**'],
    },
  },
});
```

## Best Practices

1. **Use workspace protocol** (`workspace:*`) for internal dependencies
2. **Keep shared packages focused** - one responsibility per package
3. **Export from index files** for clean imports
4. **Use path aliases** for better developer experience
5. **Configure TypeScript references** for type checking
6. **Use Turborepo** for efficient builds and caching
7. **Share configs** (ESLint, TypeScript) via packages
8. **Version shared packages** for tracking changes
9. **Document package APIs** with JSDoc or README
10. **Test shared packages** independently
