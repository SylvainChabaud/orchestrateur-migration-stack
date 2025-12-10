# Routing Guide

## Overview

This guide covers React Router 7.x patterns and navigation best practices.

## React Router 7 Setup

### Basic Configuration

```tsx
// src/routes/index.tsx
import { createBrowserRouter, RouterProvider } from 'react-router';
import { Layout } from '@/components/Layout';
import { HomePage } from '@/pages/HomePage';
import { CampaignsPage } from '@/features/campaigns/pages/CampaignsPage';
import { CampaignDetailPage } from '@/features/campaigns/pages/CampaignDetailPage';
import { NotFoundPage } from '@/pages/NotFoundPage';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <HomePage /> },
      { path: 'campaigns', element: <CampaignsPage /> },
      { path: 'campaigns/:id', element: <CampaignDetailPage /> },
      { path: '*', element: <NotFoundPage /> },
    ],
  },
]);

export function AppRouter() {
  return <RouterProvider router={router} />;
}
```

### App Entry Point

```tsx
// src/App.tsx
import { AppRouter } from '@/routes';
import { ThemeProvider } from '@mui/material/styles';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/services/queryClient';
import { theme } from '@/theme';

export function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider theme={theme}>
        <AppRouter />
      </ThemeProvider>
    </QueryClientProvider>
  );
}
```

## Navigation Hooks

### useNavigate (Replaces useHistory)

```tsx
import { useNavigate } from 'react-router';

function CampaignCard({ id }) {
  const navigate = useNavigate();

  const handleClick = () => {
    // Navigate to a route
    navigate(`/campaigns/${id}`);
  };

  const handleBack = () => {
    // Go back in history
    navigate(-1);
  };

  const handleReplace = () => {
    // Replace current history entry
    navigate('/campaigns', { replace: true });
  };

  const handleWithState = () => {
    // Pass state to destination
    navigate('/campaigns/new', { 
      state: { from: 'dashboard', prefilledData: { name: 'New Campaign' } }
    });
  };

  return (
    <div onClick={handleClick}>
      Campaign {id}
    </div>
  );
}
```

### Migration from useHistory to useNavigate

```tsx
// ❌ Old pattern (React Router v5)
import { useHistory } from 'react-router-dom';

function OldComponent() {
  const history = useHistory();
  
  history.push('/path');
  history.replace('/path');
  history.goBack();
}

// ✅ New pattern (React Router v7)
import { useNavigate } from 'react-router';

function NewComponent() {
  const navigate = useNavigate();
  
  navigate('/path');
  navigate('/path', { replace: true });
  navigate(-1);
}
```

### useParams

```tsx
import { useParams } from 'react-router';

function CampaignDetailPage() {
  const { id } = useParams<{ id: string }>();
  
  // Use id to fetch campaign data
  const { data: campaign } = useQuery({
    queryKey: ['campaign', id],
    queryFn: () => fetchCampaign(id!),
    enabled: !!id,
  });

  return <div>Campaign: {campaign?.name}</div>;
}
```

### useSearchParams

```tsx
import { useSearchParams } from 'react-router';

function CampaignsList() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const page = Number(searchParams.get('page')) || 1;
  const filter = searchParams.get('filter') || 'all';
  
  const handlePageChange = (newPage: number) => {
    setSearchParams(prev => {
      prev.set('page', String(newPage));
      return prev;
    });
  };
  
  const handleFilterChange = (newFilter: string) => {
    setSearchParams(prev => {
      prev.set('filter', newFilter);
      prev.set('page', '1'); // Reset page on filter change
      return prev;
    });
  };

  return (
    <div>
      <Filters value={filter} onChange={handleFilterChange} />
      <CampaignsTable page={page} filter={filter} />
      <Pagination page={page} onChange={handlePageChange} />
    </div>
  );
}
```

### useLocation

```tsx
import { useLocation } from 'react-router';

function BreadcrumbTracker() {
  const location = useLocation();
  
  // Access current path
  console.log(location.pathname); // e.g., '/campaigns/123'
  
  // Access search params as string
  console.log(location.search); // e.g., '?page=2&filter=active'
  
  // Access passed state
  const { from } = location.state || {};
  
  return <Breadcrumbs path={location.pathname} />;
}
```

## Route Configuration Patterns

