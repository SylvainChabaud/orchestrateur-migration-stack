# Performance and Accessibility Guide

## Overview

This guide covers performance optimization and accessibility (a11y) best practices for the enterprise stack.

## Performance Optimization

### React Performance Patterns

#### Memoization

```tsx
import { memo, useMemo, useCallback } from 'react';

// Memoize expensive components
const CampaignCard = memo(function CampaignCard({ campaign, onSelect }: Props) {
  return (
    <Card onClick={() => onSelect(campaign.id)}>
      <Typography>{campaign.name}</Typography>
    </Card>
  );
});

// Memoize expensive calculations
function CampaignStats({ campaigns }: { campaigns: Campaign[] }) {
  const statistics = useMemo(() => {
    return campaigns.reduce((acc, campaign) => ({
      totalBudget: acc.totalBudget + campaign.budget,
      activeCampaigns: acc.activeCampaigns + (campaign.status === 'active' ? 1 : 0),
      avgBudget: (acc.totalBudget + campaign.budget) / (campaigns.length || 1),
    }), { totalBudget: 0, activeCampaigns: 0, avgBudget: 0 });
  }, [campaigns]);

  return <StatsDisplay stats={statistics} />;
}

// Memoize callbacks passed to children
function CampaignsList({ campaigns }: Props) {
  const handleSelect = useCallback((id: string) => {
    navigate(`/campaigns/${id}`);
  }, [navigate]);

  return (
    <Box>
      {campaigns.map((campaign) => (
        <CampaignCard 
          key={campaign.id}
          campaign={campaign}
          onSelect={handleSelect}
        />
      ))}
    </Box>
  );
}
```

### Lazy Loading

```tsx
import { lazy, Suspense } from 'react';

// Route-level lazy loading
const CampaignsPage = lazy(() => import('./features/campaigns/pages/CampaignsPage'));
const ReportsPage = lazy(() => import('./features/reports/pages/ReportsPage'));

// Component-level lazy loading
const HeavyDataGrid = lazy(() => import('./components/HeavyDataGrid'));
const ChartComponent = lazy(() => import('./components/ChartComponent'));

// With loading fallback
function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/campaigns" element={<CampaignsPage />} />
        <Route path="/reports" element={<ReportsPage />} />
      </Routes>
    </Suspense>
  );
}

// Named exports lazy loading
const { CampaignForm } = lazy(() =>
  import('./features/campaigns/components').then((module) => ({
    default: module.CampaignForm,
  }))
);
```

### Image Optimization

```tsx
// Lazy loading images
function CampaignImage({ src, alt }: { src: string; alt: string }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      decoding="async"
      style={{ aspectRatio: '16/9', objectFit: 'cover' }}
    />
  );
}

// Responsive images with srcset
function ResponsiveImage({ src, alt }: Props) {
  return (
    <picture>
      <source
        media="(min-width: 1200px)"
        srcSet={`${src}?w=1200 1x, ${src}?w=2400 2x`}
      />
      <source
        media="(min-width: 768px)"
        srcSet={`${src}?w=768 1x, ${src}?w=1536 2x`}
      />
      <img
        src={`${src}?w=400`}
        srcSet={`${src}?w=400 1x, ${src}?w=800 2x`}
        alt={alt}
        loading="lazy"
      />
    </picture>
  );
}
```

### Virtualization for Large Lists

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

function VirtualizedCampaignList({ campaigns }: { campaigns: Campaign[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: campaigns.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 72, // Estimated row height
    overscan: 5,
  });

  return (
    <Box
      ref={parentRef}
      sx={{
        height: 600,
        overflow: 'auto',
      }}
    >
      <Box
        sx={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <Box
            key={virtualItem.key}
            sx={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <CampaignRow campaign={campaigns[virtualItem.index]} />
          </Box>
        ))}
      </Box>
    </Box>
  );
}
```

### Debouncing and Throttling

```tsx
import { useDeferredValue, useState, useMemo } from 'react';

// Using useDeferredValue for search
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  const results = useCampaignSearch(deferredQuery);

  return (
    <Box sx={{ opacity: isStale ? 0.7 : 1, transition: 'opacity 0.2s' }}>
      {results.map((result) => (
        <SearchResult key={result.id} result={result} />
      ))}
    </Box>
  );
}

// Custom debounce hook
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  const { data } = useCampaignsSearch(debouncedQuery);

  return (
    <TextField
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search campaigns..."
    />
  );
}
```

## Accessibility (a11y)

### Semantic HTML and ARIA

```tsx
// Use semantic HTML elements
function CampaignPage() {
  return (
    <main>
      <header>
        <h1>Campaigns</h1>
        <nav aria-label="Campaign actions">
          <Button>Create Campaign</Button>
        </nav>
      </header>

      <section aria-labelledby="active-campaigns">
        <h2 id="active-campaigns">Active Campaigns</h2>
        <ul role="list">
          {campaigns.map((campaign) => (
            <li key={campaign.id}>
              <CampaignCard campaign={campaign} />
            </li>
          ))}
        </ul>
      </section>

      <aside aria-label="Campaign statistics">
        <CampaignStats />
      </aside>
    </main>
  );
}

// ARIA labels for icon buttons
function ActionButtons({ campaign }: Props) {
  return (
    <Box>
      <IconButton aria-label={`Edit ${campaign.name}`}>
        <EditIcon />
      </IconButton>
      <IconButton aria-label={`Delete ${campaign.name}`}>
        <DeleteIcon />
      </IconButton>
    </Box>
  );
}
```

### Keyboard Navigation

```tsx
import { useRef, useEffect } from 'react';

