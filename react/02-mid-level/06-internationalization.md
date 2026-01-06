# Internationalization (i18n)

> **Last reviewed**: 2026-01-06


## Week 16: The Global Launch

The product is going international. Sarah lists the new requirements: Spanish and Arabic on day one, currency formatting, and right-to-left layouts that don't break. Marcus warns you that localization is more than translating text -- it affects layout, dates, numbers, and even the order of UI elements. This week is about setting up i18n infrastructure so every new locale feels native, not bolted on.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Translations** | A dictionary - same meaning, different words |
| **Formatting** | A currency exchanger - same value, different display |
| **Namespaces** | Folder labels - keep translations organized |
| **RTL layout** | A mirror - the UI flips direction |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 05 (Error Handling Patterns) - Comfort with component structure and UI feedback patterns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Configure react-i18next with namespaces and fallbacks
- [ ] Manage translation files and keep them organized
- [ ] Handle date, number, and currency formatting
- [ ] Implement pluralization rules correctly
- [ ] Support RTL languages without layout bugs
- [ ] Build a translation workflow that scales

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice i18n patterns over 4-6 weeks

---

## Chapter 1: Setting Up react-i18next

The first step is wiring up the translation system across the app.

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

## Chapter 2: Translation Files

You need a structure that makes it easy to add new locales without chaos.

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

## Chapter 3: Using Translations

Once keys exist, components should pull translations without manual string juggling.

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

## Chapter 4: Formatting

Dates, numbers, and currencies have to feel native in every locale.

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

## Chapter 5: Pluralization

Plural rules differ by language, and hardcoded English logic will break quickly.

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

## Chapter 6: RTL Support

The Arabic layout needs to mirror cleanly without breaking your CSS.

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

## Chapter 7: Lazy Loading Translations

You can't ship every language in the initial bundle.

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

## Chapter 8: Translation Workflow

A repeatable process keeps translators, developers, and QA in sync.

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

### Best Practices

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

---

## Common Mistakes

1. **Hardcoding strings** - Every user-facing string should live in translations.
2. **Ignoring RTL early** - Layout bugs compound if you add RTL late.
3. **Shipping every locale at once** - Lazy load to avoid large bundles.
4. **Skipping context** - Translators need notes for ambiguous strings.

## Practice Exercises

1. Set up i18next with lazy-loaded translations
2. Add RTL support to an existing app
3. Implement a language switcher with persistence

### Solutions

<details>
<summary>Exercise 1: Lazy-Loaded i18n</summary>

```jsx
import i18n from 'i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';
import { initReactI18next } from 'react-i18next';

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: ['en', 'es', 'ar'],
    ns: ['common', 'home'],
    defaultNS: 'common',
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json'
    },
    interpolation: {
      escapeValue: false
    },
    detection: {
      order: ['localStorage', 'navigator'],
      caches: ['localStorage']
    }
  });
```

**Key points:**
- Translations load per namespace and language.
- Language detection handles user defaults.
- Bundles stay small on first load.

</details>

<details>
<summary>Exercise 2: RTL Support</summary>

```jsx
function AppRoot({ children, locale }) {
  const direction = locale === 'ar' ? 'rtl' : 'ltr';

  useEffect(() => {
    document.documentElement.dir = direction;
    document.documentElement.lang = locale;
  }, [direction, locale]);

  return (
    <div dir={direction} className={`app ${direction}`}>
      {children}
    </div>
  );
}
```

**Key points:**
- `dir` controls text direction and layout flow.
- Use logical CSS properties for margins and padding.
- Toggle direction based on locale.

</details>

<details>
<summary>Exercise 3: Language Switcher</summary>

```jsx
function LanguageSwitcher() {
  const { i18n } = useTranslation();

  const setLanguage = async (lng) => {
    await i18n.changeLanguage(lng);
    localStorage.setItem('language', lng);
  };

  return (
    <div className="lang-switcher">
      <button
        onClick={() => setLanguage('en')}
        aria-pressed={i18n.language === 'en'}
      >
        English
      </button>
      <button
        onClick={() => setLanguage('es')}
        aria-pressed={i18n.language === 'es'}
      >
        Espanol
      </button>
      <button
        onClick={() => setLanguage('ar')}
        aria-pressed={i18n.language === 'ar'}
      >
        Arabic
      </button>
    </div>
  );
}
```

**Key points:**
- `changeLanguage` updates the app instantly.
- Persisting in localStorage keeps choices between sessions.
- Keep labels in their own language for clarity.

</details>

---

## What You Learned

This module covered:

- **i18n setup**: Wiring react-i18next with namespaces
- **Translation structure**: Organizing locales and keys
- **Formatting**: Dates, numbers, and currencies per locale
- **Pluralization**: Correct language-specific rules
- **RTL support**: Layout mirroring for right-to-left scripts
- **Workflow**: Scaling translation updates safely

**Key takeaway**: Internationalization is a product feature, not a last-minute patch.

---

## Real-World Application

This week at work, you might use these concepts to:

- Ship Spanish translations for a marketing launch
- Add RTL support to a checkout flow
- Implement locale-specific currency formatting
- Build a translation pipeline with CI checks
- Prevent UI breakage with long or complex strings

---

## Further Reading

- [react-i18next Documentation](https://react.i18next.com/)
- [ICU Message Format](https://unicode-org.github.io/icu/userguide/format_parse/messages/)

---

**Navigation**: [← Previous Module](./05-error-handling-patterns.md) | [Next Module →](./07-testing-strategies.md)
