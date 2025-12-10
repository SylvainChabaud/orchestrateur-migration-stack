# Testing Guide

## Overview

This guide covers testing with Vitest and React Testing Library for the enterprise stack.

## Vitest Setup

### Configuration

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
      ],
    },
  },
});
```

### Setup File

```ts
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));
```

## React Testing Library

### Basic Component Test

```tsx
// src/features/campaigns/components/CampaignCard.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CampaignCard } from './CampaignCard';

describe('CampaignCard', () => {
  const mockCampaign = {
    id: '1',
    name: 'Test Campaign',
    status: 'active' as const,
    budget: 1000,
    startDate: '2024-01-01',
  };

  it('renders campaign name', () => {
    render(<CampaignCard campaign={mockCampaign} />);
    
    expect(screen.getByText('Test Campaign')).toBeInTheDocument();
  });

  it('displays status badge with correct color', () => {
    render(<CampaignCard campaign={mockCampaign} />);
    
    const badge = screen.getByText('active');
    expect(badge).toHaveClass('MuiChip-colorSuccess');
  });

  it('calls onEdit when edit button is clicked', async () => {
    const user = userEvent.setup();
    const handleEdit = vi.fn();
    
    render(<CampaignCard campaign={mockCampaign} onEdit={handleEdit} />);
    
    await user.click(screen.getByRole('button', { name: /edit/i }));
    
    expect(handleEdit).toHaveBeenCalledWith('1');
  });

  it('formats budget correctly', () => {
    render(<CampaignCard campaign={mockCampaign} />);
    
    expect(screen.getByText('$1,000.00')).toBeInTheDocument();
  });
});
```

## Test Utilities

### Custom Render with Providers

```tsx
// src/test/utils.tsx
import { ReactElement, ReactNode } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router';
import { ThemeProvider } from '@mui/material/styles';
import { I18nextProvider } from 'react-i18next';
import i18n from './i18n-test';
import { theme } from '@/core/theme';

interface WrapperProps {
  children: ReactNode;
}

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        gcTime: 0,
      },
      mutations: {
        retry: false,
      },
    },
  });
}

export function createWrapper() {
  const queryClient = createTestQueryClient();

  return function Wrapper({ children }: WrapperProps) {
    return (
      <QueryClientProvider client={queryClient}>
        <BrowserRouter>
          <ThemeProvider theme={theme}>
            <I18nextProvider i18n={i18n}>
              {children}
            </I18nextProvider>
          </ThemeProvider>
        </BrowserRouter>
      </QueryClientProvider>
    );
  };
}

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  route?: string;
}

export function renderWithProviders(
  ui: ReactElement,
  options?: CustomRenderOptions
) {
  const { route = '/', ...renderOptions } = options ?? {};

  window.history.pushState({}, 'Test page', route);

  return {
    ...render(ui, { wrapper: createWrapper(), ...renderOptions }),
  };
}

export * from '@testing-library/react';
export { renderWithProviders as render };
```

### Test i18n Configuration

```ts
// src/test/i18n-test.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

i18n.use(initReactI18next).init({
  lng: 'en',
  fallbackLng: 'en',
  ns: ['common', 'campaigns'],
  defaultNS: 'common',
  resources: {
    en: {
      common: {
        save: 'Save',
        cancel: 'Cancel',
        delete: 'Delete',
        loading: 'Loading...',
      },
      campaigns: {
        title: 'Campaigns',
        create: 'Create Campaign',
        edit: 'Edit Campaign',
      },
    },
  },
  interpolation: {
    escapeValue: false,
  },
});

export default i18n;
```

## Testing Hooks

### Testing Custom Hooks with React Query

```tsx
// src/features/campaigns/hooks/useCampaigns.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { useCampaigns, useCreateCampaign } from './useCampaigns';
import { createWrapper } from '@/test/utils';
import { campaignsApi } from '../api/campaignsApi';

vi.mock('../api/campaignsApi');

describe('useCampaigns', () => {
  const mockCampaigns = {
    data: [
      { id: '1', name: 'Campaign 1', status: 'active' },
      { id: '2', name: 'Campaign 2', status: 'draft' },
    ],
    total: 2,
    page: 1,
    limit: 20,
    totalPages: 1,
  };

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('fetches campaigns successfully', async () => {
    vi.mocked(campaignsApi.getAll).mockResolvedValue(mockCampaigns);

    const { result } = renderHook(() => useCampaigns(), {
      wrapper: createWrapper(),
    });

    expect(result.current.isPending).toBe(true);

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(mockCampaigns);
    expect(campaignsApi.getAll).toHaveBeenCalledTimes(1);
  });

  it('handles error state', async () => {
    const error = new Error('Failed to fetch');
    vi.mocked(campaignsApi.getAll).mockRejectedValue(error);

    const { result } = renderHook(() => useCampaigns(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error).toBe(error);
  });

  it('passes filters to API', async () => {
    vi.mocked(campaignsApi.getAll).mockResolvedValue(mockCampaigns);
    const filters = { status: 'active', page: 2 };

    renderHook(() => useCampaigns(filters), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(campaignsApi.getAll).toHaveBeenCalledWith(filters);
    });
  });
});

