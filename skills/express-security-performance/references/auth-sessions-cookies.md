# Auth, Sessions, Cookies & Rate Limiting

## Overview

**Secure the session/credential layer: harden cookies, protect secrets, and throttle abuse.** Several distinct best practices live here:

1. **Cookie flags.** Any cookie carrying identity must set `httpOnly` (no JS access → blunts XSS theft), `secure` (HTTPS-only → no plaintext leak), and `sameSite` (`'lax'` or `'strict'` → CSRF mitigation). For sessions, also rename the default session cookie (`connect.sid` advertises express-session).

   ```js
   app.use(session({
     secret: process.env.SESSION_SECRET,
     name: 'sessionId',               // not the default 'connect.sid'
     resave: false,
     saveUninitialized: false,
     cookie: { httpOnly: true, secure: true, sameSite: 'lax' },
   }));
   ```

2. **Don't use the default/in-memory session store in production.** The default `MemoryStore` leaks memory and doesn't scale across processes; use Redis or a DB-backed store.

3. **Secrets out of source.** Session secrets, JWT signing keys, and DB passwords come from environment/secret manager, never committed. A leaked secret is a full compromise.

4. **Rate-limit and slow brute force.** Login, password-reset, and token endpoints need rate limiting (`express-rate-limit`) to make credential stuffing impractical.

5. **JWT hygiene** (if used): verify the algorithm explicitly (reject `alg: none` and algorithm-confusion), set short expiries, and don't put secrets in the payload (it's only base64, not encrypted).

## When to Use

Apply to any app with login, sessions, cookies, API keys, or JWTs — i.e. anything with a notion of an authenticated user.

## Common Rationalizations

- *"`secure: true` breaks local dev."* Gate it on environment (`secure: process.env.NODE_ENV === 'production'`), don't drop it in production.
- *"MemoryStore is fine, we're small."* It silently leaks memory and breaks the moment you run more than one process or restart — both inevitable.
- *"The secret is in the config file, that's not source."* If it's in the repo, it's leaked. Use env vars / a secret manager.
- *"Rate limiting annoys legitimate users."* Tune the window/threshold; a generous limit still stops automated abuse.
- *"JWTs are stateless so they're more secure."* They're harder to revoke. Short expiry + a revocation strategy matters.

## Red Flags

- `res.cookie(...)` or session config without `httpOnly`, `secure`, and `sameSite`.
- `secret: 'keyboard cat'` or any literal secret in source.
- express-session with no `store` configured (defaults to MemoryStore).
- Login/auth routes with no rate limiting.
- `jwt.verify(token, secret)` without an `algorithms` allowlist, or no expiry on signing.
- Default session cookie name left as `connect.sid`.

## Verification

- Inspect `Set-Cookie` in responses: confirm `HttpOnly; Secure; SameSite=...` are present on identity cookies.
- Grep the repo for high-entropy strings and obvious secret names; confirm they come from `process.env`.
- Confirm a non-memory session store is configured for production.
- Hit the login route in a loop and confirm it starts returning 429.
- For JWT, confirm `verify` pins the algorithm and tokens carry `exp`.

## Before / After

**Before**
```js
app.use(session({
  secret: 'keyboard cat',            // hardcoded
  resave: true,
  saveUninitialized: true,
  // no cookie flags, default MemoryStore, default cookie name
}));

app.post('/login', handleLogin);     // no rate limiting
```

**After**
```js
const RedisStore = require('connect-redis').default;
const rateLimit = require('express-rate-limit');

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  name: 'sessionId',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 1000 * 60 * 60,
  },
}));

const loginLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 10 });
app.post('/login', loginLimiter, handleLogin);
```