# UI Components Guide

## Overview

This guide covers the usage of MUI (Material UI) 7.x and the @peaksys/design-system for building UI components.

## Material UI 7 Integration

### Theme Provider Setup

```tsx
import { createTheme, ThemeProvider } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';

const theme = createTheme({
  colorSchemes: {
    light: true,
    dark: true,
  },
  cssVariables: {
    colorSchemeSelector: 'class',
  },
  palette: {
    primary: {
      main: '#1976d2',
      light: '#42a5f5',
      dark: '#1565c0',
    },
    secondary: {
      main: '#dc004e',
    },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 500,
    },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          borderRadius: 8,
          textTransform: 'none',
        },
      },
    },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <YourApp />
    </ThemeProvider>
  );
}
```

### Component Style Overrides in Theme

```typescript
const theme = createTheme({
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          fontSize: '1rem',
        },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          variants: [
            {
              props: { variant: 'outlined' },
              style: {
                borderWidth: '3px',
              },
            },
          ],
        },
      },
    },
  },
});
```

### Using the `sx` Prop

The `sx` prop provides a shorthand for inline styling with theme access:

```tsx
import Box from '@mui/material/Box';
import Typography from '@mui/material/Typography';

function Component() {
  return (
    <Box
      sx={{
        bgcolor: 'background.paper',
        boxShadow: 1,
        borderRadius: 1,
        p: 2,
        minWidth: 300,
      }}
    >
      <Typography sx={{ color: 'text.secondary' }}>Sessions</Typography>
      <Typography sx={{ color: 'text.primary', fontSize: 34, fontWeight: 'medium' }}>
        98.3 K
      </Typography>
    </Box>
  );
}
```

### Styled Components with MUI

```tsx
import { styled } from '@mui/material/styles';
import Button from '@mui/material/Button';

const StyledButton = styled(Button)(({ theme }) => ({
  backgroundColor: theme.palette.primary.main,
  color: theme.palette.primary.contrastText,
  padding: theme.spacing(2, 4),
  borderRadius: theme.spacing(1),
  '&:hover': {
    backgroundColor: theme.palette.primary.dark,
    transform: 'scale(1.05)',
    transition: 'all 0.2s ease-in-out',
  },
}));

// With responsive breakpoints
const ResponsiveBox = styled(Box)(({ theme }) => ({
  padding: theme.spacing(2),
  [theme.breakpoints.down('sm')]: {
    padding: theme.spacing(1),
    fontSize: '0.875rem',
  },
  [theme.breakpoints.up('md')]: {
    padding: theme.spacing(4),
    fontSize: '1.25rem',
  },
}));
```

### Conditional Styling with ownerState

```tsx
const StatRoot = styled('div', {
  name: 'MuiStat',
  slot: 'root',
})<{ ownerState: { variant?: 'outlined' | 'filled' } }>(({ theme, ownerState }) => ({
  display: 'flex',
  flexDirection: 'column',
  padding: theme.spacing(3, 4),
  backgroundColor: theme.palette.background.paper,
  borderRadius: theme.shape.borderRadius,
  boxShadow: theme.shadows[2],
  ...(ownerState.variant === 'outlined' && {
    border: `2px solid ${theme.palette.divider}`,
    boxShadow: 'none',
  }),
}));
```

### Dark Mode Support

```tsx
import { styled } from '@mui/material/styles';

const MyComponent = styled('div')(({ theme }) => [
  {
    color: '#fff',
    backgroundColor: theme.palette.primary.main,
    '&:hover': {
      backgroundColor: theme.palette.primary.dark,
    },
  },
  theme.applyStyles('dark', {
    backgroundColor: theme.palette.secondary.main,
    '&:hover': {
      backgroundColor: theme.palette.secondary.dark,
    },
  }),
]);
```

## Common MUI Components

### Layout Components

```tsx
import { Box, Container, Grid, Stack, Paper } from '@mui/material';

// Grid Layout
<Grid container spacing={2}>
  <Grid item xs={12} md={6}>
    <Paper sx={{ p: 2 }}>Content</Paper>
  </Grid>
  <Grid item xs={12} md={6}>
    <Paper sx={{ p: 2 }}>Content</Paper>
  </Grid>
</Grid>

// Stack for vertical/horizontal arrangements
<Stack direction="row" spacing={2} alignItems="center">
  <Button>One</Button>
  <Button>Two</Button>
</Stack>
```

### Typography

```tsx
import Typography from '@mui/material/Typography';

<Typography variant="h1" gutterBottom>Heading 1</Typography>
<Typography variant="body1" color="text.secondary">
  Body text with secondary color
</Typography>
```

### Buttons

```tsx
import Button from '@mui/material/Button';
import IconButton from '@mui/material/IconButton';
import DeleteIcon from '@mui/icons-material/Delete';

<Button variant="contained" color="primary">Contained</Button>
<Button variant="outlined" color="secondary">Outlined</Button>
<Button variant="text">Text Button</Button>
<IconButton aria-label="delete" color="error">
  <DeleteIcon />
</IconButton>
```

### Tabs Component

```tsx
import { Tabs, Tab, Box } from '@mui/material';
import { useState } from 'react';

function TabPanel({ children, value, index }) {
  return (
    <div role="tabpanel" hidden={value !== index}>
      {value === index && <Box sx={{ p: 3 }}>{children}</Box>}
    </div>
  );
}

function TabsComponent() {
  const [value, setValue] = useState(0);

  const handleChange = (event: React.SyntheticEvent, newValue: number) => {
    setValue(newValue);
  };

  return (
    <Box>
      <Tabs value={value} onChange={handleChange} aria-label="tabs example">
        <Tab label="Tab One" />
        <Tab label="Tab Two" />
        <Tab label="Tab Three" />
      </Tabs>
      <TabPanel value={value} index={0}>Panel One</TabPanel>
      <TabPanel value={value} index={1}>Panel Two</TabPanel>
      <TabPanel value={value} index={2}>Panel Three</TabPanel>
    </Box>
  );
}
```

### Data Grid (Peaksys Design System)

```tsx
import { DataGrid, GridColDef } from '@peaksys/design-system';

const columns: GridColDef[] = [
  { field: 'id', headerName: 'ID', width: 90 },
  { field: 'name', headerName: 'Name', width: 150, editable: true },
  { field: 'status', headerName: 'Status', width: 110 },
];

const rows = [
  { id: 1, name: 'Campaign 1', status: 'Active' },
  { id: 2, name: 'Campaign 2', status: 'Paused' },
];

function CampaignsTable() {
  return (
    <DataGrid
      rows={rows}
      columns={columns}
      pageSize={10}
      rowsPerPageOptions={[10, 25, 50]}
      checkboxSelection
      disableSelectionOnClick
    />
  );
}
```

## Component Best Practices

### 1. Always provide accessible labels

```tsx
<IconButton aria-label="delete item" onClick={handleDelete}>
  <DeleteIcon />
</IconButton>
```

### 2. Use semantic HTML elements

```tsx
<Box component="section" sx={{ mb: 4 }}>
  <Typography component="h2" variant="h4">Section Title</Typography>
</Box>
```

### 3. Memoize expensive components

```tsx
import { memo } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  return <ComplexVisualization data={data} />;
});
```

### 4. Forward refs for reusable components

```tsx
import { forwardRef } from 'react';
import { styled } from '@mui/material/styles';

const CustomInput = forwardRef<HTMLInputElement, InputProps>((props, ref) => {
  return <StyledInput ref={ref} {...props} />;
});
```