describe('useCreateCampaign', () => {
  it('creates campaign and invalidates cache', async () => {
    const newCampaign = { id: '3', name: 'New Campaign', status: 'draft' };
    vi.mocked(campaignsApi.create).mockResolvedValue(newCampaign);

    const { result } = renderHook(() => useCreateCampaign(), {
      wrapper: createWrapper(),
    });

    result.current.mutate({ name: 'New Campaign', status: 'draft', budget: 500 });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(newCampaign);
  });
});
```

## Testing Forms

```tsx
// src/features/campaigns/components/CampaignForm.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@/test/utils';
import userEvent from '@testing-library/user-event';
import { CampaignForm } from './CampaignForm';

describe('CampaignForm', () => {
  const user = userEvent.setup();

  it('submits form with valid data', async () => {
    const handleSubmit = vi.fn();
    render(<CampaignForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/name/i), 'Test Campaign');
    await user.type(screen.getByLabelText(/budget/i), '1000');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        name: 'Test Campaign',
        budget: 1000,
      });
    });
  });

  it('shows validation errors for empty required fields', async () => {
    render(<CampaignForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(screen.getByText(/name is required/i)).toBeInTheDocument();
    });
  });

  it('shows error for invalid email format', async () => {
    render(<CampaignForm onSubmit={vi.fn()} />);

    await user.type(screen.getByLabelText(/email/i), 'invalid-email');
    await user.tab(); // Trigger blur validation

    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
    });
  });

  it('disables submit button while submitting', async () => {
    const handleSubmit = vi.fn(() => new Promise((resolve) => setTimeout(resolve, 100)));
    render(<CampaignForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/name/i), 'Test');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled();
  });
});
```

## Mocking

### Mocking Modules

```tsx
// Mocking entire module
vi.mock('@/features/campaigns/api/campaignsApi', () => ({
  campaignsApi: {
    getAll: vi.fn(),
    getById: vi.fn(),
    create: vi.fn(),
    update: vi.fn(),
    delete: vi.fn(),
  },
}));

// Partial mock
vi.mock('@/utils/formatters', async (importOriginal) => {
  const actual = await importOriginal<typeof import('@/utils/formatters')>();
  return {
    ...actual,
    formatDate: vi.fn(() => '2024-01-01'),
  };
});
```

### Mocking API Responses with MSW

```tsx
// src/test/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/campaigns', () => {
    return HttpResponse.json({
      data: [
        { id: '1', name: 'Campaign 1', status: 'active' },
      ],
      total: 1,
      page: 1,
    });
  }),

  http.post('/api/campaigns', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({
      id: '2',
      ...body,
      createdAt: new Date().toISOString(),
    }, { status: 201 });
  }),

  http.delete('/api/campaigns/:id', ({ params }) => {
    return new HttpResponse(null, { status: 204 });
  }),
];

// src/test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/test/setup.ts
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Testing Navigation

```tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@/test/utils';
import userEvent from '@testing-library/user-event';
import { CampaignDetail } from './CampaignDetail';
import { useNavigate } from 'react-router';

vi.mock('react-router', async (importOriginal) => {
  const actual = await importOriginal<typeof import('react-router')>();
  return {
    ...actual,
    useNavigate: vi.fn(),
    useParams: () => ({ id: '1' }),
  };
});

describe('CampaignDetail navigation', () => {
  it('navigates back to list when back button is clicked', async () => {
    const mockNavigate = vi.fn();
    vi.mocked(useNavigate).mockReturnValue(mockNavigate);

    const user = userEvent.setup();
    render(<CampaignDetail />);

    await user.click(screen.getByRole('button', { name: /back/i }));

    expect(mockNavigate).toHaveBeenCalledWith('/campaigns');
  });
});
```

## Snapshot Testing

```tsx
import { describe, it, expect } from 'vitest';
import { render } from '@/test/utils';
import { CampaignCard } from './CampaignCard';

describe('CampaignCard snapshots', () => {
  it('matches snapshot for active campaign', () => {
    const { container } = render(
      <CampaignCard 
        campaign={{ id: '1', name: 'Test', status: 'active', budget: 1000 }} 
      />
    );

    expect(container).toMatchSnapshot();
  });
});
```

## Best Practices

1. **Use `userEvent` over `fireEvent`** for realistic user interactions
2. **Query by accessible roles** (getByRole) when possible
3. **Wait for async operations** with `waitFor` or `findBy*` queries
4. **Mock at the API layer** not component internals
5. **Test behavior not implementation** - focus on what users see
6. **Use custom render wrapper** for consistent provider setup
7. **Keep tests isolated** - no shared state between tests
