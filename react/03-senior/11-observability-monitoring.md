# Observability and Monitoring

> **Last reviewed**: 2026-01-06


## Week Eleven: The Visibility Gap

Errors are slipping into production and nobody notices until support tickets arrive. Sarah wants visibility before users complain. Marcus points out that without clear telemetry, performance and reliability regress silently. This week is about building observability for the frontend: errors, performance, logs, analytics, and alerts that surface the right signals at the right time.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Error tracking** | A smoke detector - catch failures early |
| **Performance monitoring** | A speedometer - track runtime health |
| **Logging** | A black box - context when things go wrong |
| **Alerting** | A pager - only wake someone when it matters |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 03 (System Design) - Familiarity with system boundaries and production trade-offs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Implement error tracking with actionable metadata
- [ ] Monitor performance and web vitals in production
- [ ] Build a logging strategy for frontend events
- [ ] Instrument analytics responsibly
- [ ] Create alerting rules that reduce noise
- [ ] Define dashboard metrics for operational health

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice observability systems over 6-8 weeks

---

## Chapter 1: Error Tracking with Sentry

You need error visibility with enough context to fix issues quickly.

"Logs without context are noise," Sarah says.

### Setup

```tsx
// sentry.client.config.ts
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
  replaysSessionSampleRate: 0.1, // 10% of sessions
  replaysOnErrorSampleRate: 1.0, // 100% on error

  beforeSend(event, hint) {
    // Filter or modify events
    if (event.exception) {
      const error = hint.originalException;
      if (error?.message?.includes('ResizeObserver')) {
        return null; // Ignore noisy errors
      }
    }
    return event;
  },
});
```

### Error Boundary Integration

```tsx
import * as Sentry from '@sentry/react';

function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={<ErrorFallback />}
      beforeCapture={(scope) => {
        scope.setTag('location', 'app-root');
      }}
    >
      <MainApp />
    </Sentry.ErrorBoundary>
  );
}

// Manual error reporting
function handlePaymentError(error: Error, context: PaymentContext) {
  Sentry.captureException(error, {
    tags: {
      feature: 'payments',
      provider: context.provider,
    },
    extra: {
      amount: context.amount,
      currency: context.currency,
    },
    user: {
      id: context.userId,
    },
  });
}
```

## Chapter 2: Performance Monitoring

Track runtime performance before it shows up in user complaints.

"Measure what users feel, not just server time," Marcus says.

### Web Vitals

```tsx
import { onCLS, onINP, onLCP, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating, // 'good', 'needs-improvement', 'poor'
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType,
  });

  // Use sendBeacon for reliability
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/vitals', body);
  } else {
    fetch('/api/vitals', { body, method: 'POST', keepalive: true });
  }
}

// Report all vitals
onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);
```

### Custom Performance Marks

```tsx
// Mark important operations
function usePerformanceTrack(name: string) {
  useEffect(() => {
    performance.mark(`${name}-start`);

    return () => {
      performance.mark(`${name}-end`);
      performance.measure(name, `${name}-start`, `${name}-end`);

      const [measure] = performance.getEntriesByName(name);
      if (measure) {
        sendToAnalytics({
          name: `custom.${name}`,
          value: measure.duration,
        });
      }
    };
  }, [name]);
}

// Usage
function Dashboard() {
  usePerformanceTrack('dashboard-mount');
  // ...
}
```

## Chapter 3: Logging Strategy

Logs should answer "what happened" without drowning you in noise.

"Logs should answer questions, not create them," Sarah says.

### Structured Logging

```tsx
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
}

class Logger {
  private static instance: Logger;
  private buffer: LogEntry[] = [];
  private flushInterval = 5000;

  private constructor() {
    setInterval(() => this.flush(), this.flushInterval);
    window.addEventListener('beforeunload', () => this.flush());
  }

  static getInstance() {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }

  private log(level: LogLevel, message: string, context?: Record<string, unknown>) {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context,
    };

    this.buffer.push(entry);

    if (process.env.NODE_ENV === 'development') {
      console[level](message, context);
    }

    if (level === 'error') {
      this.flush(); // Flush immediately on error
    }
  }

  info(message: string, context?: Record<string, unknown>) {
    this.log('info', message, context);
  }

  error(message: string, context?: Record<string, unknown>) {
    this.log('error', message, context);
  }

  private async flush() {
    if (this.buffer.length === 0) return;

    const logs = [...this.buffer];
    this.buffer = [];

    await fetch('/api/logs', {
      method: 'POST',
      body: JSON.stringify(logs),
      keepalive: true,
    });
  }
}

export const logger = Logger.getInstance();

// Usage
logger.info('User signed in', { userId: user.id, method: 'google' });
logger.error('Payment failed', { error: err.message, orderId });
```

## Chapter 4: Analytics

Product analytics should be precise and respectful of user privacy.

"Analytics should serve decisions, not vanity," Marcus says.

### Event Tracking

