# Input Validation & Injection

## Overview

**Validate and constrain every piece of input, and never interpolate it into a query, command, or path.** All injection — SQL, NoSQL, command, path traversal — comes from the same root cause: untrusted input treated as code or structure instead of data. The defenses are layered:

1. **Validate at the boundary.** Use a schema validator (Zod, Joi, or express-validator) on `req.body`, `req.params`, `req.query` so malformed/oversized/unexpected input is rejected before it reaches business logic.
2. **Parameterize, don't concatenate.** Use prepared statements / parameterized queries for SQL, query builders or driver parameters for NoSQL. Never build a query string with `+ userInput`.
3. **Limit the payload.** Set a body size limit so a huge request can't exhaust memory: `express.json({ limit: '10kb' })`.
4. **Avoid the shell.** Don't pass user input to `child_process.exec`; use `execFile`/`spawn` with an argument array, or avoid spawning entirely.

NoSQL injection deserves special note: in MongoDB, a JSON body like `{ "user": { "$gt": "" } }` turns a value lookup into an operator. Validate that fields are the expected primitive type, and consider `express-mongo-sanitize` to strip `$`/`.` keys.

## When to Use

Apply to any route that reads `req.body`, `req.query`, `req.params`, headers, or uploaded files and uses them in a database query, filesystem path, shell command, template, or redirect.

## Common Rationalizations

- *"It's an internal API, inputs are trusted."* Internal callers get compromised, get bugs, and get reused publicly later. Validate anyway — it's also free input documentation.
- *"The ORM/driver escapes it."* Only if you use it correctly. String-built queries bypass the ORM's protection entirely. Confirm parameterization.
- *"I check it on the frontend."* Client-side checks are UX, not security; the attacker doesn't use your frontend.
- *"It's just a number."* Until it's `"1 OR 1=1"` or a 50MB string. Coerce and bound it.

## Red Flags

- `req.query`/`req.body`/`req.params` interpolated directly into a SQL string, Mongo filter, file path, or `exec()`.
- No validation library and no manual type/shape checks on request handlers.
- `express.json()` with no `limit` option (default is 100kb but unbounded array growth elsewhere can still hurt — size and shape both matter).
- `child_process.exec(`...${userInput}...`)`.
- `res.sendFile(req.params.path)` or `fs.readFile('./uploads/' + req.params.name)` without normalizing/whitelisting (path traversal via `../`).
- User input used as a redirect target without an allowlist (open redirect).

## Verification

- Grep for `exec(`, string concatenation near `query(`/`.find(`/`.aggregate(`, and `+ req.` patterns.
- Confirm each public route has a validation schema; trace one input from request to query and verify it's parameterized.
- Test with hostile input: `' OR '1'='1`, `{ "$gt": "" }`, `../../etc/passwd`, a multi-megabyte body — confirm each is rejected or safely handled.
- Confirm a body size limit is set on the parsers.

## Before / After

**Before**
```js
app.get('/users', async (req, res) => {
  // SQL injection + no validation
  const rows = await db.query(
    `SELECT * FROM users WHERE name = '${req.query.name}'`
  );
  res.json(rows);
});
```

**After**
```js
const { z } = require('zod');
app.use(express.json({ limit: '10kb' }));

const querySchema = z.object({ name: z.string().min(1).max(100) });

app.get('/users', async (req, res, next) => {
  const parsed = querySchema.safeParse(req.query);
  if (!parsed.success) return res.status(400).json({ error: 'invalid query' });

  // parameterized — input is data, never code
  const rows = await db.query('SELECT * FROM users WHERE name = $1', [parsed.data.name]);
  res.json(rows);
});
```