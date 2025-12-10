# State Management Guide

## Overview

This guide covers Zustand for client state and TanStack React Query for server state management.

## Zustand for Client State

### Basic Store

```tsx
// src/stores/uiStore.ts
import { create } from 'zustand';

interface UIState {
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  theme: 'light',
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setTheme: (theme) => set({ theme }),
}));
```

### Using the Store

```tsx
function Sidebar() {
  // Subscribe to specific slice
  const sidebarOpen = useUIStore((state) => state.sidebarOpen);
  const toggleSidebar = useUIStore((state) => state.toggleSidebar);

  return (
    <Drawer open={sidebarOpen} onClose={toggleSidebar}>
      {/* sidebar content */}
    </Drawer>
  );
}
```

### Store with Computed Values

```tsx
interface CartState {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartState>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ 
    items: [...state.items, item] 
  })),
  removeItem: (id) => set((state) => ({ 
    items: state.items.filter(item => item.id !== id) 
  })),
  clearCart: () => set({ items: [] }),
}));

// Computed selector (outside the store)
export const useCartTotal = () => useCartStore(
  (state) => state.items.reduce((sum, item) => sum + item.price, 0)
);

export const useCartItemCount = () => useCartStore(
  (state) => state.items.length
);
```

### Store with Middleware

```tsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

interface UserPreferences {
  language: string;
  notifications: boolean;
  setLanguage: (lang: string) => void;
  toggleNotifications: () => void;
}

export const usePreferencesStore = create<UserPreferences>()(
  devtools(
    persist(
      (set) => ({
        language: 'en',
        notifications: true,
        setLanguage: (language) => set({ language }),
        toggleNotifications: () => set((state) => ({ 
          notifications: !state.notifications 
        })),
      }),
      {
        name: 'user-preferences', // localStorage key
      }
    ),
    { name: 'PreferencesStore' } // devtools name
  )
);
```

### Immer Middleware for Complex Updates

```tsx
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface FormState {
  formData: {
    name: string;
    address: {
      street: string;
      city: string;
    };
    contacts: Array<{ email: string }>;
  };
  updateName: (name: string) => void;
  updateCity: (city: string) => void;
  addContact: (email: string) => void;
}

export const useFormStore = create<FormState>()(
  immer((set) => ({
    formData: {
      name: '',
      address: { street: '', city: '' },
      contacts: [],
    },
    updateName: (name) => set((state) => {
      state.formData.name = name;
    }),
    updateCity: (city) => set((state) => {
      state.formData.address.city = city;
    }),
    addContact: (email) => set((state) => {
      state.formData.contacts.push({ email });
    }),
  }))
);
```

## TanStack React Query for Server State

### Query Client Setup

```tsx
// src/services/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 30,   // 30 minutes (was cacheTime)
      retry: 1,
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

### Provider Setup

```tsx
// src/App.tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/services/queryClient';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <AppContent />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### Basic Query

```tsx
import { useQuery } from '@tanstack/react-query';
import { campaignsApi } from '@/services/api/campaigns';

function CampaignsList() {
  const { 
    data: campaigns, 
    isLoading, 
    isError, 
    error,
    refetch 
  } = useQuery({
    queryKey: ['campaigns'],
    queryFn: campaignsApi.getAll,
  });

  if (isLoading) return <LoadingSpinner />;
  if (isError) return <ErrorMessage error={error} />;

  return (
    <div>
      {campaigns.map(campaign => (
        <CampaignCard key={campaign.id} campaign={campaign} />
      ))}
    </div>
  );
}
```

### Query with Parameters

```tsx
function CampaignDetail({ id }: { id: string }) {
  const { data: campaign, isLoading } = useQuery({
    queryKey: ['campaign', id],
    queryFn: () => campaignsApi.getById(id),
    enabled: !!id, // Only fetch if id exists
  });

  if (isLoading) return <LoadingSpinner />;
  
  return <div>{campaign?.name}</div>;
}
```

### Query with Filters

```tsx
function FilteredCampaigns() {
  const [filters, setFilters] = useState({
    status: 'active',
    page: 1,
    pageSize: 10,
  });

  const { data, isLoading } = useQuery({
    queryKey: ['campaigns', filters],
    queryFn: () => campaignsApi.getFiltered(filters),
    keepPreviousData: true, // Smooth pagination
  });

  return (
    <div>
      <FilterControls filters={filters} onChange={setFilters} />
      <CampaignsTable data={data?.items} loading={isLoading} />
      <Pagination 
        page={filters.page} 
        total={data?.total}
        onChange={(page) => setFilters(f => ({ ...f, page }))}
      />
    </div>
  );
}
```

