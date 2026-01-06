# Web Workers and Off-Main-Thread Work

> **Last reviewed**: 2026-01-06


## Week Nineteen: The Frozen Chart

A dashboard chart locks up every time it loads. The data is huge, the main thread is stuck parsing, and the UI stops responding. Support calls it "the frozen chart bug."

"The UI is a single lane road," Sarah says. "Heavy work needs a side road."

Marcus adds, "If the main thread is blocked, it does not matter how fast your API is."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Main thread** | A single-lane bridge - only one car moves at a time |
| **Web worker** | A side workshop - heavy work without blocking the front desk |
| **Message passing** | A courier - send packages back and forth |
| **Transferables** | A moving truck - hand off heavy cargo without copying |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 04 (Advanced Performance) - Comfort with profiling and main-thread bottlenecks.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Identify when main-thread work must move off-thread
- [ ] Create and manage Web Workers in a modern bundler setup
- [ ] Design worker messages and handle errors safely
- [ ] Use transferables for large data payloads
- [ ] Integrate workers into React with clean hooks
- [ ] Decide when workers are worth the overhead

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Use workers across multiple performance incidents over 6-8 weeks

---

## Chapter 1: Spot the Main-Thread Bottleneck

Start with evidence, not guesses.

"If you can't measure it, you can't move it," Sarah says.

```tsx
performance.mark('start-parse');
const result = parseHugePayload(payload);
performance.mark('end-parse');
performance.measure('parse', 'start-parse', 'end-parse');
```

"If a task blocks the main thread for more than a few frames, move it," Sarah says.

---

## Chapter 2: Create a Worker

Use a dedicated worker for a single job stream.

"Give heavy work its own lane," Marcus says.

```ts
const worker = new Worker(new URL('./stats.worker.ts', import.meta.url), {
  type: 'module',
});

worker.postMessage({ type: 'PARSE', payload });

worker.onmessage = event => {
  setStats(event.data.result);
};
```

"Workers are just modules that run off-thread," Marcus says.

---

## Chapter 3: Message Shape and Safety

Keep messages explicit and serializable.

"If you can't serialize it, don't send it," Sarah says.

```ts
// main thread
worker.postMessage({ type: 'AGGREGATE', payload });

// worker
self.onmessage = event => {
  if (event.data.type === 'AGGREGATE') {
    const result = aggregate(event.data.payload);
    self.postMessage({ type: 'DONE', result });
  }
};
```

"If you cannot serialize it, do not send it," Sarah says.

---

## Chapter 4: Transfer Big Payloads

Copying large buffers is slow. Transfer instead.

"Transfer ownership, don't copy the truck," Marcus says.

```ts
const buffer = new ArrayBuffer(1024 * 1024);
worker.postMessage({ buffer }, [buffer]);
```

"Transferables move ownership. The original buffer becomes unusable," Marcus says.

---

## Chapter 5: Integrate with React

Wrap the worker in a hook so components stay clean.

"Hide the worker behind a clean hook," Sarah says.

```tsx
function useWorkerTask<TInput, TOutput>(workerUrl: URL) {
  const [result, setResult] = useState<TOutput | null>(null);
  const workerRef = useRef<Worker | null>(null);

  useEffect(() => {
    const worker = new Worker(workerUrl, { type: 'module' });
    workerRef.current = worker;
    worker.onmessage = event => setResult(event.data);
    return () => {
      worker.terminate();
      workerRef.current = null;
    };
  }, [workerUrl]);

  const run = useCallback((input: TInput) => {
    workerRef.current?.postMessage(input);
  }, []);

  return { result, run };
}
```

"Your UI code should not know how the worker works," Sarah says.

---

## Chapter 6: Manage Lifecycle and Pools

Workers are not free. Start them when needed and shut them down.

"Workers are tools, not defaults," Marcus says.

- Use a single worker for a stream of tasks
- Terminate workers on unmount or after idle
- Prefer a small pool for heavy, parallel workloads
- Provide a fallback for environments without workers

"A worker is a tool, not a default," Marcus says.

---

## Common Mistakes

1. **Sending non-serializable data** - functions, DOM nodes, or class instances.
2. **Ignoring worker errors** - uncaught errors crash the worker silently.
3. **Copying huge payloads** - transfer buffers instead.
4. **Overusing workers** - tiny tasks can be slower off-thread.

Example:

```ts
// ❌ send a DOM node and function
worker.postMessage({ node: document.body, onDone: () => {} });

// ✅ send serializable data
worker.postMessage({ html: document.body.outerHTML });
```

---

## Practice Exercises

1. Move a CPU-heavy aggregation function into a worker.
2. Convert a large `Float32Array` pipeline to use transferables.
3. Build a `useWorkerTask` hook with cleanup and error handling.

### Solutions

<details>
<summary>Exercise 1: Worker Aggregation</summary>

```ts
// main thread
worker.postMessage({ type: 'AGGREGATE', payload });

// worker
self.onmessage = event => {
  const result = aggregate(event.data.payload);
  self.postMessage({ type: 'DONE', result });
};
```

**Key points:**
- Keep message types explicit.
- Return only the computed result.
- Handle errors inside the worker.

</details>

<details>
<summary>Exercise 2: Transferables</summary>

```ts
worker.postMessage({ buffer }, [buffer]);
```

**Key points:**
- Transfer ownership of large buffers.
- Avoid copy costs on large data.
- Expect the original buffer to detach.

</details>

<details>
<summary>Exercise 3: Hook Cleanup</summary>

```tsx
useEffect(() => {
  const worker = new Worker(workerUrl, { type: 'module' });
  return () => worker.terminate();
}, [workerUrl]);
```

**Key points:**
- Terminate workers on unmount.
- Keep worker lifecycle inside the hook.
- Expose a simple `run` API to components.

</details>

---

## What You Learned

This module covered:

- **Main-thread bottlenecks**: How to spot them
- **Workers**: Offload CPU-heavy tasks
- **Message passing**: Safe, serializable communication
- **Transferables**: Fast handoff of large data
- **React integration**: Hooks that hide worker details

**Key takeaway**: Workers protect responsiveness when CPU work grows.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move JSON parsing or CSV export off the main thread
- Keep charts responsive during heavy computation
- Prevent input lag during complex data transforms
- Build worker-based search indexing
- Improve perceived performance for large datasets

---

## Further Reading

- [MDN: Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
- [MDN: Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)
- [Web.dev: Off-main-thread work](https://web.dev/off-main-thread/)

---

**Navigation**: [← Previous Module](./18-modern-react-apis.md) | [Next Module →](./20-privacy-and-pii-logging.md)