```tsx
interface AnalyticsEvent {
  name: string;
  properties?: Record<string, string | number | boolean>;
}

class Analytics {
  private queue: AnalyticsEvent[] = [];

  track(name: string, properties?: Record<string, unknown>) {
    const event = {
      name,
      properties: {
        ...properties,
        timestamp: Date.now(),
        url: window.location.pathname,
        sessionId: this.getSessionId(),
      },
    };

    this.queue.push(event);
    this.flush();
  }

  page(name: string) {
    this.track('page_view', { page: name });
  }

  identify(userId: string, traits?: Record<string, unknown>) {
    this.track('identify', { userId, ...traits });
  }

  private flush() {
    // Debounced flush to analytics service
  }
}

export const analytics = new Analytics();

// React hook
function usePageView(pageName: string) {
  useEffect(() => {
    analytics.page(pageName);
  }, [pageName]);
}
```

### User Session Tracking

```tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function AnalyticsProvider({ children }: { children: React.ReactNode }) {
  const location = useLocation();

  useEffect(() => {
    analytics.page(location.pathname);
  }, [location]);

  return children;
}
```

## Chapter 5: Alerting

Alerts should be actionable, not constant.

"Alerts should wake you for real problems only," Sarah says.

### Alert Configuration

```typescript
// alerting-rules.ts
const rules = {
  errorRate: {
    threshold: 0.01, // 1% error rate
    window: '5m',
    severity: 'critical',
    channels: ['slack', 'pagerduty'],
  },
  p95Latency: {
    threshold: 2000, // 2 seconds
    window: '10m',
    severity: 'warning',
    channels: ['slack'],
  },
  webVitals: {
    LCP: { threshold: 2500, severity: 'warning' },
    INP: { threshold: 200, severity: 'warning' },
    CLS: { threshold: 0.1, severity: 'info' },
  },
};
```

### Health Checks

```tsx
// pages/api/health.ts
export async function GET() {
  const checks = await Promise.all([
    checkDatabase(),
    checkRedis(),
    checkExternalAPI(),
  ]);

  const healthy = checks.every(c => c.status === 'healthy');

  return Response.json(
    {
      status: healthy ? 'healthy' : 'unhealthy',
      checks,
      timestamp: new Date().toISOString(),
    },
    { status: healthy ? 200 : 503 }
  );
}
```

## Chapter 6: Dashboard Metrics

Dashboards tell you if the system is healthy at a glance.

"Dashboards tell the story at a glance," Marcus says.

Key metrics to track:

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Error Rate | < 0.1% | > 1% |
| P95 Response Time | < 500ms | > 2s |
| LCP | < 2.5s | > 4s |
| Apdex Score | > 0.9 | < 0.7 |
| Session Duration | Baseline | -50% |

---

## Common Mistakes

1. **Logging without context** - Errors without user or route info are hard to debug.
2. **Alert fatigue** - Too many alerts get ignored.
3. **Sampling too low** - You miss rare but critical issues.
4. **No ownership** - Metrics without owners do not improve.


Example:
```ts
// ❌ Log and lose the context
console.error('Checkout failed');

// ✅ Capture with context
captureException(err, { tags: { area: 'checkout' } });
```

## Practice Exercises

1. Set up Sentry with custom error boundaries
2. Implement Web Vitals tracking dashboard
3. Create alerting rules for critical metrics

### Solutions

<details>
<summary>Exercise 1: Error Boundaries + Sentry</summary>

```tsx
function AppShell() {
  return (
    <Sentry.ErrorBoundary
      fallback={<ErrorFallback />}
      beforeCapture={(scope) => {
        scope.setTag('area', 'app-shell');
        scope.setContext('release', { version: window.__BUILD_ID__ });
      }}
    >
      <App />
    </Sentry.ErrorBoundary>
  );
}
```

**Key points:**
- Boundaries keep crashes isolated.
- Sentry captures stack traces with useful tags.
- Fallback UI keeps users moving.

</details>

<details>
<summary>Exercise 2: Web Vitals Dashboard</summary>

```tsx
import { onCLS, onINP, onLCP } from 'web-vitals';

function report(metric) {
  const payload = {
    ...metric,
    path: window.location.pathname,
    buildId: window.__BUILD_ID__
  };

  navigator.sendBeacon('/api/metrics', JSON.stringify(payload));
}

onCLS(report);
onINP(report);
onLCP(report);
```

**Key points:**
- Send vitals to a backend for aggregation.
- Plot metrics per route and device.
- Track regressions by release.

</details>

<details>
<summary>Exercise 3: Alert Rules</summary>

```js
const alerts = [
  { metric: 'errorRate', threshold: 0.01, window: '5m', severity: 'critical' },
  { metric: 'LCP', threshold: 4000, window: '15m', severity: 'warning' }
];
```

**Key points:**
- Alerts should be tied to user impact.
- Use longer windows for noisy metrics.
- Route critical alerts to on-call channels only.

</details>

---

## What You Learned

This module covered:

- **Error tracking**: Capturing crashes with context
- **Performance monitoring**: Measuring runtime health
- **Logging**: Structured data for debugging
- **Analytics**: Product signals with intent
- **Alerting**: Actionable thresholds

**Key takeaway**: Observability is the difference between guessing and knowing.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add error boundaries around critical routes
- Track Core Web Vitals by page
- Define alert thresholds for error spikes
- Create a dashboard for frontend health
- Route logs with user context for faster triage

---

## Further Reading

- [Sentry React Documentation](https://docs.sentry.io/platforms/javascript/guides/react/)
- [Web Vitals](https://web.dev/vitals/)

---

**Navigation**: [← Previous Module](./10-design-handoff-collaboration.md) | [Next Module →](./12-production-debugging-triage.md)
