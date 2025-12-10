# Authentication Guide

## Overview

This guide covers authentication patterns using OIDC (OpenID Connect) with the enterprise stack.

## OIDC Configuration

### Auth Provider Setup

```tsx
// src/core/auth/authConfig.ts
export interface OidcConfig {
  authority: string;
  client_id: string;
  redirect_uri: string;
  post_logout_redirect_uri: string;
  scope: string;
  response_type: string;
  silent_redirect_uri: string;
  automaticSilentRenew: boolean;
  loadUserInfo: boolean;
}

export const oidcConfig: OidcConfig = {
  authority: import.meta.env.VITE_OIDC_AUTHORITY,
  client_id: import.meta.env.VITE_OIDC_CLIENT_ID,
  redirect_uri: `${window.location.origin}/callback`,
  post_logout_redirect_uri: `${window.location.origin}/logout`,
  scope: 'openid profile email',
  response_type: 'code',
  silent_redirect_uri: `${window.location.origin}/silent-renew`,
  automaticSilentRenew: true,
  loadUserInfo: true,
};
```

### Auth Context with Zustand

```tsx
// src/core/auth/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  roles: string[];
  permissions: string[];
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  
  setUser: (user: User | null) => void;
  setToken: (token: string | null) => void;
  login: (user: User, token: string) => void;
  logout: () => void;
  hasRole: (role: string) => boolean;
  hasPermission: (permission: string) => boolean;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      isLoading: true,

      setUser: (user) => set({ user, isAuthenticated: !!user }),
      
      setToken: (token) => set({ token }),
      
      login: (user, token) => set({
        user,
        token,
        isAuthenticated: true,
        isLoading: false,
      }),
      
      logout: () => set({
        user: null,
        token: null,
        isAuthenticated: false,
      }),

      hasRole: (role) => {
        const { user } = get();
        return user?.roles.includes(role) ?? false;
      },

      hasPermission: (permission) => {
        const { user } = get();
        return user?.permissions.includes(permission) ?? false;
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ token: state.token }),
    }
  )
);
```

### Auth Provider Component

```tsx
// src/core/auth/AuthProvider.tsx
import { createContext, useContext, useEffect, ReactNode } from 'react';
import { useAuthStore } from './authStore';
import { authApi } from './authApi';

interface AuthContextValue {
  login: () => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const { setUser, setToken, logout: logoutStore, token } = useAuthStore();

  useEffect(() => {
    // Initialize auth state on mount
    const initAuth = async () => {
      if (token) {
        try {
          const user = await authApi.getCurrentUser();
          setUser(user);
        } catch {
          logoutStore();
        }
      }
    };

    initAuth();
  }, []);

  const login = async () => {
    // Redirect to OIDC provider
    window.location.href = authApi.getLoginUrl();
  };

  const logout = async () => {
    try {
      await authApi.logout();
    } finally {
      logoutStore();
      window.location.href = '/';
    }
  };

  const refreshToken = async () => {
    try {
      const { token: newToken } = await authApi.refreshToken();
      setToken(newToken);
    } catch {
      logout();
    }
  };

  return (
    <AuthContext.Provider value={{ login, logout, refreshToken }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

## Protected Routes

### Route Guard Component

```tsx
// src/core/auth/ProtectedRoute.tsx
import { Navigate, Outlet, useLocation } from 'react-router';
import { useAuthStore } from './authStore';
import { CircularProgress, Box } from '@mui/material';

interface ProtectedRouteProps {
  requiredRoles?: string[];
  requiredPermissions?: string[];
  fallback?: string;
}

