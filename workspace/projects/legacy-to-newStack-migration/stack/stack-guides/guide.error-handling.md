# Error Handling and Logging Guide

## Overview

This guide covers error handling patterns and logging strategies for the enterprise stack.

## Error Boundary Components

### Global Error Boundary

```tsx
// src/core/errors/ErrorBoundary.tsx
import { Component, ReactNode, ErrorInfo } from 'react';
import { Box, Typography, Button } from '@mui/material';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to monitoring service
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Call custom error handler
    this.props.onError?.(error, errorInfo);
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <Box sx={{ p: 4, textAlign: 'center' }}>
          <Typography variant="h4" gutterBottom>
            Something went wrong
          </Typography>
          <Typography color="text.secondary" sx={{ mb: 2 }}>
            {this.state.error?.message}
          </Typography>
          <Button variant="contained" onClick={this.handleReset}>
            Try Again
          </Button>
        </Box>
      );
    }

    return this.props.children;
  }
}
```

### Feature-Level Error Boundary

```tsx
// src/core/errors/QueryErrorBoundary.tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from './ErrorBoundary';

interface Props {
  children: ReactNode;
}

export function QueryErrorBoundary({ children }: Props) {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          fallback={<ErrorFallback onRetry={reset} />}
          onError={(error) => {
            // Log API errors
            console.error('Query error:', error);
          }}
        >
          {children}
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

function ErrorFallback({ onRetry }: { onRetry: () => void }) {
  return (
    <Box sx={{ p: 4, textAlign: 'center' }}>
      <Typography variant="h6">Failed to load data</Typography>
      <Button onClick={onRetry} sx={{ mt: 2 }}>
        Retry
      </Button>
    </Box>
  );
}
```

## API Error Handling

### Custom Error Classes

```tsx
// src/core/errors/ApiError.ts
export class ApiError extends Error {
  constructor(
    public status: number,
    message: string,
    public code?: string,
    public details?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'ApiError';
  }

  static isApiError(error: unknown): error is ApiError {
    return error instanceof ApiError;
  }

  static fromResponse(response: Response, body?: unknown): ApiError {
    const data = body as { message?: string; code?: string; details?: Record<string, unknown> };
    return new ApiError(
      response.status,
      data?.message || response.statusText,
      data?.code,
      data?.details
    );
  }
}

export class NetworkError extends Error {
  constructor(message = 'Network error occurred') {
    super(message);
    this.name = 'NetworkError';
  }
}

export class ValidationError extends Error {
  constructor(
    message: string,
    public fields: Record<string, string[]>
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

### Error Handler Utility

```tsx
// src/core/errors/handleError.ts
import { ApiError, NetworkError, ValidationError } from './ApiError';

interface ErrorResult {
  message: string;
  type: 'error' | 'warning' | 'info';
  shouldRetry: boolean;
  fieldErrors?: Record<string, string>;
}

export function handleError(error: unknown): ErrorResult {
  // Network errors
  if (error instanceof TypeError && error.message.includes('fetch')) {
    return {
      message: 'Unable to connect to server. Please check your connection.',
      type: 'error',
      shouldRetry: true,
    };
  }

  if (error instanceof NetworkError) {
    return {
      message: error.message,
      type: 'error',
      shouldRetry: true,
    };
  }

  // API errors
  if (ApiError.isApiError(error)) {
    switch (error.status) {
      case 400:
        return {
          message: error.message || 'Invalid request',
          type: 'warning',
          shouldRetry: false,
          fieldErrors: error.details as Record<string, string>,
        };
      case 401:
        return {
          message: 'Your session has expired. Please log in again.',
          type: 'warning',
          shouldRetry: false,
        };
      case 403:
        return {
          message: 'You do not have permission to perform this action.',
          type: 'error',
          shouldRetry: false,
        };
      case 404:
        return {
          message: 'The requested resource was not found.',
          type: 'warning',
          shouldRetry: false,
        };
      case 409:
        return {
          message: error.message || 'Conflict with existing data.',
          type: 'warning',
          shouldRetry: false,
        };
      case 422:
        return {
          message: 'Validation failed. Please check your input.',
          type: 'warning',
          shouldRetry: false,
          fieldErrors: error.details as Record<string, string>,
        };
      case 429:
        return {
          message: 'Too many requests. Please wait and try again.',
          type: 'warning',
          shouldRetry: true,
        };
      case 500:
      case 502:
      case 503:
        return {
          message: 'Server error. Please try again later.',
          type: 'error',
          shouldRetry: true,
        };
      default:
        return {
          message: error.message || 'An unexpected error occurred.',
          type: 'error',
          shouldRetry: false,
        };
    }
  }

  // Validation errors
  if (error instanceof ValidationError) {
    return {
      message: error.message,
      type: 'warning',
      shouldRetry: false,
      fieldErrors: Object.fromEntries(
        Object.entries(error.fields).map(([key, messages]) => [key, messages[0]])
      ),
    };
  }

  // Generic errors
  if (error instanceof Error) {
    return {
      message: error.message,
      type: 'error',
      shouldRetry: false,
    };
  }

  return {
    message: 'An unexpected error occurred.',
    type: 'error',
    shouldRetry: false,
  };
}
```

## Toast Notifications

### Toast Provider

```tsx
// src/core/notifications/ToastProvider.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';
import { Snackbar, Alert, AlertColor } from '@mui/material';

