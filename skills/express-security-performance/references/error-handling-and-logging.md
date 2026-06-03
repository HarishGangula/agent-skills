# Error Handling & Logging

## Overview

**Handle errors centrally, never leak internals to clients, and log safely.** Two concerns intertwine here — one security, one stability:

1. **Don't leak stack traces or internals.** In production, an unhandled error must not send the stack trace, SQL error text, or file paths to the client — that's reconnaissance for an attacker and noise for a user. Use a central error-handling middleware that logs the detail server-side and returns a generic message.

   ```js
   app.use((err, req, res, next) => {
     logger.error(err);                       // full detail, server-side only
     res.status(err.status || 500).json({ error: 'Internal Server Error' });
   });
   ```

2. **Catch async errors.** In Express 4, errors thrown in an `async` handler are *not* caught by Express unless you pass them to `next(err)` — an unhandled rejection can crash the process. Wrap async handlers (or use a wrapper helper / `express-async-errors`). In Express 5 this is automatic; don't add redundant wrapping if they're on 5.

3. **Don't crash on synchronous exceptions, but don't swallow them either.** Let the central handler deal with operational errors; let truly unexpected errors crash and be restarted by the process manager (a crashed-and-restarted process is safer than a corrupted, limping one). Avoid a blanket `process.on('uncaughtException')` that keeps running in an unknown state.

4. **Log safely.** Don't log secrets, tokens, passwords, full request bodies with PII, or full card numbers. Use a real logger (pino/winston), not `console.log`, so logs are structured, leveled, and shippable.

## When to Use

Apply to any Express app — error handling and logging are universal. Pay special attention when you see `async` route handlers, `try/catch` scattered through handlers, or `console.log` in production code.

## Common Rationalizations

- *"Showing the stack trace helps debugging."* In dev, sure. In production it leaks structure to attackers and confuses users. Log it server-side instead.
- *"My async handler has a try/catch so I'm fine."* Only if every path calls `next(err)` or responds. A missed branch becomes an unhandled rejection.
- *"`uncaughtException` keeps us up."* It keeps a process running after it entered an undefined state — worse than a clean restart.
- *"`console.log` is fine."* It's unstructured, unleveled, synchronous (can block), and easy to leave a secret in. Use a logger.

## Red Flags

- `res.status(500).send(err.stack)` or `res.json(err)` — internals to client.
- `async (req, res) => { ... }` handlers with no surrounding catch and no async wrapper (Express 4).
- No central `(err, req, res, next)` error middleware, or it's registered *before* the routes.
- `process.on('uncaughtException', () => { /* keep going */ })`.
- `console.log(req.body)` / logging tokens, passwords, or PII.

## Verification

- Trigger an error in production mode and confirm the client gets a generic message, not a stack trace, while the server log has the full detail.
- Confirm the error-handling middleware is registered **last**, after all routes.
- For Express 4, confirm every async handler either uses a wrapper or calls `next(err)` on failure.
- Grep logs/log calls for secrets, tokens, passwords, full bodies; confirm a structured logger is used over `console`.

## Before / After

**Before**
```js
app.get('/report/:id', async (req, res) => {
  const data = await buildReport(req.params.id); // if this throws → unhandled rejection
  res.json(data);
});

app.use((err, req, res, next) => {
  res.status(500).send(err.stack);   // leaks stack trace to client
});
```

**After**
```js
// async wrapper forwards rejections to the error middleware (Express 4)
const asyncH = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get('/report/:id', asyncH(async (req, res) => {
  const data = await buildReport(req.params.id);
  res.json(data);
}));

// central handler, registered LAST
app.use((err, req, res, next) => {
  logger.error({ err, path: req.path });          // full detail, server-side
  res.status(err.status || 500).json({ error: 'Internal Server Error' });
});
```