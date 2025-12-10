# Build and Tooling Guide

## Overview

This guide covers Vite configuration and build tooling for the enterprise stack.

## Vite Configuration

### Basic Configuration

```ts
// vite.config.ts
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';
import svgr from 'vite-plugin-svgr';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');

  return {
    plugins: [
      react({
        // Enable React Fast Refresh
        fastRefresh: true,
      }),
      tsconfigPaths(), // Support path aliases from tsconfig
      svgr(), // Import SVGs as React components
    ],

    // Development server configuration
    server: {
      port: 3000,
      open: true,
      proxy: {
        '/api': {
          target: env.VITE_API_URL || 'http://localhost:8080',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ''),
        },
      },
    },

    // Build configuration
    build: {
      outDir: 'dist',
      sourcemap: mode === 'development',
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: mode === 'production',
        },
      },
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ['react', 'react-dom'],
            router: ['react-router'],
            query: ['@tanstack/react-query'],
            ui: ['@mui/material', '@emotion/react', '@emotion/styled'],
          },
        },
      },
    },

    // Path resolution
    resolve: {
      alias: {
        '@': '/src',
        '@components': '/src/components',
        '@features': '/src/features',
        '@core': '/src/core',
        '@utils': '/src/utils',
      },
    },

    // Define global constants
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    },

    // Optimize dependencies
    optimizeDeps: {
      include: ['@mui/material', '@emotion/react', '@emotion/styled'],
    },
  };
});
```

## TypeScript Configuration

### Base tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@features/*": ["src/features/*"],
      "@core/*": ["src/core/*"],
      "@utils/*": ["src/utils/*"]
    },
    
    "types": ["vite/client", "vitest/globals"]
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Node tsconfig

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "strict": true
  },
  "include": ["vite.config.ts", "vitest.config.ts"]
}
```

## Environment Variables

### .env Files Structure

```bash
# .env (default values, committed)
VITE_APP_NAME=CARS Client
VITE_API_VERSION=v1

# .env.local (local overrides, not committed)
VITE_API_URL=http://localhost:8080

# .env.development
VITE_API_URL=https://dev-api.example.com
VITE_DEBUG=true

# .env.production
VITE_API_URL=https://api.example.com
VITE_DEBUG=false
```

### Type-Safe Environment Variables

```ts
// src/core/env.ts
import { z } from 'zod';

const envSchema = z.object({
  VITE_APP_NAME: z.string().default('CARS Client'),
  VITE_API_URL: z.string().url(),
  VITE_API_VERSION: z.string().default('v1'),
  VITE_DEBUG: z.string().transform((s) => s === 'true').default('false'),
  MODE: z.enum(['development', 'production', 'test']),
});

export const env = envSchema.parse({
  VITE_APP_NAME: import.meta.env.VITE_APP_NAME,
  VITE_API_URL: import.meta.env.VITE_API_URL,
  VITE_API_VERSION: import.meta.env.VITE_API_VERSION,
  VITE_DEBUG: import.meta.env.VITE_DEBUG,
  MODE: import.meta.env.MODE,
});

// Type declaration for import.meta.env
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_NAME: string;
  readonly VITE_API_URL: string;
  readonly VITE_API_VERSION: string;
  readonly VITE_DEBUG: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## ESLint Configuration

```js
// eslint.config.js
import js from '@eslint/js';
import globals from 'globals';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  { ignores: ['dist', 'node_modules'] },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      ecmaVersion: 2022,
      globals: globals.browser,
    },
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/consistent-type-imports': 'error',
      '@typescript-eslint/no-explicit-any': 'warn',
    },
  }
);
```

## Prettier Configuration

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

## Package.json Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "build:analyze": "vite build --mode analyze",
    "preview": "vite preview",
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "lint": "eslint src --ext ts,tsx",
    "lint:fix": "eslint src --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css,json}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,css,json}\"",
    "type-check": "tsc --noEmit",
    "prepare": "husky"
  }
}
```

## Git Hooks with Husky

```bash
# Install
npm install -D husky lint-staged
npx husky init
```

```bash
# .husky/pre-commit
npx lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  }
}
```

## Bundle Analysis

```ts
// vite.config.ts - add analyzer plugin
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig(({ mode }) => ({
  plugins: [
    // ... other plugins
    mode === 'analyze' &&
      visualizer({
        filename: 'dist/stats.html',
        open: true,
        gzipSize: true,
        brotliSize: true,
      }),
  ].filter(Boolean),
}));
```

## CI/CD Pipeline (GitHub Actions Example)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Type check
        run: npm run type-check
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm run test:coverage
      
      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
```

## Performance Optimization

### Code Splitting

```tsx
import { lazy, Suspense } from 'react';

// Route-based code splitting
const CampaignsPage = lazy(() => import('./features/campaigns/pages/CampaignsPage'));
const SettingsPage = lazy(() => import('./features/settings/pages/SettingsPage'));

// Component-based code splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/campaigns" element={<CampaignsPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </Routes>
    </Suspense>
  );
}
```

### Asset Optimization

```ts
// vite.config.ts
export default defineConfig({
  build: {
    // Asset inlining threshold
    assetsInlineLimit: 4096, // 4kb
    
    // Chunk size warning
    chunkSizeWarningLimit: 500,
    
    rollupOptions: {
      output: {
        // Asset file naming
        assetFileNames: 'assets/[name]-[hash][extname]',
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
      },
    },
  },
});
```

## Best Practices

1. **Use path aliases** for cleaner imports
2. **Split vendor chunks** for better caching
3. **Enable source maps** only in development
4. **Validate environment variables** at build time
5. **Use code splitting** for large components/pages
6. **Configure proper caching headers** in production
7. **Run type checking before build** in CI/CD