interface Toast {
  id: string;
  message: string;
  severity: AlertColor;
  duration?: number;
}

interface ToastContextValue {
  showToast: (message: string, severity?: AlertColor, duration?: number) => void;
  showError: (message: string) => void;
  showSuccess: (message: string) => void;
  showWarning: (message: string) => void;
  showInfo: (message: string) => void;
}

const ToastContext = createContext<ToastContextValue | null>(null);

export function ToastProvider({ children }: { children: ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const showToast = useCallback(
    (message: string, severity: AlertColor = 'info', duration = 5000) => {
      const id = Date.now().toString();
      setToasts((prev) => [...prev, { id, message, severity, duration }]);
    },
    []
  );

  const handleClose = useCallback((id: string) => {
    setToasts((prev) => prev.filter((toast) => toast.id !== id));
  }, []);

  const value: ToastContextValue = {
    showToast,
    showError: (message) => showToast(message, 'error'),
    showSuccess: (message) => showToast(message, 'success'),
    showWarning: (message) => showToast(message, 'warning'),
    showInfo: (message) => showToast(message, 'info'),
  };

  return (
    <ToastContext.Provider value={value}>
      {children}
      {toasts.map((toast) => (
        <Snackbar
          key={toast.id}
          open
          autoHideDuration={toast.duration}
          onClose={() => handleClose(toast.id)}
          anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        >
          <Alert
            severity={toast.severity}
            onClose={() => handleClose(toast.id)}
            variant="filled"
          >
            {toast.message}
          </Alert>
        </Snackbar>
      ))}
    </ToastContext.Provider>
  );
}

export function useToast() {
  const context = useContext(ToastContext);
  if (!context) {
    throw new Error('useToast must be used within ToastProvider');
  }
  return context;
}
```

## React Query Error Handling

### Global Error Handlers

```tsx
// src/core/api/queryClient.ts
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';
import { handleError } from '@/core/errors/handleError';
import { useToast } from '@/core/notifications/ToastProvider';

// Create outside component for singleton
let toastRef: ReturnType<typeof useToast> | null = null;

export function setToastRef(toast: ReturnType<typeof useToast>) {
  toastRef = toast;
}

export const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      // Only show toast for queries that have already loaded data
      // (avoid showing errors for initial loads - handle in component)
      if (query.state.data !== undefined) {
        const { message, type } = handleError(error);
        if (type === 'error') {
          toastRef?.showError(message);
        }
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error) => {
      const { message } = handleError(error);
      toastRef?.showError(message);
    },
  }),
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        const { shouldRetry } = handleError(error);
        return shouldRetry && failureCount < 3;
      },
    },
  },
});
```

### Mutation with Error Handling

```tsx
// src/features/campaigns/hooks/useCreateCampaign.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { campaignsApi } from '../api/campaignsApi';
import { useToast } from '@/core/notifications/ToastProvider';
import { handleError } from '@/core/errors/handleError';

export function useCreateCampaign() {
  const queryClient = useQueryClient();
  const { showSuccess, showError } = useToast();

  return useMutation({
    mutationFn: campaignsApi.create,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['campaigns'] });
      showSuccess(`Campaign "${data.name}" created successfully!`);
    },
    onError: (error) => {
      const { message, fieldErrors } = handleError(error);
      showError(message);
      
      // Return field errors for form handling
      return { fieldErrors };
    },
  });
}
```

## Logging Service

```tsx
// src/core/logging/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
  error?: Error;
}

class Logger {
  private isDevelopment = import.meta.env.DEV;

  private log(level: LogLevel, message: string, context?: Record<string, unknown>, error?: Error) {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context,
      error,
    };

    // Console logging in development
    if (this.isDevelopment) {
      const consoleMethod = level === 'debug' ? 'log' : level;
      console[consoleMethod](`[${level.toUpperCase()}]`, message, context, error);
    }

    // Send to monitoring service in production
    if (!this.isDevelopment && (level === 'warn' || level === 'error')) {
      this.sendToMonitoring(entry);
    }
  }

  private async sendToMonitoring(entry: LogEntry) {
    try {
      // Example: Send to your monitoring service
      await fetch('/api/logs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(entry),
      });
    } catch {
      // Fail silently - don't break the app for logging
    }
  }

  debug(message: string, context?: Record<string, unknown>) {
    this.log('debug', message, context);
  }

  info(message: string, context?: Record<string, unknown>) {
    this.log('info', message, context);
  }

  warn(message: string, context?: Record<string, unknown>) {
    this.log('warn', message, context);
  }

  error(message: string, error?: Error, context?: Record<string, unknown>) {
    this.log('error', message, context, error);
  }
}

export const logger = new Logger();

// Usage
logger.info('Campaign created', { campaignId: '123', name: 'Test' });
logger.error('Failed to create campaign', error, { input: campaignData });
```

## Best Practices

1. **Use Error Boundaries** to prevent crashes from breaking the entire app
2. **Categorize errors** by type (API, network, validation) for appropriate handling
3. **Show user-friendly messages** - never expose technical details to users
4. **Log errors** for debugging but sanitize sensitive data
5. **Provide retry options** for recoverable errors
6. **Use toast notifications** for transient errors
7. **Handle form validation errors** inline with fields
8. **Set up monitoring** for production error tracking
