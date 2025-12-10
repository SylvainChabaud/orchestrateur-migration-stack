# Stack Guide - Enterprise React Stack

## Overview

This guide defines the target technology stack for migrating legacy React applications to a modern enterprise architecture.

## Target Stack Summary

| Category | Technology | Version |
|----------|-----------|---------|
| UI Framework | React | 19.2 |
| Language | TypeScript | 5.9 |
| Routing | React Router | 7.9.6 |
| Global State | Zustand | 5.0.8 |
| Server State | TanStack React Query | 5.90.7 |
| Design System | @peaksys/design-system | 8.42.0 |
| UI Components | MUI (Material UI) | 7.3.5 |
| Forms | react-hook-form | 7.66.1 |
| Validation | Zod | 4.1.x |
| i18n | i18next / react-i18next | 25.6.2 |
| Testing | Vitest + React Testing Library | 4.0.8 / 16.3.0 |
| Build Tool | Vite | 7.2.4 |
| Styling | Emotion | 11.14.0 |

## Core Principles

### 1. Type Safety First
- Use TypeScript strict mode
- Define interfaces for all data structures
- Use Zod for runtime validation
- Leverage type inference where possible

### 2. Server State vs Client State
- **Server State**: Data fetched from APIs → TanStack Query
- **Client State**: UI state, user preferences → Zustand
- Never duplicate server state in client stores

### 3. Component Architecture
- Functional components only (no class components)
- Custom hooks for reusable logic
- Composition over inheritance
- Atomic design principles (atoms, molecules, organisms)

### 4. Performance by Default
- Code splitting with React.lazy
- Virtualization for long lists
- Memoization with useMemo/useCallback
- Optimistic updates with React Query

## Migration Path

### From Legacy Patterns
- `useHistory()` → `useNavigate()` (React Router 7)
- Redux → Zustand (simpler, less boilerplate)
- Class components → Functional components with hooks
- Axios → React Query mutations
- CSS-in-JS (styled-components) → Emotion + MUI sx prop
- Jest → Vitest (faster, ESM native)

## File Structure

```
src/
├── components/           # Shared UI components
│   ├── atoms/
│   ├── molecules/
│   └── organisms/
├── features/             # Feature-based modules
│   └── [feature]/
│       ├── components/
│       ├── hooks/
│       ├── stores/
│       ├── types/
│       └── index.ts
├── hooks/                # Shared custom hooks
├── stores/               # Global Zustand stores
├── services/             # API clients
├── types/                # Shared TypeScript types
├── utils/                # Utility functions
└── pages/                # Route pages
```

## Key Dependencies

```json
{
  "dependencies": {
    "react": "^19.2.0",
    "react-dom": "^19.2.0",
    "react-router": "^7.9.6",
    "zustand": "^5.0.8",
    "@tanstack/react-query": "^5.90.7",
    "@peaksys/design-system": "^8.42.0",
    "@mui/material": "^7.3.5",
    "@emotion/react": "^11.14.0",
    "@emotion/styled": "^11.14.0",
    "react-hook-form": "^7.66.1",
    "@hookform/resolvers": "^3.x",
    "zod": "^4.1.0",
    "i18next": "^25.6.2",
    "react-i18next": "^15.x"
  },
  "devDependencies": {
    "typescript": "^5.9.0",
    "vite": "^7.2.4",
    "vitest": "^4.0.8",
    "@testing-library/react": "^16.3.0",
    "@testing-library/user-event": "^14.x"
  }
}
```
