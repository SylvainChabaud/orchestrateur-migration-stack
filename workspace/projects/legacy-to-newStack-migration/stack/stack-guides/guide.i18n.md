# Internationalization (i18n) Guide

## Overview

This guide covers i18next and react-i18next integration for multi-language support.

## Setup Configuration

### i18n Configuration File

```tsx
// src/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    debug: import.meta.env.DEV,
    defaultNS: 'translation',
    ns: ['translation', 'common', 'campaigns'],
    interpolation: {
      escapeValue: false, // React already escapes
    },
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    detection: {
      order: ['querystring', 'cookie', 'localStorage', 'navigator'],
      caches: ['localStorage', 'cookie'],
    },
    react: {
      useSuspense: true,
      bindI18n: 'languageChanged loaded',
      bindI18nStore: 'added removed',
    },
  });

export default i18n;
```

### Provider Setup

```tsx
// src/App.tsx
import { Suspense } from 'react';
import { I18nextProvider } from 'react-i18next';
import i18n from '@/i18n';

function App() {
  return (
    <I18nextProvider i18n={i18n}>
      <Suspense fallback={<LoadingSpinner />}>
        <AppContent />
      </Suspense>
    </I18nextProvider>
  );
}
```

## Using Translations

### useTranslation Hook

```tsx
import { useTranslation } from 'react-i18next';

function CampaignsList() {
  const { t, i18n, ready } = useTranslation('campaigns');

  if (!ready) {
    return <LoadingSpinner />;
  }

  const changeLanguage = (lng: string) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div>
      <h1>{t('list.title')}</h1>
      <p>{t('list.description', { count: 5 })}</p>
      
      {/* Language switcher */}
      <select value={i18n.language} onChange={(e) => changeLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="fr">Français</option>
        <option value="de">Deutsch</option>
      </select>
    </div>
  );
}
```

### Using keyPrefix

```tsx
function CampaignForm() {
  const { t } = useTranslation('campaigns', {
    keyPrefix: 'form',
  });

  return (
    <form>
      <label>{t('name.label')}</label>
      <input placeholder={t('name.placeholder')} />
      <span>{t('name.helperText')}</span>
    </form>
  );
}
```

### Multiple Namespaces

```tsx
function Dashboard() {
  const { t } = useTranslation(['campaigns', 'common']);

  return (
    <div>
      <h1>{t('campaigns:dashboard.title')}</h1>
      <button>{t('common:buttons.save')}</button>
    </div>
  );
}
```

## Translation Patterns

### Interpolation

```json
// locales/en/campaigns.json
{
  "welcome": "Hello, {{name}}!",
  "itemCount": "{{count}} item",
  "itemCount_plural": "{{count}} items"
}
```

```tsx
t('welcome', { name: 'John' }); // "Hello, John!"
t('itemCount', { count: 1 });   // "1 item"
t('itemCount', { count: 5 });   // "5 items"
```

### Pluralization

```json
{
  "campaigns": {
    "count_zero": "No campaigns",
    "count_one": "1 campaign",
    "count_other": "{{count}} campaigns"
  }
}
```

```tsx
t('campaigns.count', { count: 0 });  // "No campaigns"
t('campaigns.count', { count: 1 });  // "1 campaign"
t('campaigns.count', { count: 10 }); // "10 campaigns"
```

### Nesting

```json
{
  "common": {
    "greeting": "Hello"
  },
  "welcome": "$t(common.greeting), {{name}}!"
}
```

### Context

```json
{
  "friend": "A friend",
  "friend_male": "A boyfriend",
  "friend_female": "A girlfriend"
}
```

```tsx
t('friend');                      // "A friend"
t('friend', { context: 'male' }); // "A boyfriend"
```

## Trans Component

For translations with embedded React components:

```tsx
import { Trans } from 'react-i18next';
import { Link } from 'react-router';

function WelcomeMessage() {
  return (
    <Trans
      i18nKey="welcome.message"
      values={{ name: 'John', count: 5 }}
      components={{
        bold: <strong />,
        link: <Link to="/messages" />,
      }}
    />
  );
}
```