### Mutations

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateCampaignForm() {
  const queryClient = useQueryClient();

  const createMutation = useMutation({
    mutationFn: campaignsApi.create,
    onSuccess: (newCampaign) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['campaigns'] });
      
      // Or update cache directly
      queryClient.setQueryData(['campaign', newCampaign.id], newCampaign);
      
      toast.success('Campaign created!');
    },
    onError: (error) => {
      toast.error(`Failed to create: ${error.message}`);
    },
  });

  const handleSubmit = (data: CampaignFormData) => {
    createMutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <Button 
        type="submit" 
        loading={createMutation.isPending}
        disabled={createMutation.isPending}
      >
        Create Campaign
      </Button>
    </form>
  );
}
```

### Optimistic Updates

```tsx
function CampaignStatus({ campaign }: { campaign: Campaign }) {
  const queryClient = useQueryClient();

  const toggleMutation = useMutation({
    mutationFn: (newStatus: string) => 
      campaignsApi.updateStatus(campaign.id, newStatus),
    
    onMutate: async (newStatus) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['campaign', campaign.id] });
      
      // Snapshot previous value
      const previousCampaign = queryClient.getQueryData(['campaign', campaign.id]);
      
      // Optimistically update
      queryClient.setQueryData(['campaign', campaign.id], (old: Campaign) => ({
        ...old,
        status: newStatus,
      }));
      
      return { previousCampaign };
    },
    
    onError: (err, newStatus, context) => {
      // Rollback on error
      queryClient.setQueryData(
        ['campaign', campaign.id], 
        context?.previousCampaign
      );
    },
    
    onSettled: () => {
      // Refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['campaign', campaign.id] });
    },
  });

  return (
    <Switch 
      checked={campaign.status === 'active'}
      onChange={() => toggleMutation.mutate(
        campaign.status === 'active' ? 'paused' : 'active'
      )}
    />
  );
}
```

## Combining Zustand and React Query

### Pattern: UI State in Zustand, Server State in Query

```tsx
// Zustand for UI filters
const useFiltersStore = create<FiltersState>((set) => ({
  status: 'all',
  search: '',
  dateRange: null,
  setStatus: (status) => set({ status }),
  setSearch: (search) => set({ search }),
  setDateRange: (dateRange) => set({ dateRange }),
}));

// React Query uses Zustand filters
function CampaignsPage() {
  const filters = useFiltersStore();
  
  const { data: campaigns } = useQuery({
    queryKey: ['campaigns', filters],
    queryFn: () => campaignsApi.getFiltered(filters),
  });

  return (
    <div>
      <CampaignFilters />
      <CampaignsList campaigns={campaigns} />
    </div>
  );
}
```

### Custom Hook Pattern

```tsx
// src/features/campaigns/hooks/useCampaigns.ts
export function useCampaigns() {
  const filters = useFiltersStore();
  
  return useQuery({
    queryKey: ['campaigns', filters],
    queryFn: () => campaignsApi.getFiltered(filters),
    select: (data) => data.items, // Transform response
  });
}

export function useCampaign(id: string) {
  return useQuery({
    queryKey: ['campaign', id],
    queryFn: () => campaignsApi.getById(id),
    enabled: !!id,
  });
}

export function useCreateCampaign() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: campaignsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['campaigns'] });
    },
  });
}
```

## Best Practices

### 1. Separate Server State from Client State

- **Server State** → React Query (campaigns, users, data from API)
- **Client State** → Zustand (UI state, form state, preferences)

### 2. Use Query Keys Consistently

```tsx
// Query key factory
export const campaignKeys = {
  all: ['campaigns'] as const,
  lists: () => [...campaignKeys.all, 'list'] as const,
  list: (filters: Filters) => [...campaignKeys.lists(), filters] as const,
  details: () => [...campaignKeys.all, 'detail'] as const,
  detail: (id: string) => [...campaignKeys.details(), id] as const,
};

// Usage
useQuery({ queryKey: campaignKeys.detail(id), ... });
queryClient.invalidateQueries({ queryKey: campaignKeys.lists() });
```

### 3. Select Only What You Need

```tsx
// ❌ Subscribe to entire store
const store = useUIStore();

// ✅ Subscribe to specific slices
const sidebarOpen = useUIStore((state) => state.sidebarOpen);
```
