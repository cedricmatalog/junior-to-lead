# Observability and Monitoring

Know what's happening in production before users tell you.

## Learning Objectives

By the end of this module, you will:
- Implement error tracking and reporting
- Set up performance monitoring
- Configure logging and analytics
- Create actionable alerts

## Error Tracking with Sentry

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

## Performance Monitoring

### Web Vitals

```tsx
import { onCLS, onFID, onLCP, onFCP, onTTFB } from 'web-vitals';

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
onFID(sendToAnalytics);
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

## Logging Strategy

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

## Analytics

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

## Alerting

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
    FID: { threshold: 100, severity: 'warning' },
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

## Dashboard Metrics

Key metrics to track:

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Error Rate | < 0.1% | > 1% |
| P95 Response Time | < 500ms | > 2s |
| LCP | < 2.5s | > 4s |
| Apdex Score | > 0.9 | < 0.7 |
| Session Duration | Baseline | -50% |

## Practice Exercises

1. Set up Sentry with custom error boundaries
2. Implement Web Vitals tracking dashboard
3. Create alerting rules for critical metrics

## Further Reading

- [Sentry React Documentation](https://docs.sentry.io/platforms/javascript/guides/react/)
- [Web Vitals](https://web.dev/vitals/)
