# Styles Guide

## Overview

This guide covers styling approaches using Emotion and MUI's styling system.

## Styling Approaches

### 1. MUI `sx` Prop (Recommended for quick styling)

```tsx
import Box from '@mui/material/Box';

<Box
  sx={{
    // Theme-aware values
    bgcolor: 'background.paper',
    color: 'text.primary',
    
    // Spacing (theme.spacing multiplier)
    p: 2,           // padding: theme.spacing(2) = 16px
    mt: 1,          // marginTop: 8px
    mx: 'auto',     // marginLeft/Right: auto
    
    // Responsive values
    width: { xs: '100%', sm: 400, md: 600 },
    
    // Pseudo-selectors
    '&:hover': {
      bgcolor: 'action.hover',
    },
    
    // Nested selectors
    '& .child': {
      color: 'primary.main',
    },
  }}
>
  Content
</Box>
```

### 2. Styled Components (Recommended for reusable components)

```tsx
import { styled } from '@mui/material/styles';
import Paper from '@mui/material/Paper';

const StyledCard = styled(Paper)(({ theme }) => ({
  padding: theme.spacing(2),
  backgroundColor: theme.palette.background.default,
  borderRadius: theme.shape.borderRadius,
  boxShadow: theme.shadows[3],
  
  // Responsive
  [theme.breakpoints.down('sm')]: {
    padding: theme.spacing(1),
  },
  
  // Dark mode
  ...theme.applyStyles('dark', {
    backgroundColor: theme.palette.grey[900],
  }),
}));

// Usage
<StyledCard>Content</StyledCard>
```

### 3. Emotion CSS Prop

```tsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

const dynamicStyle = (props) => css`
  color: ${props.color};
  padding: 20px;
  
  &:hover {
    opacity: 0.8;
  }
`;

function Component({ color }) {
  return <div css={dynamicStyle({ color })}>Styled content</div>;
}
```

### 4. Emotion Styled (Standalone)

```tsx
import styled from '@emotion/styled';

const Container = styled.div`
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 24px;
`;

const Title = styled.h1<{ variant?: 'primary' | 'secondary' }>`
  font-size: 2rem;
  color: ${props => props.variant === 'primary' ? '#1976d2' : '#333'};
`;
```

## Theme-Aware Styling

### Accessing Theme Values

```tsx
import { styled, useTheme } from '@mui/material/styles';

// In styled components
const StyledDiv = styled('div')(({ theme }) => ({
  color: theme.palette.primary.main,
  fontSize: theme.typography.h6.fontSize,
  borderRadius: theme.shape.borderRadius,
  transition: theme.transitions.create(['background-color'], {
    duration: theme.transitions.duration.short,
  }),
}));

// In components with hook
function Component() {
  const theme = useTheme();
  
  return (
    <div style={{ color: theme.palette.primary.main }}>
      Content
    </div>
  );
}
```

### Spacing System

```tsx
// theme.spacing(n) = n * 8px (default)
sx={{
  p: 1,    // 8px
  p: 2,    // 16px
  p: 3,    // 24px
  mt: 4,   // marginTop: 32px
  mx: 2,   // marginLeft/Right: 16px
  py: 1.5, // paddingTop/Bottom: 12px
}}
```

### Breakpoints

```tsx
// Default breakpoints:
// xs: 0px, sm: 600px, md: 900px, lg: 1200px, xl: 1536px

// In sx prop
sx={{
  width: {
    xs: '100%',  // 0-600px
    sm: '80%',   // 600-900px
    md: 600,     // 900-1200px
    lg: 800,     // 1200px+
  },
}}

// In styled components
const ResponsiveBox = styled(Box)(({ theme }) => ({
  width: '100%',
  [theme.breakpoints.up('sm')]: {
    width: '80%',
  },
  [theme.breakpoints.up('md')]: {
    width: 600,
  },
}));
```

## CSS Variables

### Using MUI CSS Variables

```tsx
import { styled } from '@mui/material/styles';

const StyledComponent = styled('div')(({ theme }) => ({
  // Access CSS variables
  backgroundColor: theme.vars.palette.primary.main,
  color: theme.vars.palette.text.primary,
  boxShadow: theme.vars.shadow.sm,
  borderRadius: theme.vars.radius.sm,
}));
```

### Custom CSS Variables

```tsx
const theme = createTheme({
  cssVariables: true,
  palette: {
    primary: {
      main: '#1976d2',
    },
  },
});

// In CSS
.my-class {
  color: var(--mui-palette-primary-main);
}
```

## Styling Patterns

### Component Variants

```tsx
const StyledButton = styled('button')<{ variant?: 'primary' | 'secondary' }>(
  ({ theme, variant = 'primary' }) => ({
    padding: theme.spacing(1, 2),
    borderRadius: theme.shape.borderRadius,
    cursor: 'pointer',
    
    ...(variant === 'primary' && {
      backgroundColor: theme.palette.primary.main,
      color: theme.palette.primary.contrastText,
      '&:hover': {
        backgroundColor: theme.palette.primary.dark,
      },
    }),
    
    ...(variant === 'secondary' && {
      backgroundColor: 'transparent',
      border: `1px solid ${theme.palette.primary.main}`,
      color: theme.palette.primary.main,
      '&:hover': {
        backgroundColor: theme.palette.action.hover,
      },
    }),
  })
);
```

### Composition Pattern

```tsx
// Base styles
const baseCardStyles = (theme) => ({
  padding: theme.spacing(2),
  borderRadius: theme.shape.borderRadius,
  backgroundColor: theme.palette.background.paper,
});

// Extended styles
const StyledInfoCard = styled('div')(({ theme }) => ({
  ...baseCardStyles(theme),
  border: `1px solid ${theme.palette.info.main}`,
}));

const StyledSuccessCard = styled('div')(({ theme }) => ({
  ...baseCardStyles(theme),
  border: `1px solid ${theme.palette.success.main}`,
}));
```

### Animation Styles

```tsx
import { keyframes } from '@emotion/react';
import { styled } from '@mui/material/styles';

const fadeIn = keyframes`
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
`;

const AnimatedCard = styled('div')({
  animation: `${fadeIn} 0.3s ease-out`,
});

// With MUI transitions
const HoverCard = styled(Paper)(({ theme }) => ({
  transition: theme.transitions.create(['transform', 'box-shadow'], {
    duration: theme.transitions.duration.short,
  }),
  '&:hover': {
    transform: 'translateY(-4px)',
    boxShadow: theme.shadows[8],
  },
}));
```

## File Organization

```
src/
├── theme/
│   ├── index.ts            # Theme export
│   ├── palette.ts          # Color palette
│   ├── typography.ts       # Typography settings
│   ├── components.ts       # Component overrides
│   └── breakpoints.ts      # Custom breakpoints
├── components/
│   └── Button/
│       ├── Button.tsx
│       ├── Button.styles.ts # Styled components
│       └── index.ts
```

### Example styles file:

```tsx
// Button.styles.ts
import { styled } from '@mui/material/styles';
import MuiButton from '@mui/material/Button';

export const StyledButton = styled(MuiButton)(({ theme }) => ({
  borderRadius: 8,
  textTransform: 'none',
  fontWeight: 600,
}));

export const IconWrapper = styled('span')(({ theme }) => ({
  display: 'flex',
  alignItems: 'center',
  marginRight: theme.spacing(1),
}));
```
