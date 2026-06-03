# Security Headers & Helmet

## Overview

**Use Helmet and disable `X-Powered-By`.** Express advertises itself by default (`X-Powered-By: Express`), which hands attackers a free fingerprint to target version-specific exploits. Beyond that, browsers enforce a set of HTTP response headers — Content-Security-Policy, HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy — that defend against XSS, clickjacking, MIME sniffing, and protocol downgrade. Setting these by hand is error-prone; [Helmet](https://helmetjs.github.io/) bundles sensible defaults in one line.

```js
const helmet = require('helmet');
app.use(helmet()); // sets a dozen headers with safe defaults
```

If you don't want the full Helmet dependency, at minimum disable the fingerprint:

```js
app.disable('x-powered-by');
```

Helmet's defaults are a baseline, not a finish line. The header that needs real thought is **Content-Security-Policy** — the default CSP is restrictive and will block inline scripts/styles and third-party origins, so tune it to the app rather than disabling it.

## When to Use

Apply to any Express app that serves responses to a browser, returns HTML, or sets cookies. For pure machine-to-machine JSON APIs the header set is smaller but `X-Powered-By` removal and HSTS still apply.

## Common Rationalizations

- *"It's just a JSON API, headers don't matter."* `X-Powered-By` still fingerprints you, HSTS still prevents downgrade, and `X-Content-Type-Options: nosniff` still stops a browser from reinterpreting your JSON. Cheap to set, real to skip.
- *"Helmet's CSP broke my page so I turned CSP off."* Turning it off removes the single most effective XSS mitigation. Tune the directives instead.
- *"The proxy sets headers."* Sometimes true — then verify it actually does (curl the response) rather than assuming. Duplicated/conflicting headers are their own bug.
- *"I set the headers manually already."* Fine if complete and correct, but Helmet keeps them current as best practices shift; manual sets drift.

## Red Flags

- No `helmet()` and no `app.disable('x-powered-by')` — response carries `X-Powered-By: Express`.
- `helmet({ contentSecurityPolicy: false })` — CSP deliberately disabled.
- Security headers set inline in individual route handlers (inconsistent coverage) rather than as app-level middleware.
- `Access-Control-Allow-Origin: *` combined with credentials (see transport-and-config.md).

## Verification

- `curl -I https://your-app/` and inspect headers: confirm `X-Powered-By` is absent and `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff` are present.
- Load the app in a browser with devtools open; check the console for CSP violations and tune directives to allow only the origins/inline-hashes you actually need.
- Confirm Helmet middleware is registered **early**, before routes, so every response is covered.

## Before / After

**Before**
```js
const express = require('express');
const app = express();
// no helmet, X-Powered-By: Express leaks on every response
app.get('/', (req, res) => res.send('<h1>hi</h1>'));
```

**After**
```js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],          // add specific origins/hashes as needed
      styleSrc: ["'self'", "'unsafe-inline'"], // narrow this if you can
    },
  },
}));

app.get('/', (req, res) => res.send('<h1>hi</h1>'));
```