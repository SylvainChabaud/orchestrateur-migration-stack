# API Client Guide

## Overview

This guide covers API client patterns using TanStack Query and custom API utilities.

## API Client Setup

### Base API Client

```tsx
// src/core/api/client.ts
import { QueryClient } from '@tanstack/react-query';

export const API_BASE_URL = import.meta.env.VITE_API_URL || '/api';

interface RequestConfig extends RequestInit {
  params?: Record<string, string | number | boolean | undefined>;
}

class ApiClient {
  private baseUrl: string;
  private defaultHeaders: HeadersInit;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
    };
  }

  private async request<T>(endpoint: string, config: RequestConfig = {}): Promise<T> {
    const { params, ...requestConfig } = config;
    
    // Build URL with query params
    const url = new URL(`${this.baseUrl}${endpoint}`);
    if (params) {
      Object.entries(params).forEach(([key, value]) => {
        if (value !== undefined) {
          url.searchParams.append(key, String(value));
        }
      });
    }

    // Get auth token
    const token = localStorage.getItem('auth_token');
    
    const response = await fetch(url.toString(), {
      ...requestConfig,
      headers: {
        ...this.defaultHeaders,
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
        ...requestConfig.headers,
      },
    });

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new ApiError(response.status, error.message || 'An error occurred', error);
    }

    // Handle empty responses (204)
    if (response.status === 204) {
      return undefined as T;
    }

    return response.json();
  }

  get<T>(endpoint: string, config?: RequestConfig): Promise<T> {
    return this.request<T>(endpoint, { ...config, method: 'GET' });
  }

  post<T>(endpoint: string, data?: unknown, config?: RequestConfig): Promise<T> {
    return this.request<T>(endpoint, {
      ...config,
      method: 'POST',
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  put<T>(endpoint: string, data?: unknown, config?: RequestConfig): Promise<T> {
    return this.request<T>(endpoint, {
      ...config,
      method: 'PUT',
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  patch<T>(endpoint: string, data?: unknown, config?: RequestConfig): Promise<T> {
    return this.request<T>(endpoint, {
      ...config,
      method: 'PATCH',
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  delete<T>(endpoint: string, config?: RequestConfig): Promise<T> {
    return this.request<T>(endpoint, { ...config, method: 'DELETE' });
  }
}

export const apiClient = new ApiClient(API_BASE_URL);

// Custom API Error
export class ApiError extends Error {
  constructor(
    public status: number,
    message: string,
    public data?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}
```

### Query Client Configuration

```tsx
// src/core/api/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: (failureCount, error) => {
        if (error instanceof ApiError && error.status === 401) {
          return false; // Don't retry on auth errors
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: false,
    },
  },
});
```

## Feature-Specific API

### Campaigns API

```tsx
// src/features/campaigns/api/campaignsApi.ts
import { apiClient } from '@/core/api/client';

export interface Campaign {
  id: string;
  name: string;
  status: 'draft' | 'active' | 'paused';
  budget: number;
  startDate: string;
  endDate?: string;
  createdAt: string;
  updatedAt: string;
}

export interface CampaignFilters {
  status?: string;
  search?: string;
  page?: number;
  limit?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export const campaignsApi = {
  getAll: (filters?: CampaignFilters) => 
    apiClient.get<PaginatedResponse<Campaign>>('/campaigns', { params: filters }),

  getById: (id: string) => 
    apiClient.get<Campaign>(`/campaigns/${id}`),

  create: (data: Omit<Campaign, 'id' | 'createdAt' | 'updatedAt'>) => 
    apiClient.post<Campaign>('/campaigns', data),

  update: (id: string, data: Partial<Campaign>) => 
    apiClient.patch<Campaign>(`/campaigns/${id}`, data),

  delete: (id: string) => 
    apiClient.delete<void>(`/campaigns/${id}`),

  updateStatus: (id: string, status: Campaign['status']) =>
    apiClient.patch<Campaign>(`/campaigns/${id}/status`, { status }),

  getStatistics: (id: string, dateRange?: { from: string; to: string }) =>
    apiClient.get<CampaignStats>(`/campaigns/${id}/statistics`, { params: dateRange }),
};
```

## Custom Hooks with React Query

### Basic Query Hook

