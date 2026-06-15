# Performance — Deployment / Environment

## Overview

**Run in production mode, use all the cores, restart on crash, cache/offload at the edges.** Environment-level levers from the official Express performance guide — often bigger wins than code tuning:

1. **`NODE_ENV=production`.** The single highest-leverage setting. Express caches view templates and CSS, generates less verbose error output, and skips dev-only work — measurably faster, often several times the throughput. Must be set in the actual production environment (shell/orchestrator), not hardcoded in the app.

2. **Use a process manager and restart on crash.** A crashed Node process should come back automatically. Use PM2, systemd, or the orchestrator's restart policy (Kubernetes liveness/readiness). Don't rely on a bare `node server.js`.

3. **Run multiple instances (use all cores).** One Node process uses one core. Run one process per core via the cluster module, PM2 cluster mode, or (better) multiple container replicas behind a load balancer. In Kubernetes/serverless, scale via replicas rather than in-process clustering.

4. **Cache.** Cache expensive results — a reverse-proxy cache (nginx, Varnish, CDN) for static and cacheable responses, an application cache (Redis/in-memory) for hot computed data. Set proper `Cache-Control`/`ETag` headers.

5. **Serve static assets from a proxy/CDN, not Node.** `express.static` works, but nginx/CDN serves files far more efficiently and frees Node for dynamic work.

6. **Terminate TLS and gzip at the proxy** for high-traffic apps (see performance-runtime.md and transport-and-config.md).

## When to Use

When reviewing how the app is deployed, scaled, and operated — `Dockerfile`, `package.json` scripts, PM2/systemd config, Kubernetes manifests, nginx config, or any "why is this slow under load" question.

## Common Rationalizations

- *"`NODE_ENV` doesn't matter much."* It does — one of the largest single throughput levers Express documents. Set it in production.
- *"One process is fine, the box has headroom."* It's pinned to one core while others idle. Cluster or replicate.
- *"If it crashes I'll restart it manually."* At 3am? Use a process manager / restart policy.
- *"Node can serve the static files."* It can, but inefficiently; a proxy/CDN does it an order of magnitude cheaper and offloads the event loop.
- *"Caching is premature optimization."* For genuinely expensive, frequently-requested, stable responses it's the cheapest big win available.

## Red Flags

- `NODE_ENV` unset or set to `development` in production; or `NODE_ENV` hardcoded in app code.
- App started with bare `node server.js`, no PM2/systemd/orchestrator restart policy.
- Single process on a multi-core host; no cluster mode and no horizontal replicas.
- All static assets served by `express.static` in a high-traffic public app, no CDN/proxy.
- No caching layer and no `Cache-Control`/`ETag` on cacheable responses.
- No liveness/readiness probes in a Kubernetes deployment.

## Verification

- Confirm `NODE_ENV=production` in the actual prod environment: `console.log(process.env.NODE_ENV)` or check the orchestrator config — not the source.
- Confirm a restart strategy exists (PM2 `ecosystem.config.js`, systemd unit, or K8s `restartPolicy` + probes).
- Confirm the app runs as many instances as cores/replicas; check PM2 cluster mode or replica count.
- Confirm static assets and TLS/gzip are handled by the proxy/CDN for high-traffic apps.
- Check responses for `Cache-Control`/`ETag`; confirm a cache fronts expensive endpoints.

## Before / After

**Before**
```json
// package.json — dev-style start, single process, no NODE_ENV
{
  "scripts": {
    "start": "node server.js"
  }
}
```
```js
// server.js — Node serves everything, single core
app.use(express.static('public'));
app.listen(3000);
```

**After**
```json
// package.json — production start via PM2 cluster mode
{
  "scripts": {
    "start": "NODE_ENV=production pm2-runtime start ecosystem.config.js"
  }
}
```
```js
// ecosystem.config.js — one process per core, auto-restart
module.exports = {
  apps: [{
    name: 'api',
    script: 'server.js',
    instances: 'max',      // use all cores
    exec_mode: 'cluster',
    env: { NODE_ENV: 'production' },
    autorestart: true,
  }],
};
```
nginx (or a CDN) fronts the app: terminates TLS, gzips responses, serves `public/` static assets, and caches what it can — leaving Node to do only dynamic work.