### Nested Routes with Layout

```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorBoundary />,
    children: [
      { index: true, element: <HomePage /> },
      {
        path: 'campaigns',
        element: <CampaignsLayout />,
        children: [
          { index: true, element: <CampaignsList /> },
          { path: ':id', element: <CampaignDetail /> },
          { path: ':id/edit', element: <CampaignEdit /> },
          { path: 'new', element: <CampaignCreate /> },
        ],
      },
      {
        path: 'settings',
        element: <SettingsLayout />,
        children: [
          { index: true, element: <GeneralSettings /> },
          { path: 'profile', element: <ProfileSettings /> },
          { path: 'security', element: <SecuritySettings /> },
        ],
      },
    ],
  },
]);
```

### Protected Routes

```tsx
import { Navigate, Outlet, useLocation } from 'react-router';
import { useAuth } from '@/hooks/useAuth';

function ProtectedRoute() {
  const { isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <Outlet />;
}

// Usage in router config
const router = createBrowserRouter([
  {
    path: '/login',
    element: <LoginPage />,
  },
  {
    element: <ProtectedRoute />,
    children: [
      { path: '/', element: <Dashboard /> },
      { path: '/campaigns', element: <Campaigns /> },
    ],
  },
]);
```

### Route with Loader (Data Fetching)

```tsx
import { useLoaderData } from 'react-router';

// Loader function
async function campaignLoader({ params }) {
  const response = await fetch(`/api/campaigns/${params.id}`);
  if (!response.ok) {
    throw new Response('Campaign not found', { status: 404 });
  }
  return response.json();
}

// Route configuration
{
  path: 'campaigns/:id',
  element: <CampaignDetail />,
  loader: campaignLoader,
}

// Component
function CampaignDetail() {
  const campaign = useLoaderData();
  
  return <div>{campaign.name}</div>;
}
```

## Navigation Components

### Link Component

```tsx
import { Link, NavLink } from 'react-router';

// Basic link
<Link to="/campaigns">View Campaigns</Link>

// Link with state
<Link to="/campaigns/new" state={{ from: 'dashboard' }}>
  Create Campaign
</Link>

// NavLink for active styling
<NavLink
  to="/campaigns"
  className={({ isActive }) => isActive ? 'nav-active' : ''}
>
  Campaigns
</NavLink>

// NavLink with MUI styling
<NavLink to="/campaigns">
  {({ isActive }) => (
    <Button variant={isActive ? 'contained' : 'text'}>
      Campaigns
    </Button>
  )}
</NavLink>
```

### Outlet for Nested Content

```tsx
import { Outlet } from 'react-router';

function CampaignsLayout() {
  return (
    <div>
      <CampaignsHeader />
      <CampaignsSidebar />
      <main>
        {/* Child routes render here */}
        <Outlet />
      </main>
    </div>
  );
}
```

## Route Constants

```tsx
// src/routes/paths.ts
export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  CAMPAIGNS: {
    LIST: '/campaigns',
    DETAIL: (id: string) => `/campaigns/${id}`,
    EDIT: (id: string) => `/campaigns/${id}/edit`,
    CREATE: '/campaigns/new',
  },
  SETTINGS: {
    ROOT: '/settings',
    PROFILE: '/settings/profile',
    SECURITY: '/settings/security',
  },
} as const;

// Usage
import { ROUTES } from '@/routes/paths';

navigate(ROUTES.CAMPAIGNS.DETAIL(campaignId));
<Link to={ROUTES.CAMPAIGNS.LIST}>View All</Link>
```

## Navigation Guards

### Blocking Navigation (Confirm Leave)

```tsx
import { useBlocker } from 'react-router';

function FormWithUnsavedChanges() {
  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);

  const blocker = useBlocker(
    ({ currentLocation, nextLocation }) =>
      hasUnsavedChanges && currentLocation.pathname !== nextLocation.pathname
  );

  return (
    <>
      <form>
        {/* form fields */}
      </form>
      
      {blocker.state === 'blocked' && (
        <ConfirmDialog
          open
          title="Unsaved Changes"
          message="You have unsaved changes. Are you sure you want to leave?"
          onConfirm={() => blocker.proceed()}
          onCancel={() => blocker.reset()}
        />
      )}
    </>
  );
}
```
