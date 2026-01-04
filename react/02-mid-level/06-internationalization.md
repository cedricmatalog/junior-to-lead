# Internationalization (i18n)

Build applications that work globally across languages and cultures.

## Learning Objectives

By the end of this module, you will:
- Set up react-i18next for translations
- Handle pluralization and formatting
- Support right-to-left (RTL) languages
- Manage translation workflows

## Setting Up react-i18next

```jsx
// i18n.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import en from './locales/en.json';
import es from './locales/es.json';
import ar from './locales/ar.json';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      en: { translation: en },
      es: { translation: es },
      ar: { translation: ar },
    },
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false, // React already escapes
    },
  });

export default i18n;

// index.js
import './i18n';
// ... rest of app
```

## Translation Files

```json
// locales/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "loading": "Loading..."
  },
  "greeting": "Hello, {{name}}!",
  "items": {
    "one": "{{count}} item",
    "other": "{{count}} items"
  },
  "cart": {
    "empty": "Your cart is empty",
    "total": "Total: {{price, currency}}"
  }
}

// locales/es.json
{
  "common": {
    "save": "Guardar",
    "cancel": "Cancelar",
    "loading": "Cargando..."
  },
  "greeting": "¡Hola, {{name}}!",
  "items": {
    "one": "{{count}} artículo",
    "other": "{{count}} artículos"
  }
}
```

## Using Translations

```jsx
import { useTranslation, Trans } from 'react-i18next';

function Welcome({ user }) {
  const { t, i18n } = useTranslation();

  return (
    <div>
      {/* Simple translation */}
      <h1>{t('greeting', { name: user.name })}</h1>

      {/* Pluralization */}
      <p>{t('items', { count: user.itemCount })}</p>

      {/* Nested keys */}
      <button>{t('common.save')}</button>

      {/* Language switcher */}
      <select
        value={i18n.language}
        onChange={(e) => i18n.changeLanguage(e.target.value)}
      >
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="ar">العربية</option>
      </select>
    </div>
  );
}

// For translations with React components inside
function RichText() {
  const { t } = useTranslation();

  return (
    <Trans i18nKey="termsNotice">
      By signing up, you agree to our <Link to="/terms">Terms</Link> and{' '}
      <Link to="/privacy">Privacy Policy</Link>.
    </Trans>
  );
}
```

## Formatting

```jsx
// Configure formatters
i18n.init({
  interpolation: {
    format: function(value, format, lng) {
      if (format === 'currency') {
        return new Intl.NumberFormat(lng, {
          style: 'currency',
          currency: 'USD'
        }).format(value);
      }
      if (format === 'date') {
        return new Intl.DateTimeFormat(lng).format(value);
      }
      return value;
    }
  }
});

// Usage
function Price({ amount }) {
  const { t } = useTranslation();
  return <span>{t('cart.total', { price: amount })}</span>;
  // Output: "Total: $99.99" or "Total: 99,99 $"
}

// Date formatting
function formatDate(date, locale) {
  return new Intl.DateTimeFormat(locale, {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

// Number formatting
function formatNumber(number, locale) {
  return new Intl.NumberFormat(locale).format(number);
}
// 1234567.89 → "1,234,567.89" (en) or "1.234.567,89" (de)
```

## Pluralization

```json
// en.json - Simple plural
{
  "message": {
    "one": "You have {{count}} message",
    "other": "You have {{count}} messages"
  }
}

// ar.json - Arabic has 6 plural forms!
{
  "message": {
    "zero": "ليس لديك رسائل",
    "one": "لديك رسالة واحدة",
    "two": "لديك رسالتان",
    "few": "لديك {{count}} رسائل",
    "many": "لديك {{count}} رسالة",
    "other": "لديك {{count}} رسالة"
  }
}
```

## RTL Support

```jsx
// Detect and apply RTL
function App() {
  const { i18n } = useTranslation();
  const isRTL = ['ar', 'he', 'fa'].includes(i18n.language);

  useEffect(() => {
    document.documentElement.dir = isRTL ? 'rtl' : 'ltr';
    document.documentElement.lang = i18n.language;
  }, [i18n.language, isRTL]);

  return <MainApp />;
}

// CSS for RTL
.card {
  margin-left: 20px;
}

[dir="rtl"] .card {
  margin-left: 0;
  margin-right: 20px;
}

// Or use logical properties (modern CSS)
.card {
  margin-inline-start: 20px; /* Works for both LTR and RTL */
}
```

## Lazy Loading Translations

```jsx
import i18n from 'i18next';
import Backend from 'i18next-http-backend';

i18n
  .use(Backend)
  .use(initReactI18next)
  .init({
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    ns: ['common', 'home', 'profile'],
    defaultNS: 'common',
    fallbackLng: 'en',
  });

// Load namespace when needed
function ProfilePage() {
  const { t } = useTranslation('profile');
  // Loads /locales/en/profile.json
}
```

## Translation Workflow

```jsx
// Extract keys automatically
// npx i18next-parser 'src/**/*.{js,jsx}'

// Translation management
// 1. Extract new keys
// 2. Send to translators (Phrase, Crowdin, Lokalise)
// 3. Import translated files
// 4. Test all locales

// CI check for missing translations
function validateTranslations() {
  const languages = ['en', 'es', 'ar'];
  const baseKeys = getAllKeys(resources.en);

  languages.forEach(lang => {
    const langKeys = getAllKeys(resources[lang]);
    const missing = baseKeys.filter(key => !langKeys.includes(key));
    if (missing.length) {
      console.warn(`Missing in ${lang}:`, missing);
    }
  });
}
```

## Best Practices

1. **Never hardcode text** - All user-facing strings in translation files
2. **Use namespaces** - Organize by feature/page
3. **Context for translators** - Add descriptions for ambiguous keys
4. **Test all locales** - Especially RTL and long text
5. **Handle missing translations** - Graceful fallback

```json
// Add context for translators
{
  "save_button": "Save",
  "save_button_description": "Button text for saving user profile changes"
}
```

## Practice Exercises

1. Set up i18next with lazy-loaded translations
2. Add RTL support to an existing app
3. Implement a language switcher with persistence

## Further Reading

- [react-i18next Documentation](https://react.i18next.com/)
- [ICU Message Format](https://unicode-org.github.io/icu/userguide/format_parse/messages/)
