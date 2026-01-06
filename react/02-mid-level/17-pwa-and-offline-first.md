# PWA and Offline-First

> **Last reviewed**: 2026-01-06


## Week Seventeen: The Train Ride Demo

You demo the app on a train. The network drops, the screen goes blank, and the demo ends early.

"Users go offline more than you think," Sarah says. "Your UI should degrade gracefully."

Marcus adds, "Offline-first is a promise. Keep it small, keep it honest."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Service worker** | A doorman - decides what gets in when the network fails |
| **Cache strategy** | A pantry plan - what to store and when to refill |
| **Offline UI** | An emergency kit - the essentials when everything breaks |
| **Update flow** | A delivery schedule - new versions arrive on a timer |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 15 (Deployment and CI) - Familiarity with production builds and releases.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Register a service worker safely
- [ ] Choose cache strategies for pages and assets
- [ ] Build offline fallbacks for critical screens
- [ ] Handle update flows without breaking users
- [ ] Test offline behavior with DevTools
- [ ] Decide when PWA features are worth it

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply offline patterns over 3-5 weeks

---

## Chapter 1: The First Offline Moment

Sarah pulls the cable. The UI breaks.

"Let's get the basics: service worker registration." 

```js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js');
  });
}
```

"Now you can intercept network requests."

---

## Chapter 2: Cache Strategies

Choose one per route:

Sarah asks which screens need freshness and which can be cached.

- Cache-first for static assets
- Network-first for dynamic pages
- Stale-while-revalidate for dashboards

"One strategy doesn't fit all," Marcus says.

---

## Chapter 3: Offline Fallbacks

Create a simple offline page.

"Make failure graceful," Marcus says.

```html
<!-- public/offline.html -->
<h1>You're offline</h1>
<p>Check your connection and try again.</p>
```

"This is honesty with users."

---

## Chapter 4: Updates and Reloads

Service workers update in the background.

Marcus warns about surprise reloads that drop user progress.

"Notify users when a new version is ready," Sarah says.

---

## Chapter 5: Testing Offline

Use DevTools:

Sarah says, "Test the tunnel before you ship the train."

1) Application > Service Workers
2) Network > Offline
3) Refresh and verify fallbacks

---

## Chapter 6: When to Ship PWA Features

"Don't add offline just because it's cool," Marcus says.

Focus on:
- Field workers
- Low-connectivity regions
- Critical workflows

---

## Common Mistakes

1. **Caching everything** - Stale data ruins trust.
2. **No offline UI** - Users think the app is broken.
3. **Silent updates** - Users stay on old versions.
4. **Ignoring scope** - Keep offline features focused.


Example:
```js
// ❌ Cache all API responses forever
caches.match(request) || fetch(request)

// ✅ Cache static assets, not sensitive APIs
if (request.destination === 'script') { /* cache */ }
```

---

## Practice Exercises

1. Register a service worker in a small app
2. Add an offline fallback page
3. Decide cache strategies for three routes

### Solutions

<details>
<summary>Exercise 1: Register SW</summary>

```js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

**Key points:**
- Register after load.
- Keep scope clear.
- Test locally.

</details>

<details>
<summary>Exercise 2: Offline Page</summary>

```html
<h1>Offline</h1>
<p>Reconnect to continue.</p>
```

**Key points:**
- Keep messaging short.
- Make it obvious the app is offline.
- Link back when online.

</details>

<details>
<summary>Exercise 3: Cache Plan</summary>

```text
Assets: cache-first
Dashboard: stale-while-revalidate
Checkout: network-first
```

**Key points:**
- Cache static assets aggressively.
- Keep critical flows fresh.
- Revalidate dashboards in the background.

</details>

---

## What You Learned

This module covered:

- **Service workers**: Intercept network requests
- **Strategies**: Cache-first vs network-first
- **Offline UI**: Honest fallbacks for users
- **Updates**: Controlled refresh flows
- **Testing**: Validate offline behavior

**Key takeaway**: Offline-first is about resilience, not magic.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add offline fallbacks to a read-only page
- Cache critical static assets
- Notify users about new versions
- Test offline behavior before a release
- Decide if a PWA is worth the complexity

---

## Further Reading

- [MDN: Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Workbox](https://developer.chrome.com/docs/workbox/)
- [PWA Checklist](https://web.dev/pwa-checklist/)

---

**Navigation**: [← Previous Module](./16-react-router-data-apis.md)
