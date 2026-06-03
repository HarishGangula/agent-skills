# Performance — Runtime / In-Code

## Overview

**Never block the event loop, compress responses, and keep heavy work off the main thread.** Node is single-threaded for your JavaScript; one synchronous, CPU-heavy, or blocking call stalls *every* concurrent request. The official Express performance guidance centers on this:

1. **Use asynchronous, non-blocking I/O — never the sync variants in a request path.** `fs.readFileSync`, `crypto.pbkdf2Sync`, `JSON.parse` of a huge payload, synchronous DB drivers — each blocks the event loop. Use the async/promise versions. Sync calls are acceptable only at startup (e.g. loading config once before `listen`), never per-request.

2. **Gzip/Brotli compression.** Compressing responses cuts payload size dramatically. For high-traffic production, do it at the reverse proxy (nginx) rather than in Node — it's more efficient. For lower traffic or when there's no proxy, use the `compression` middleware:

   ```js
   const compression = require('compression');
   app.use(compression());
   ```

3. **Don't do heavy computation in the request handler.** CPU-bound work (image processing, large crypto, big aggregations, report generation) blocks the loop. Offload to a worker thread, a queue/background job, or a separate service.

4. **Avoid redundant work per request.** Compile regexes, read config, and build static objects once at module load, not inside the handler. Cache expensive-but-stable results.

## When to Use

Apply to any handler that touches the filesystem, does crypto, parses/serializes large data, runs loops over big collections, or makes blocking calls. Also whenever response payloads are sizeable (JSON lists, HTML).

## Common Rationalizations

- *"`readFileSync` is just one file, it's fast."* Multiply by every concurrent request; under load it serializes them all. Use async.
- *"Compression costs CPU."* The bandwidth and latency win almost always dominates, and the proxy can do it cheaply.
- *"I'll hash the password synchronously, it's simpler."* `pbkdf2Sync`/`bcrypt` sync blocks the loop for tens of ms per call — a login storm becomes a denial of service. Use the async form.
- *"It's a small loop."* Over a 100k-row result it isn't, and it runs on the one thread serving everyone.

## Red Flags

- Any `*Sync` call (`readFileSync`, `writeFileSync`, `existsSync`, `pbkdf2Sync`, `execSync`) inside a route handler or middleware.
- No compression and no proxy-level compression, with large JSON/HTML responses.
- CPU-heavy work (image resize, PDF/report generation, big `.map`/`.reduce` over large arrays, regex over huge strings) inline in a handler.
- `new RegExp(...)`, config reads, or large constant construction inside the handler instead of module scope.
- A user-controllable input feeding a catastrophic-backtracking regex (ReDoS) — both a perf and a security issue.

## Verification

- Grep for `Sync(` within route/middleware code; confirm any remaining sync calls are startup-only.
- Confirm compression exists (middleware or proxy); check `Content-Encoding: gzip` on a response.
- Identify the heaviest handler and confirm CPU-bound work is offloaded (worker/queue) rather than inline.
- Move invariant work (regex, config, constants) to module scope; confirm handlers do only request-specific work.
- For password hashing, confirm the async API is used.

## Before / After

**Before**
```js
const fs = require('fs');

app.get('/template', (req, res) => {
  // blocks the event loop on every request
  const tpl = fs.readFileSync('./template.html', 'utf8');
  // recompiles the regex every call
  const cleaned = tpl.replace(new RegExp(req.query.term, 'g'), '');
  res.send(cleaned);
});
```

**After**
```js
const fs = require('fs/promises');

// read once at startup, not per request
let templatePromise = fs.readFile('./template.html', 'utf8');

app.use(require('compression')());

app.get('/template', async (req, res, next) => {
  try {
    const tpl = await templatePromise;
    const term = String(req.query.term || '').slice(0, 100); // bound input (avoid ReDoS)
    const cleaned = tpl.split(term).join('');                  // no dynamic regex
    res.send(cleaned);
  } catch (err) { next(err); }
});
```