export function ProtectedRoute({
  requiredRoles = [],
  requiredPermissions = [],
  fallback = '/login',
}: ProtectedRouteProps) {
  const { isAuthenticated, isLoading, hasRole, hasPermission } = useAuthStore();
  const location = useLocation();

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
        <CircularProgress />
      </Box>
    );
  }

  if (!isAuthenticated) {
    return <Navigate to={fallback} state={{ from: location }} replace />;
  }

  // Check roles
  if (requiredRoles.length > 0) {
    const hasRequiredRole = requiredRoles.some(hasRole);
    if (!hasRequiredRole) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  // Check permissions
  if (requiredPermissions.length > 0) {
    const hasRequiredPermission = requiredPermissions.some(hasPermission);
    if (!hasRequiredPermission) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  return <Outlet />;
}
```

### Route Configuration

```tsx
// src/routes/index.tsx
import { Routes, Route } from 'react-router';
import { ProtectedRoute } from '@/core/auth/ProtectedRoute';

export function AppRoutes() {
  return (
    <Routes>
      {/* Public routes */}
      <Route path="/login" element={<LoginPage />} />
      <Route path="/callback" element={<CallbackPage />} />
      
      {/* Protected routes - require authentication */}
      <Route element={<ProtectedRoute />}>
        <Route path="/" element={<Layout />}>
          <Route index element={<DashboardPage />} />
          <Route path="campaigns" element={<CampaignsPage />} />
          <Route path="campaigns/:id" element={<CampaignDetailPage />} />
        </Route>
      </Route>

      {/* Admin routes - require admin role */}
      <Route element={<ProtectedRoute requiredRoles={['admin']} />}>
        <Route path="/admin" element={<AdminLayout />}>
          <Route path="users" element={<UsersPage />} />
          <Route path="settings" element={<AdminSettingsPage />} />
        </Route>
      </Route>

      {/* Manager routes - require specific permission */}
      <Route element={<ProtectedRoute requiredPermissions={['campaigns:manage']} />}>
        <Route path="/campaigns/create" element={<CreateCampaignPage />} />
        <Route path="/campaigns/:id/edit" element={<EditCampaignPage />} />
      </Route>

      {/* Error pages */}
      <Route path="/unauthorized" element={<UnauthorizedPage />} />
      <Route path="*" element={<NotFoundPage />} />
    </Routes>
  );
}
```

## Role-Based UI Components

### Permission-Based Rendering

```tsx
// src/core/auth/RequirePermission.tsx
import { ReactNode } from 'react';
import { useAuthStore } from './authStore';

interface RequirePermissionProps {
  permission: string | string[];
  children: ReactNode;
  fallback?: ReactNode;
}

export function RequirePermission({
  permission,
  children,
  fallback = null,
}: RequirePermissionProps) {
  const { hasPermission } = useAuthStore();

  const permissions = Array.isArray(permission) ? permission : [permission];
  const hasAccess = permissions.some(hasPermission);

  return hasAccess ? <>{children}</> : <>{fallback}</>;
}

// Usage
function CampaignCard({ campaign }) {
  return (
    <Card>
      <CardContent>
        <Typography>{campaign.name}</Typography>
      </CardContent>
      <CardActions>
        <RequirePermission permission="campaigns:edit">
          <Button onClick={handleEdit}>Edit</Button>
        </RequirePermission>
        <RequirePermission permission="campaigns:delete">
          <Button color="error" onClick={handleDelete}>Delete</Button>
        </RequirePermission>
      </CardActions>
    </Card>
  );
}
```

### Role-Based Hook

```tsx
// src/core/auth/usePermissions.ts
import { useMemo } from 'react';
import { useAuthStore } from './authStore';

export function usePermissions() {
  const { user, hasRole, hasPermission } = useAuthStore();

  const permissions = useMemo(() => ({
    canCreateCampaign: hasPermission('campaigns:create'),
    canEditCampaign: hasPermission('campaigns:edit'),
    canDeleteCampaign: hasPermission('campaigns:delete'),
    canManageUsers: hasRole('admin'),
    canViewReports: hasPermission('reports:view'),
    canExportData: hasPermission('data:export'),
  }), [user]);

  return permissions;
}

// Usage in component
function CampaignsPage() {
  const { canCreateCampaign, canDeleteCampaign } = usePermissions();

  return (
    <Page>
      <PageHeader
        title="Campaigns"
        action={
          canCreateCampaign && (
            <Button onClick={handleCreate}>Create Campaign</Button>
          )
        }
      />
      <CampaignList
        campaigns={campaigns}
        showDeleteAction={canDeleteCampaign}
      />
    </Page>
  );
}
```

## Token Management

### Axios Interceptor for Token Refresh

```tsx
// src/core/api/authInterceptor.ts
import axios from 'axios';
import { useAuthStore } from '@/core/auth/authStore';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Request interceptor - add token
api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - handle 401
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const { refreshToken, logout } = useAuthStore.getState();
        await refreshToken();
        
        const newToken = useAuthStore.getState().token;
        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        
        return api(originalRequest);
      } catch {
        useAuthStore.getState().logout();
        window.location.href = '/login';
      }
    }

    return Promise.reject(error);
  }
);

export { api };
```

## Callback Handler

```tsx
// src/pages/CallbackPage.tsx
import { useEffect } from 'react';
import { useNavigate, useSearchParams } from 'react-router';
import { useAuthStore } from '@/core/auth/authStore';
import { authApi } from '@/core/auth/authApi';
import { CircularProgress, Box, Typography } from '@mui/material';

export function CallbackPage() {
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();
  const { login } = useAuthStore();

  useEffect(() => {
    const handleCallback = async () => {
      const code = searchParams.get('code');
      const state = searchParams.get('state');

      if (!code) {
        navigate('/login', { state: { error: 'Authentication failed' } });
        return;
      }

      try {
        const { user, token } = await authApi.exchangeCode(code, state);
        login(user, token);
        
        // Redirect to original destination or home
        const returnTo = sessionStorage.getItem('returnTo') || '/';
        sessionStorage.removeItem('returnTo');
        navigate(returnTo);
      } catch (error) {
        navigate('/login', { state: { error: 'Authentication failed' } });
      }
    };

    handleCallback();
  }, [searchParams, login, navigate]);

  return (
    <Box sx={{ display: 'flex', flexDirection: 'column', alignItems: 'center', gap: 2, pt: 8 }}>
      <CircularProgress />
      <Typography>Authenticating...</Typography>
    </Box>
  );
}
```

## Best Practices

1. **Store tokens securely** - Use httpOnly cookies when possible
2. **Implement token refresh** - Handle expired tokens gracefully
3. **Use role-based access control** - Combine routes and UI guards
4. **Handle auth errors globally** - Use interceptors for consistent handling
5. **Validate permissions server-side** - Never trust client-only checks
6. **Clear sensitive data on logout** - Remove tokens and cached user data
7. **Use state parameter for CSRF protection** in OIDC flow
