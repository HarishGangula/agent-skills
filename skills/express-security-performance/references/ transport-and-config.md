# Transport, CORS, Config & Dependencies

## Overview

**Encrypt transport, configure trust correctly, scope CORS tightly, and keep dependencies clean.** This is the deployment-facing security layer:

1. **TLS everywhere.** Serve over HTTPS and redirect HTTP → HTTPS. In most production setups TLS is terminated at a reverse proxy (nginx, ALB, Cloudflare), not in Node. That's fine and usually preferable — but then Express must be told to trust the proxy.

2. **`trust proxy`.** Behind a proxy, `req.protocol`, `req.ip`, and `secure` cookies depend on the `X-Forwarded-*` headers. Set `app.set('trust proxy', 1)` (or the correct hop count) so HTTPS detection and rate-limiting-by-IP work. Getting this wrong either breaks secure cookies or lets clients spoof their IP.

3. **CORS, scoped.** Don't reflexively `Access-Control-Allow-Origin: *`, and **never** combine `*` with credentials. Allowlist the specific origins that need access.

   ```js
   const cors = require('cors');
   app.use(cors({ origin: ['https://app.example.com'], credentials: true }));
   ```

4. **Config from environment.** `NODE_ENV`, DB URLs, secrets, and feature flags come from env vars, not literals. (`NODE_ENV=production` also has major performance effects — see performance-deployment.md.)

5. **Dependency hygiene.** Most real-world Node breaches come through vulnerable dependencies. Run `npm audit`, keep deps current, and avoid abandoned packages. Pin/lockfile your versions so builds are reproducible.

## When to Use

Apply to any deployed Express service — especially when reviewing how it's exposed to the network and how it's built/shipped.

## Common Rationalizations

- *"We're behind a proxy so HTTPS isn't my problem."* Partly true, but `trust proxy` and `secure` cookies still are, and the Node↔proxy hop may need TLS too in zero-trust setups.
- *"`origin: '*'` is simpler."* It also lets any site call your API with the user's ambient credentials if you allow credentials. Allowlist.
- *"`npm audit` is all noise."* Some is; the criticals aren't. Triage rather than ignore — and lockfiles make the triage reproducible.
- *"It works without `trust proxy`."* Until you rate-limit by IP and everyone shares the proxy's IP, or secure cookies silently never set.

## Red Flags

- App listens on plain HTTP with no redirect and no proxy doing TLS.
- Behind a proxy but `trust proxy` is unset → `req.secure` always false, IP-based rate limiting all collapses to one IP.
- `cors()` with no options (defaults to reflecting all origins) or `origin: '*'` with `credentials: true`.
- Secrets / connection strings as literals; `NODE_ENV` not set to `production` in prod.
- No `package-lock.json`/lockfile committed; `npm audit` shows unaddressed criticals; dependencies years stale.

## Verification

- Confirm TLS: either Node serves HTTPS or the proxy terminates it and `trust proxy` is set; verify `req.protocol === 'https'` resolves correctly behind the proxy.
- Inspect the CORS response headers for a cross-origin request; confirm only intended origins are allowed and `*`+credentials never co-occur.
- Confirm config comes from `process.env`; grep for hardcoded URLs/secrets.
- Run `npm audit` (and `npm outdated`); confirm a lockfile is committed.

## Before / After

**Before**
```js
const app = express();
// behind nginx but proxy not trusted; CORS wide open
app.use(cors());                          // reflects any origin
app.use(session({ cookie: { secure: true } })); // never sets — req.secure is false
app.listen(3000);
```

**After**
```js
const app = express();
app.set('trust proxy', 1);                // nginx is the single trusted hop

app.use(cors({
  origin: ['https://app.example.com'],
  credentials: true,
}));

app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: { secure: true, httpOnly: true, sameSite: 'lax' }, // now sets correctly
}));

app.listen(process.env.PORT || 3000);
```