```json
{
  "welcome": {
    "message": "Hello <bold>{{name}}</bold>, you have {{count}} unread messages. <link>Go to messages</link>."
  }
}
```

## Translation Files Structure

```
public/
└── locales/
    ├── en/
    │   ├── translation.json  # Default namespace
    │   ├── common.json       # Common translations
    │   └── campaigns.json    # Feature-specific
    ├── fr/
    │   ├── translation.json
    │   ├── common.json
    │   └── campaigns.json
    └── de/
        ├── translation.json
        ├── common.json
        └── campaigns.json
```

### Example Translation File

```json
// locales/en/campaigns.json
{
  "list": {
    "title": "Campaigns",
    "description": "Manage your marketing campaigns",
    "emptyState": "No campaigns found",
    "createFirst": "Create your first campaign"
  },
  "detail": {
    "title": "Campaign Details",
    "tabs": {
      "overview": "Overview",
      "analytics": "Analytics",
      "settings": "Settings"
    }
  },
  "form": {
    "name": {
      "label": "Campaign Name",
      "placeholder": "Enter campaign name",
      "helperText": "Choose a descriptive name"
    },
    "status": {
      "label": "Status",
      "options": {
        "active": "Active",
        "paused": "Paused",
        "draft": "Draft"
      }
    }
  },
  "actions": {
    "create": "Create Campaign",
    "edit": "Edit",
    "delete": "Delete",
    "duplicate": "Duplicate"
  },
  "messages": {
    "created": "Campaign created successfully",
    "updated": "Campaign updated successfully",
    "deleted": "Campaign deleted"
  }
}
```

## Language Switcher Component

```tsx
import { useTranslation } from 'react-i18next';
import { Select, MenuItem } from '@mui/material';

const languages = [
  { code: 'en', label: 'English', flag: '🇬🇧' },
  { code: 'fr', label: 'Français', flag: '🇫🇷' },
  { code: 'de', label: 'Deutsch', flag: '🇩🇪' },
];

function LanguageSwitcher() {
  const { i18n } = useTranslation();

  const handleChange = (event: SelectChangeEvent) => {
    i18n.changeLanguage(event.target.value);
  };

  return (
    <Select
      value={i18n.language}
      onChange={handleChange}
      size="small"
    >
      {languages.map(({ code, label, flag }) => (
        <MenuItem key={code} value={code}>
          {flag} {label}
        </MenuItem>
      ))}
    </Select>
  );
}
```

## TypeScript Support

### Type-Safe Keys

```tsx
// src/types/i18n.d.ts
import 'react-i18next';
import common from '../../public/locales/en/common.json';
import campaigns from '../../public/locales/en/campaigns.json';

declare module 'react-i18next' {
  interface CustomTypeOptions {
    defaultNS: 'translation';
    resources: {
      common: typeof common;
      campaigns: typeof campaigns;
    };
  }
}
```

### Usage with Types

```tsx
// Now you get autocomplete for translation keys
const { t } = useTranslation('campaigns');
t('list.title'); // Autocomplete suggests valid keys
```

## Best Practices

### 1. Organize by Feature

Keep translations organized by feature/namespace for maintainability.

### 2. Use Semantic Keys

```json
// ✅ Good
{ "form.validation.required": "This field is required" }

// ❌ Bad
{ "error1": "This field is required" }
```

### 3. Extract Common Translations

Put shared translations in `common.json`:

```json
// common.json
{
  "buttons": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete"
  },
  "validation": {
    "required": "This field is required",
    "email": "Invalid email address"
  }
}
```

### 4. Handle Loading States

```tsx
function TranslatedContent() {
  const { t, ready } = useTranslation('campaigns', {
    useSuspense: false,
  });

  if (!ready) {
    return <Skeleton />;
  }

  return <div>{t('content')}</div>;
}
```