// Focus management
function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      // Store current focus
      previousFocusRef.current = document.activeElement as HTMLElement;
      // Focus the modal
      modalRef.current?.focus();
    } else {
      // Restore focus when closing
      previousFocusRef.current?.focus();
    }
  }, [isOpen]);

  // Trap focus within modal
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Escape') {
      onClose();
    }

    if (e.key === 'Tab' && modalRef.current) {
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0] as HTMLElement;
      const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement.focus();
      }
    }
  };

  if (!isOpen) return null;

  return (
    <Box
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      tabIndex={-1}
      onKeyDown={handleKeyDown}
    >
      {children}
    </Box>
  );
}

// Keyboard accessible menu
function DropdownMenu({ items }: { items: MenuItem[] }) {
  const [activeIndex, setActiveIndex] = useState(-1);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex((prev) => Math.min(prev + 1, items.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex((prev) => Math.max(prev - 1, 0));
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        items[activeIndex]?.onClick();
        break;
    }
  };

  return (
    <MenuList role="menu" onKeyDown={handleKeyDown}>
      {items.map((item, index) => (
        <MenuItem
          key={item.id}
          role="menuitem"
          tabIndex={index === activeIndex ? 0 : -1}
          aria-selected={index === activeIndex}
        >
          {item.label}
        </MenuItem>
      ))}
    </MenuList>
  );
}
```

### Form Accessibility

```tsx
function AccessibleForm() {
  const { register, formState: { errors } } = useForm();

  return (
    <form aria-labelledby="form-title">
      <Typography id="form-title" variant="h2">
        Create Campaign
      </Typography>

      <FormControl error={!!errors.name}>
        <FormLabel htmlFor="campaign-name">
          Campaign Name
          <Typography component="span" color="error"> *</Typography>
        </FormLabel>
        <TextField
          {...register('name', { required: 'Name is required' })}
          id="campaign-name"
          aria-describedby={errors.name ? 'name-error' : 'name-hint'}
          aria-invalid={!!errors.name}
          aria-required="true"
        />
        <FormHelperText id="name-hint">
          Enter a descriptive name for your campaign
        </FormHelperText>
        {errors.name && (
          <FormHelperText id="name-error" error role="alert">
            {errors.name.message}
          </FormHelperText>
        )}
      </FormControl>

      <Button type="submit" aria-describedby="submit-hint">
        Create Campaign
      </Button>
      <Typography id="submit-hint" variant="caption" sx={{ srOnly: true }}>
        Submitting will create a new campaign
      </Typography>
    </form>
  );
}
```

### Live Regions for Dynamic Content

```tsx
function NotificationArea() {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  return (
    <>
      {/* Polite announcements for non-urgent updates */}
      <Box
        role="status"
        aria-live="polite"
        aria-atomic="true"
        sx={{ srOnly: true }}
      >
        {notifications.filter((n) => n.type === 'info').slice(-1)[0]?.message}
      </Box>

      {/* Assertive announcements for urgent updates */}
      <Box
        role="alert"
        aria-live="assertive"
        aria-atomic="true"
        sx={{ srOnly: true }}
      >
        {notifications.filter((n) => n.type === 'error').slice(-1)[0]?.message}
      </Box>

      {/* Visual notifications */}
      <Stack spacing={1}>
        {notifications.map((notification) => (
          <Alert key={notification.id} severity={notification.type}>
            {notification.message}
          </Alert>
        ))}
      </Stack>
    </>
  );
}
```

### Color Contrast and Visual Design

```tsx
// Theme with accessible colors
const theme = createTheme({
  palette: {
    // Ensure 4.5:1 contrast ratio for text
    primary: {
      main: '#1565c0', // Darker blue for better contrast
      contrastText: '#ffffff',
    },
    error: {
      main: '#c62828', // Darker red
      contrastText: '#ffffff',
    },
    text: {
      primary: '#1a1a1a', // Near black for maximum contrast
      secondary: '#555555', // Still meets WCAG AA
    },
  },
});

// Focus indicators
const accessibleStyles = {
  '&:focus-visible': {
    outline: '3px solid',
    outlineColor: 'primary.main',
    outlineOffset: '2px',
  },
};
```

### Screen Reader Only Content

```tsx
// sx prop for visually hidden content
const srOnly = {
  position: 'absolute',
  width: '1px',
  height: '1px',
  padding: 0,
  margin: '-1px',
  overflow: 'hidden',
  clip: 'rect(0, 0, 0, 0)',
  whiteSpace: 'nowrap',
  border: 0,
};

// Usage
<Typography sx={srOnly}>
  This text is only visible to screen readers
</Typography>

// Skip link for keyboard navigation
function SkipLink() {
  return (
    <Button
      href="#main-content"
      sx={{
        ...srOnly,
        '&:focus': {
          position: 'fixed',
          top: 16,
          left: 16,
          width: 'auto',
          height: 'auto',
          clip: 'auto',
          zIndex: 9999,
        },
      }}
    >
      Skip to main content
    </Button>
  );
}
```

## Testing Accessibility

```tsx
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('CampaignForm accessibility', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(<CampaignForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

## Best Practices Summary

### Performance
1. **Use React.memo** for pure components
2. **Virtualize long lists** with @tanstack/react-virtual
3. **Lazy load routes and heavy components**
4. **Debounce expensive operations**
5. **Optimize images** with lazy loading and srcset

### Accessibility
1. **Use semantic HTML** elements
2. **Provide ARIA labels** for non-text content
3. **Ensure keyboard navigation** works
4. **Maintain focus management** in modals
5. **Use live regions** for dynamic updates
6. **Ensure sufficient color contrast** (4.5:1 minimum)
7. **Test with screen readers** and axe