```tsx
// src/features/campaigns/hooks/useCampaigns.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { campaignsApi, type Campaign, type CampaignFilters } from '../api/campaignsApi';

// Query Keys Factory
export const campaignKeys = {
  all: ['campaigns'] as const,
  lists: () => [...campaignKeys.all, 'list'] as const,
  list: (filters: CampaignFilters) => [...campaignKeys.lists(), filters] as const,
  details: () => [...campaignKeys.all, 'detail'] as const,
  detail: (id: string) => [...campaignKeys.details(), id] as const,
  statistics: (id: string) => [...campaignKeys.detail(id), 'statistics'] as const,
};

// Get all campaigns
export function useCampaigns(filters?: CampaignFilters) {
  return useQuery({
    queryKey: campaignKeys.list(filters ?? {}),
    queryFn: () => campaignsApi.getAll(filters),
  });
}

// Get single campaign
export function useCampaign(id: string) {
  return useQuery({
    queryKey: campaignKeys.detail(id),
    queryFn: () => campaignsApi.getById(id),
    enabled: !!id,
  });
}

// Create campaign
export function useCreateCampaign() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: campaignsApi.create,
    onSuccess: (newCampaign) => {
      // Invalidate list queries
      queryClient.invalidateQueries({ queryKey: campaignKeys.lists() });
      
      // Optionally set the new data in cache
      queryClient.setQueryData(campaignKeys.detail(newCampaign.id), newCampaign);
    },
  });
}

// Update campaign
export function useUpdateCampaign() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Campaign> }) =>
      campaignsApi.update(id, data),
    onSuccess: (updatedCampaign, { id }) => {
      // Update the cache directly
      queryClient.setQueryData(campaignKeys.detail(id), updatedCampaign);
      // Invalidate lists
      queryClient.invalidateQueries({ queryKey: campaignKeys.lists() });
    },
  });
}

// Delete campaign
export function useDeleteCampaign() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: campaignsApi.delete,
    onSuccess: (_, deletedId) => {
      // Remove from cache
      queryClient.removeQueries({ queryKey: campaignKeys.detail(deletedId) });
      // Invalidate lists
      queryClient.invalidateQueries({ queryKey: campaignKeys.lists() });
    },
  });
}
```

### Optimistic Updates

```tsx
export function useUpdateCampaignStatus() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, status }: { id: string; status: Campaign['status'] }) =>
      campaignsApi.updateStatus(id, status),

    onMutate: async ({ id, status }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: campaignKeys.detail(id) });

      // Snapshot previous value
      const previousCampaign = queryClient.getQueryData<Campaign>(campaignKeys.detail(id));

      // Optimistically update
      if (previousCampaign) {
        queryClient.setQueryData(campaignKeys.detail(id), {
          ...previousCampaign,
          status,
        });
      }

      return { previousCampaign };
    },

    onError: (err, { id }, context) => {
      // Rollback on error
      if (context?.previousCampaign) {
        queryClient.setQueryData(campaignKeys.detail(id), context.previousCampaign);
      }
    },

    onSettled: (_, __, { id }) => {
      // Refetch to ensure sync with server
      queryClient.invalidateQueries({ queryKey: campaignKeys.detail(id) });
    },
  });
}
```

## Pagination Hook

```tsx
import { useQuery, keepPreviousData } from '@tanstack/react-query';

export function usePaginatedCampaigns(filters: CampaignFilters) {
  return useQuery({
    queryKey: campaignKeys.list(filters),
    queryFn: () => campaignsApi.getAll(filters),
    placeholderData: keepPreviousData, // Keep previous data while fetching new page
  });
}

// Usage in component
function CampaignsList() {
  const [page, setPage] = useState(1);
  const [filters, setFilters] = useState<CampaignFilters>({});

  const { data, isPending, isPlaceholderData } = usePaginatedCampaigns({
    ...filters,
    page,
    limit: 20,
  });

  return (
    <Box>
      {isPending ? (
        <CircularProgress />
      ) : (
        <>
          <Box sx={{ opacity: isPlaceholderData ? 0.5 : 1 }}>
            {data?.data.map((campaign) => (
              <CampaignCard key={campaign.id} campaign={campaign} />
            ))}
          </Box>
          
          <Pagination
            page={page}
            count={data?.totalPages ?? 0}
            onChange={(_, newPage) => setPage(newPage)}
            disabled={isPlaceholderData}
          />
        </>
      )}
    </Box>
  );
}
```

## Infinite Query

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

export function useInfiniteCampaigns(filters?: Omit<CampaignFilters, 'page'>) {
  return useInfiniteQuery({
    queryKey: [...campaignKeys.lists(), 'infinite', filters],
    queryFn: ({ pageParam }) => 
      campaignsApi.getAll({ ...filters, page: pageParam, limit: 20 }),
    initialPageParam: 1,
    getNextPageParam: (lastPage) => 
      lastPage.page < lastPage.totalPages ? lastPage.page + 1 : undefined,
  });
}

// Usage
function InfiniteCampaignList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteCampaigns();

  const allCampaigns = data?.pages.flatMap((page) => page.data) ?? [];

  return (
    <>
      {allCampaigns.map((campaign) => (
        <CampaignCard key={campaign.id} campaign={campaign} />
      ))}
      
      <Button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Loading more...'
          : hasNextPage
          ? 'Load More'
          : 'No more campaigns'}
      </Button>
    </>
  );
}
```

## Error Handling

```tsx
// Global error handler
import { QueryCache, MutationCache } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) => {
      if (error instanceof ApiError && error.status === 401) {
        // Redirect to login
        window.location.href = '/login';
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error) => {
      // Show toast notification
      toast.error(error instanceof ApiError ? error.message : 'An error occurred');
    },
  }),
});
```

## Best Practices

1. **Use Query Key Factories** - Centralize and type query keys
2. **Separate API logic from hooks** - Keep API calls in separate files
3. **Use Optimistic Updates** for better UX on mutations
4. **Configure staleTime** appropriately for your data freshness needs
5. **Handle loading and error states** in all components
6. **Use placeholderData** for pagination to prevent layout shifts
